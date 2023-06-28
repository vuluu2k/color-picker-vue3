<script>
  export default {
    emits: ['update:value', 'change'],
    props: {
      value: {
        type: Number,
        default: 100,
      },
    },
    data() {
      return {
        startX: 0,
        endX: 0,
        selection: null,
      }
    },
    methods: {
      handleChange(event) {
        const value = Number(event.target.value)
        this.$emit('update:value', value, this.selection)
        this.$emit('change', value, this.selection)
      },
      onMouseDown(event) {
        this.endX = event.clientX
        this.startX = event.clientX / 2
        document.addEventListener('mousemove', this.onMouseMove)
        document.addEventListener('mouseup', this.onMouseUp)
      },
      onMouseMove(event) {
        const width = this.endX - this.startX
        const alpha = Math.round(((event.clientX - this.startX) / width) * 100)
        if (0 <= alpha && alpha <= 100) {
          this.$emit('update:value', alpha, this.selection)
          this.$emit('change', alpha, this.selection)
        }
      },
      onMouseUp(event) {
        document.removeEventListener('mousemove', this.onMouseMove)
        document.removeEventListener('mouseup', this.onMouseUp)
      },
      savedSelection() {
        const sel = window.getSelection()
        if (sel.rangeCount !== 0 && !sel.focusNode?.className.startsWith('picker-more')) {
          this.selection = window.getSelection().getRangeAt(0)
        } else this.selection = null
      },
      restoreSelection() {
        if (this.selection) {
          const sel = window.getSelection()
          sel.removeAllRanges()
          sel.addRange(this.selection)
          this.selection = null
        }
      },
    },
    beforeUnmount() {
      document.removeEventListener('mousemove', this.onMouseMove)
      document.removeEventListener('mouseup', this.onMouseUp)
    },
  }
</script>

<template>
  <div class="picker-more-input-progress" @mousedown="onMouseDown">
    <input
      type="number"
      min="0"
      max="100"
      :value="value"
      @change="handleChange"
      @mousedown="savedSelection"
      @keypress.stop
      @keydown.stop
      @keyup.stop
      @blur="restoreSelection" />
    <span>%</span>
  </div>
</template>

<style scoped>
  .picker-more-input-progress {
    cursor: ew-resize;
  }
  input {
    all: unset;
    height: 100%;
    padding: 0 6px;
  }
  input[type='number'] {
    -moz-appearance: textfield;
  }
  /* Chrome, Safari, Edge, Opera */
  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
</style>
