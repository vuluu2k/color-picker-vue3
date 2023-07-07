<script>
  import ColorPickerCustom from './components/ColorPickerCustom.vue'
  import ColorGradient from './components/ColorGradient.vue'
  import ColorPickerGradient from './components/ColorPickerGradient.vue'
  import ColorGenerate from './components/ColorGenerate.vue'
  export default {
    data() {
      return {
        color: '#1677ff',
        selection: null,
        background: '#000000',
        colorPlatte: [],
      }
    },
    components: {
      ColorPickerCustom,
      ColorGradient,
      ColorPickerGradient,
      ColorGenerate,
    },
    methods: {
      handleChangeGradient(gradient) {
        this.background = gradient
      },
      shadeColor(color, percent) {
        let R = parseInt(color.substring(1, 3), 16)
        let G = parseInt(color.substring(3, 5), 16)
        let B = parseInt(color.substring(5, 7), 16)

        R = parseInt((R * (100 + percent)) / 100)
        G = parseInt((G * (100 + percent)) / 100)
        B = parseInt((B * (100 + percent)) / 100)

        R = R < 255 ? R : 255
        G = G < 255 ? G : 255
        B = B < 255 ? B : 255

        const RR =
          R.toString(16).length == 1 ? '0' + R.toString(16) : R.toString(16)
        const GG =
          G.toString(16).length == 1 ? '0' + G.toString(16) : G.toString(16)
        const BB =
          B.toString(16).length == 1 ? '0' + B.toString(16) : B.toString(16)

        return '#' + RR + GG + BB
      },

      tintColor(color, percent) {
        let R = parseInt(color.substring(1, 3), 16)
        let G = parseInt(color.substring(3, 5), 16)
        let B = parseInt(color.substring(5, 7), 16)

        R = parseInt((255 - R) * (percent / 100) + R)
        G = parseInt((255 - G) * (percent / 100) + G)
        B = parseInt((255 - B) * (percent / 100) + B)

        const RR =
          R.toString(16).length == 1 ? '0' + R.toString(16) : R.toString(16)
        const GG =
          G.toString(16).length == 1 ? '0' + G.toString(16) : G.toString(16)
        const BB =
          B.toString(16).length == 1 ? '0' + B.toString(16) : B.toString(16)

        return '#' + RR + GG + BB
      },
      handleChangePlatte(value) {
        const colors = this.generateColorPalette(value)
        const level = this.getLuminance(value)
        this.colorPlatte = colors.slice(6 - level, 11 - level)
        console.log(this.getLuminance(value))
      },

      generateColorPalette(color) {
        const palette = []
        // Add tints
        for (let i = 10; i >= 1; i -= 2) {
          palette.push(this.tintColor(color, i * 10))
        }

        palette.push(color)
        // Add original color

        // Add shades
        for (let i = 1; i <= 10; i += 2) {
          palette.push(this.shadeColor(color, -i * 10))
        }
        return palette
      },
      getLuminance(color) {
        const c = color.substring(1) // strip #
        const rgb = parseInt(c, 16) // convert rrggbb to decimal
        const r = (rgb >> 16) & 0xff // extract red
        const g = (rgb >> 8) & 0xff // extract green
        const b = (rgb >> 0) & 0xff // extract blue

        const luminance = 0.2126 * r + 0.7152 * g + 0.0722 * b // per ITU-R BT.709
        const level = luminance / 255
        if (level <= 0.2) return 5
        else if (level <= 0.4) return 4
        else if (level <= 0.6) return 3
        else if (level <= 0.8) return 2
        return 1
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

    <ColorPickerGradient v-model:value="background" />

    <div class="color-platte">
      <div
        class="color-platte-item"
        v-for="color in colorPlatte"
        :style="{ background: color }"
      ></div>
    </div>

    <div class="wrapper">
      <ColorGenerate @change="handleChangePlatte" />
    </div>
  </main>
</template>

<style scoped>
  .color-platte {
    width: 100%;
    height: 50px;
    display: flex;
  }
  .color-platte-item {
    width: 50px;
    height: 50px;
  }
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
