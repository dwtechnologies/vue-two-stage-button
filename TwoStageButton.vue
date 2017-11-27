<template>
  <button :class="classObject" @click="onClick" @blur="onBlur" :disabled="disabled || setDisabled">
    <slot v-if="stage === stages.success" name="success"></slot>
    <slot v-else-if="stage === stages.error" name="error"></slot>
    <slot v-else-if="stage === stages.initial" name="stage-1"></slot>
    <slot v-else-if="stage === stages.engaged"name="stage-2"></slot>
  </button>
</template>

<script>

const stages = { initial: 1, engaged: 2, success: 3, error: -1 }

export default {
  name: 'two-stage-button',
  props: {
    classes: {
      type: String,
      default: 'two-stage-button'
    },
    loading: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    error: {
      type: Boolean,
      default: false
    },
    action: {
      type: Function,
      required: true
    },
    successTimeout: {
      type: Number,
      default: 0
    },
    errorTimeout: {
      type: Number,
      default: 0
    },
    engagedTimeout: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      stages,
      stage: stages.initial,
      setLoading: false,
      setDisabled: false,
      setError: false
    }
  },
  created () {
    this.engagedTimer = null
    this.successTimer = null
    this.errorTimer = null
  },
  beforeDestroy () {
    clearTimeout(this.engagedTimer)
    clearTimeout(this.successTimer)
    clearTimeout(this.errorTimer)
  },
  methods: {
    onClick () {
      switch (this.stage) {
        case stages.initial:
          this.stage = stages.engaged
          this.$emit('engage')
          if (this.engagedTimeout) {
            this.engagedTimer = setTimeout(() => {
              this.stage = stages.initial
            }, this.engagedTimeout * 1000)
          }
          break

        case stages.engaged:
          this.$emit('confirm')
          const action = this.action()
          if (typeof action === 'object' && typeof action.then === 'function') {
            this.setDisabled = true
            this.setLoading = true
            action.then(() => {
              this.setLoading = false
              this.setDisabled = false
              this.stage = stages.success
              this.$emit('success')
              if (this.successTimeout) {
                this.successTimer = setTimeout(() => {
                  this.stage = stages.initial
                }, this.successTimeout * 1000)
              }
            }).catch(() => {
              this.setLoading = false
              this.setDisabled = false
              this.stage = stages.error
              this.$emit('error')
              if (this.errorTimeout) {
                this.errorTimer = setTimeout(() => {
                  this.stage = stages.initial
                }, this.errorTimeout * 1000)
              }
            })
          } else {
            this.setDisabled = false
            this.engaged = false
          }
          break
        default:
          this.stage = stages.initial
          this.$emit('reset')
          break
      }
    },
    onBlur () {
      if (this.disabled || this.setDisabled) return
      this.$emit('reset')
      this.stage = stages.initial
      this.setError = false
    }
  },
  computed: {
    staticClasses () {
      const propClasses = this.classes.trim().split(' ')
      let classes = {}
      for (const name of propClasses) {
        classes[name] = true
      }
      return classes
    },
    classObject () {
      let classes = {...this.staticClasses}
      classes['engaged'] = this.stage === stages.engaged
      classes['success'] = this.stage === stages.success
      classes['error'] = this.stage === stages.error
      classes['loading'] = this.loading || this.setLoading
      return classes
    }
  },
  watch: {
    error: function (errorState) {
      this.stage = errorState ? stages.error : stages.initial
    }
  }
}
</script>

<style scoped>
.two-stage-button {
  -webkit-appearance: none;
  background: hsl(0, 0%, 86%); 
  color: black;
  cursor: pointer;
  justify-content: center;
  padding: 0.5em 1em;
  border-radius: 0.25em;
  border: none;
  text-align: center;
  white-space: nowrap;
  font-size: 0.9em;
  outline: none;
  transition: background 0.125s;
  transition: filter 0.125s;
}

.two-stage-button:hover {
  filter: brightness(90%);
}

.two-stage-button.engaged {
  background-color: hsl(48, 100%, 67%);
}

.two-stage-button.success {
  color: white;
  background-color: hsl(141, 71%, 48%);
}

.two-stage-button.error {
  color: white;
  background-color: hsl(14, 100%, 53%);
}

.two-stage-button[disabled], .two-stage-button.loading  {
  background: hsl(0, 0%, 96%);
  color: #aaa;
}
</style>
