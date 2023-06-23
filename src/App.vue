<script>
  import ColorPickerCustom from './components/ColorPickerCustom.vue'
  import ColorGradient from './components/ColorGradient.vue'
  export default {
    data() {
      return {
        backgroundColor: '#1677ff',
        selection: null,
      }
    },
    components: {
      ColorPickerCustom,
      ColorGradient,
    },
    watch: {
      backgroundColor(value) {
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

    <ColorPickerCustom
      v-model:value="backgroundColor"
      v-model:selection="selection"
    />
    <div class="wrapper">
      <ColorGradient />
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
    background-color: #fff;
    width:fit-content;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  }
</style>
