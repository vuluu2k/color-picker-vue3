<script>
  import ColorPicker from './components/ColorPicker.vue'
  export default {
    data() {
      return {
        show: false,
        backgroundColor: '#1677ff',
        pickerGps: {
          top: 0,
          left: 0,
        },
      }
    },
    components: {
      ColorPicker,
    },
    mounted() {
      const picker = this.$refs.colorPicker.getBoundingClientRect()
      this.pickerGps.top = picker.bottom + 2
      this.pickerGps.left = picker.left
    },
    methods: {
      toggle(event) {
        event.preventDefault()
        this.show = !this.show
      },
      handlePickerColor(event) {
        this.backgroundColor = event
      },
    },
    watch: {
      backgroundColor(value) {
        const selection = window.getSelection()
        const color = value
        const range = selection.getRangeAt(0)

        const span = document.createElement('span')
        span.style.color = color
        span.textContent = range.toString()

        range.deleteContents()
        range.insertNode(span)
      },
    },
  }
</script>

<template>
  <main class="main">
    <div style="margin-bottom: 12px">Hello Xin chào các bạn</div>
    <input
      type="color"
      :value="backgroundColor"
      @change="($event) => handlePickerColor($event.target.value)"
    />
    <div
      @mousedown="toggle"
      class="color-picker"
      ref="colorPicker"
      :style="{ backgroundColor }"
    ></div>

    <Teleport to="body">
      <div
        v-if="show"
        :style="{
          position: 'absolute',
          top: `${pickerGps.top}px`,
          left: `${pickerGps.left}px`,
        }"
        ref="pickerPanel"
      >
        <ColorPicker
          :value="backgroundColor"
          @change="handlePickerColor"
          @blur="show = false"
        />
      </div>
    </Teleport>
  </main>
</template>

<style scoped>
  .color-picker {
    width: 26px;
    height: 26px;
    border-radius: 4px;
  }

  .color-picker-panel {
    padding: 8px;
    border-radius: 8px;
    z-index: 1;
    background-color: #fff;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  }
</style>
