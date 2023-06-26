<script>
  import ColorPickerCustom from './components/ColorPickerCustom.vue'
  import ColorGradient from './components/ColorGradient.vue'
  export default {
    data() {
      return {
        color: '#1677ff',
        selection: null,
        background: '#FFFFFF',
      }
    },
    components: {
      ColorPickerCustom,
      ColorGradient,
    },
    methods: {
      handleChangeGradient(gradient) {
        this.background = gradient
      },
    },
    watch: {
      color(value) {
        const selection = window.getSelection()
        if (
          (selection.rangeCount !== 0 && selection?.toString().length !== 0) ||
          this.selection
        ) {
          const color = value
          const range = this.selection || selection.getRangeAt(0)

          const span = document.createElement('span')
          span.style.color = color
          span.textContent = range.toString()

          range.deleteContents()
          range.insertNode(span)
        }
      },
    },
  }
</script>

<template>
  <main class="main">
    <p style="margin-bottom: 12px, min-width: 600px">
      Hello everyone, You can selection text and change color with color picker
    </p>
    <ColorPickerCustom v-model:value="color" v-model:selection="selection" />

    <div class="background-test" :style="{ background }"></div>

    <div class="wrapper">
      <ColorGradient @change="handleChangeGradient" />
    </div>
  </main>
</template>

<style scoped>
  .color-picker {
    width: 26px;
    height: 26px;
    border-radius: 4px;
  }

  .wrapper {
    padding: 8px;
    border-radius: 8px;
    width: fit-content;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  }

  .background-test {
    width: 400px;
    height: 200px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 12px;
  }
</style>
