<template>
  <button :class="classes">
    <slot v-if="!engaged" name="stage-1"></slot>
    <slot v-else name="stage-2"></slot>
  </button>
</template>

<script>

export default {
  name: 'two-stage-button',
  props: {
    classes: {
      type: String,
      default: 'button two-stage-button'
    },
    loading: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      engaged: false
    }
  },
  methods: {
    onClick () {

    },
    onBlur () {

    }
  }
}
</script>
