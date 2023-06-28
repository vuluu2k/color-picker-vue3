<script>
  import { debounce } from 'lodash'
  export default {
    emits: ['change', 'blur'],
    props: {
      value: {
        type: String,
        default: '#000000',
      },
      output: {
        type: String,
        default: 'hex',
      },
    },
    data() {
      return {
        selector: 'hex',
        clientX: 0,
        clientY: 0,
        hsb: {
          hue: 0,
          saturation: 0,
          brightness: 0,
        },
        rgb: {
          red: 0,
          green: 0,
          blue: 0,
        },
        hex: '#000000',
        alpha: -1,
        clientXGradient: 0,
        clientXAlpha: 0,
        selection: null,
      }
    },
    mounted() {
      // const selectRect = this.$refs.pickerSelect.getBoundingClientRect()
      // this.clientX = -8
      // this.clientY = selectRect.height - 8
      this.initColor(this.value)
    },
    computed: {
      topSelect() {
        return `${this.clientY}px`
      },
      leftSelect() {
        return `${this.clientX}px`
      },
      backgroundGradient() {
        const [red, green, blue] = this.hsbToRgb(this.hsb.hue, 100, 100)
        return `rgb(${red},${green},${blue})`
      },
      backgroundPicker() {
        return `rgb(${this.rgb.red},${this.rgb.green},${this.rgb.blue})`
      },
      rgbPreview() {
        return `rgba(${this.rgb.red},${this.rgb.green},${this.rgb.blue}, ${
          this.alpha / 100
        })`
      },
    },
    methods: {
      updateSaturationAndBrightness() {
        const rect = this.$refs.pickerSaturation.getBoundingClientRect()
        const saturation = Math.round(((this.clientX + 8) / rect.width) * 100)
        const brightness =
          100 - Math.round(((this.clientY + 8) / rect.height) * 100)
        this.hsb.saturation = saturation
        this.hsb.brightness = brightness

        this.updateColorFromHsb()
      },
      updateHue() {
        const gradientRect = this.$refs.pickerGradient.getBoundingClientRect()
        const hue = Math.round(
          ((this.clientXGradient + 6) / gradientRect.width) * 360
        )
        this.hsb.hue = hue
        this.updateColorFromHsb()
      },
      updateAlpha() {
        const alphaRect = this.$refs.pickerAlpha.getBoundingClientRect()
        const alpha = Math.round(
          ((this.clientXAlpha + 6) / alphaRect.width) * 100
        )
        this.alpha = alpha

        const hexOpacity = Math.round((alpha / 100) * 255)
          .toString(16)
          .padStart(2, '0')

        this.hex = `#${this.hex.replace('#', '').slice(0, 6)}${hexOpacity}`
      },
      rgbToHex(r, g, b) {
        return ((r << 16) + (g << 8) + b).toString(16).padStart(6, '0')
      },
      hexToRgb(hex) {
        let alpha = false,
          h = hex.slice(hex.startsWith('#') ? 1 : 0)
        if (h.length === 3) h = [...h].map((x) => x + x).join('')
        else if (h.length === 8) alpha = true

        h = parseInt(h, 16)

        return [
          h >>> (alpha ? 24 : 16),
          (h & (alpha ? 0x00ff0000 : 0x00ff00)) >>> (alpha ? 16 : 8),
          (h & (alpha ? 0x0000ff00 : 0x0000ff)) >>> (alpha ? 8 : 0),
          alpha ? h & 0x000000ff : 255,
        ]
      },
      hsbToRgb(h, s, b) {
        s /= 100
        b /= 100
        const k = (n) => (n + h / 60) % 6
        const f = (n) => b * (1 - s * Math.max(0, Math.min(k(n), 4 - k(n), 1)))
        const red = Math.round(255 * f(5))
        const green = Math.round(255 * f(3))
        const blue = Math.round(255 * f(1))
        return [red, green, blue]
      },
      rgbToHsb(r, g, b) {
        r /= 255
        g /= 255
        b /= 255
        const v = Math.max(r, g, b),
          n = v - Math.min(r, g, b)
        const h =
          n === 0
            ? 0
            : n && v === r
            ? (g - b) / n
            : v === g
            ? 2 + (b - r) / n
            : 4 + (r - g) / n

        const hue = Math.round(60 * (h < 0 ? h + 6 : h))
        const saturation = Math.round(v && (n / v) * 100)
        const brightness = Math.round(v * 100)
        return [hue, saturation, brightness]
      },
      updateColorFromHsb(
        h = this.hsb.hue,
        s = this.hsb.saturation,
        b = this.hsb.brightness
      ) {
        ;[this.rgb.red, this.rgb.green, this.rgb.blue] = this.hsbToRgb(h, s, b)
        this.hex = `#${this.rgbToHex(
          this.rgb.red,
          this.rgb.green,
          this.rgb.blue
        )}`
      },
      handleHsbInput() {
        this.updateColorFromHsb()
      },
      handleRgbInput(event) {
        const name = event.target.name
        const value = event.target.value
        const red = name == 'red' ? Number(value) : this.rgb.red
        const green = name == 'green' ? Number(value) : this.rgb.green
        const blue = name == 'blue' ? Number(value) : this.rgb.blue

        this.handleRgb(red, green, blue)
      },
      handleRgb(red, green, blue) {
        ;[this.hsb.hue, this.hsb.saturation, this.hsb.brightness] =
          this.rgbToHsb(red, green, blue)
        this.hex = `#${this.rgbToHex(red, green, blue)}`
      },
      handleHexInput(event) {
        this.handleHex(event.target.value)
      },
      handleHex(value) {
        const replaceValue = value.replaceAll('#', '')
        this.hex = `#${replaceValue}`
        const getValue =
          replaceValue.length == 7
            ? replaceValue.slice(0, 6)
            : replaceValue.length >= 8
            ? replaceValue.slice(0, 8)
            : replaceValue
        const [red, green, blue, alpha] = this.hexToRgb(getValue)
        this.rgb = { red, green, blue }

        this.alpha = Math.round((alpha / 255) * 100)
        ;[this.hsb.hue, this.hsb.saturation, this.hsb.brightness] =
          this.rgbToHsb(red, green, blue)
      },
      onMouseDown(event) {
        event.preventDefault()
        const selectRect = this.$refs.pickerSelect.getBoundingClientRect()
        this.clientX = event.clientX - selectRect.left - 8
        this.clientY = event.clientY - selectRect.top - 8
        this.updateSaturationAndBrightness()
        document.addEventListener('mousemove', this.onMouseMove)
        document.addEventListener('mouseup', this.onMouseUp)
      },
      onMouseMove(event) {
        const selectRect = this.$refs.pickerSelect.getBoundingClientRect()
        if (selectRect.top < event.clientY && event.clientY < selectRect.bottom)
          this.clientY = event.clientY - selectRect.top - 8
        if (selectRect.left < event.clientX && event.clientX < selectRect.right)
          this.clientX = event.clientX - selectRect.left - 8
        if (event.clientX > selectRect.right)
          this.clientX = selectRect.width - 8
        if (event.clientX < selectRect.left) this.clientX = -8
        if (event.clientY > selectRect.bottom)
          this.clientY = selectRect.height - 8
        if (event.clientY < selectRect.top) this.clientY = -8
        this.updateSaturationAndBrightness()
      },
      onMouseUp(event) {
        document.removeEventListener('mousemove', this.onMouseMove)
        document.removeEventListener('mouseup', this.onMouseUp)
      },
      onMouseDownGradient(event) {
        event.preventDefault()
        const gradientRect = this.$refs.pickerGradient.getBoundingClientRect()
        this.clientXGradient = event.clientX - gradientRect.left - 8
        this.updateHue()
        document.addEventListener('mousemove', this.onMouseMoveGradient)
        document.addEventListener('mouseup', this.onMouseUpGradient)
      },
      onMouseMoveGradient(event) {
        const gradientRect = this.$refs.pickerGradient.getBoundingClientRect()
        if (
          gradientRect.left < event.clientX &&
          event.clientX < gradientRect.right
        )
          this.clientXGradient = event.clientX - gradientRect.left - 6
        if (event.clientX > gradientRect.right)
          this.clientXGradient = gradientRect.width - 6
        if (event.clientX < gradientRect.left) this.clientXGradient = -6
        this.updateHue()
      },
      onMouseUpGradient(event) {
        document.removeEventListener('mousemove', this.onMouseMoveGradient)
        document.removeEventListener('mouseup', this.onMouseUpGradient)
      },
      onMouseDownAlpha(event) {
        event.preventDefault()
        const alphaRect = this.$refs.pickerAlpha.getBoundingClientRect()
        this.clientXAlpha = event.clientX - alphaRect.left - 8
        this.updateAlpha()
        document.addEventListener('mousemove', this.onMouseMoveAlpha)
        document.addEventListener('mouseup', this.onMouseUpAlpha)
      },
      onMouseMoveAlpha(event) {
        const alphaRect = this.$refs.pickerAlpha.getBoundingClientRect()
        if (alphaRect.left < event.clientX && event.clientX < alphaRect.right)
          this.clientXAlpha = event.clientX - alphaRect.left - 6
        if (event.clientX > alphaRect.right)
          this.clientXAlpha = alphaRect.width - 6
        if (event.clientX < alphaRect.left) this.clientXAlpha = -6
        this.updateAlpha()
      },
      onMouseUpAlpha(event) {
        document.removeEventListener('mousemove', this.onMouseMoveAlpha)
        document.removeEventListener('mouseup', this.onMouseUpAlpha)
      },
      onMouseDownInput(event) {
        this.savedSelection()
      },
      onMouseUpInput(event) {
        event.preventDefault()
      },
      savedSelection(event) {
        const sel = window.getSelection()
        if (
          sel.rangeCount !== 0 &&
          !sel.focusNode?.className.startsWith('color-picker')
        ) {
          this.selection = window.getSelection().getRangeAt(0)
        }
      },
      restoreSelection() {
        const sel = window.getSelection()
        if (this.selection) {
          sel.removeAllRanges()
          sel.addRange(this.selection)
          this.selection = null
        }
      },
      initColor(color) {
        if (color.startsWith('#')) {
          this.hex = color
          this.handleHex(color)
        }
        if (color.startsWith('rgb')) {
          const rgbaRegex = /rgba?\((\d+),\s*(\d+),\s*(\d+),?\s*([\d\.]+)?\)/i
          const matches = color.match(rgbaRegex)

          if (matches) {
            this.rgb.red = parseInt(matches[1])
            this.rgb.green = parseInt(matches[2])
            this.rgb.blue = parseInt(matches[3])
            this.alpha = matches[4] ? parseFloat(matches[4]) * 100 : 100
          }
          this.handleRgb(this.rgb.red, this.rgb.green, this.rgb.blue)
        }
      },
    },
    beforeUnmount() {
      document.removeEventListener('mousemove', this.onMouseMove)
      document.removeEventListener('mouseup', this.onMouseUp)
      document.removeEventListener('mousemove', this.onMouseMoveGradient)
      document.removeEventListener('mouseup', this.onMouseUpGradient)
      document.removeEventListener('mousemove', this.onMouseMoveAlpha)
      document.removeEventListener('mouseup', this.onMouseUpAlpha)
    },
    watch: {
      hsb: {
        handler(value) {
          const gradientRect = this.$refs.pickerGradient.getBoundingClientRect()
          const selectRect = this.$refs.pickerSelect.getBoundingClientRect()
          this.clientXGradient = gradientRect.width * (value.hue / 360) - 6
          this.clientX = selectRect.width * (value.saturation / 100) - 8
          this.clientY =
            selectRect.height * ((100 - value.brightness) / 100) - 8
        },
        deep: true,
      },
      alpha(value) {
        const alphaRect = this.$refs.pickerAlpha.getBoundingClientRect()
        this.clientXAlpha = alphaRect.width * (value / 100) - 6
        this.$emit(
          'change',
          (this.output == 'hex' && this.hex) ||
            (this.output == 'rgb' && this.rgbPreview),
          this.selection
        )
      },
      hex(newValue) {
        this.output == 'hex' && this.$emit('change', newValue, this.selection)
      },
      rgbPreview(newValue) {
        this.output == 'rgb' && this.$emit('change', newValue, this.selection)
      },
      value(newValue) {
        this.initColor(newValue)
      },
    },
  }
</script>

<template>
  <div class="color-picker-panel">
    <div class="color-picker-select">
      <div
        class="color-picker-palette"
        @mousedown="onMouseDown"
        ref="pickerSelect">
        <div
          :style="{
            position: 'absolute',
            left: leftSelect,
            top: topSelect,
            zIndex: 1,
          }">
          <div
            class="color-picker-handler"
            :style="{ background: backgroundPicker }" />
        </div>
        <div
          class="color-picker-saturation"
          ref="pickerSaturation"
          :style="{
            'background-color': backgroundGradient,
            'background-image':
              'linear-gradient(0deg, rgb(0, 0, 0), transparent), linear-gradient(90deg, rgb(255, 255, 255), rgba(255, 255, 255, 0))',
          }" />
      </div>
    </div>

    <div class="color-picker-slider-container">
      <div class="color-picker-slider-group">
        <div class="color-picker-slider-hue">
          <div
            class="color-picker-palette"
            @mousedown="onMouseDownGradient"
            ref="pickerGradient">
            <div
              :style="{
                position: 'absolute',
                left: `${this.clientXGradient}px`,
                top: '-2px',
                zIndex: 1,
              }">
              <div
                class="color-picker-handler"
                :style="{ backgroundColor: backgroundGradient }"></div>
            </div>
            <div class="color-picker-gradient"></div>
          </div>
        </div>
        <div class="color-picker-slider-alpha">
          <div class="color-picker-palette" @mousedown="onMouseDownAlpha">
            <div
              :style="{
                position: 'absolute',
                left: `${this.clientXAlpha}px`,
                top: '-2px',
                zIndex: 1,
              }">
              <div
                class="color-picker-handler"
                :style="{ background: rgbPreview }"></div>
            </div>
            <div
              class="color-picker-opacity"
              ref="pickerAlpha"
              :style="{
                background: `linear-gradient(to right, rgba(255, 0, 4, 0), ${backgroundPicker}`,
              }"></div>
          </div>
        </div>
      </div>
      <div class="color-picker-color-block">
        <div
          class="color-picker-color-block-inner"
          :style="{ background: rgbPreview }"></div>
      </div>
    </div>

    <div class="color-picker-input-container">
      <div class="color-picker-selector">
        <div
          class="color-picker-selector-item"
          :class="{ 'color-picker-selector-item-actived': selector === 'hex' }"
          @mousedown.prevent="selector = 'hex'">
          HEX
        </div>
        <div
          class="color-picker-selector-item"
          :class="{ 'color-picker-selector-item-actived': selector === 'rgb' }"
          @mousedown.prevent="selector = 'rgb'">
          RGB
        </div>
        <div
          class="color-picker-selector-item"
          :class="{ 'color-picker-selector-item-actived': selector === 'hsb' }"
          @mousedown.prevent="selector = 'hsb'">
          HSB
        </div>
      </div>
      <div class="color-picker-input">
        <div class="color-picker-input-type">
          <div class="color-picker-input-hex" v-if="selector === 'hex'">
            <input
              @keydown.stop
              @keyup.stop
              @keypress.stop
              data-id="input-color"
              type="text"
              :value="hex"
              name="hex"
              maxlength="9"
              @input="handleHexInput"
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @mouseup="onMouseUpInput" />
          </div>
          <div class="color-picker-input-rgb" v-if="selector === 'rgb'">
            <input
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @keydown.stop
              @keyup.stop
              @keypress.stop
              type="number"
              min="0"
              max="255"
              maxlength="3"
              name="red"
              v-model="rgb.red"
              @input="handleRgbInput" />
            <input
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @keydown.stop
              @keyup.stop
              @keypress.stop
              type="number"
              min="0"
              max="255"
              maxlength="3"
              name="green"
              v-model="rgb.green"
              @input="handleRgbInput" />
            <input
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @keydown.stop
              @keyup.stop
              @keypress.stop
              type="number"
              min="0"
              max="255"
              name="blue"
              v-model="rgb.blue"
              @input="handleRgbInput" />
          </div>
          <div class="color-picker-input-hsb" v-if="selector === 'hsb'">
            <input
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @keydown.stop
              @keyup.stop
              @keypress.stop
              name="hue"
              type="number"
              min="0"
              max="360"
              maxlength="3"
              v-model="hsb.hue"
              @input="handleHsbInput" />
            <input
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @keydown.stop
              @keyup.stop
              @keypress.stop
              name="saturation"
              type="number"
              min="0"
              max="100"
              maxlength="3"
              v-model="hsb.saturation"
              @input="handleHsbInput" />
            <input
              @blur="restoreSelection"
              @mousedown="onMouseDownInput"
              @keydown.stop
              @keyup.stop
              @keypress.stop
              name="brightness"
              type="number"
              min="0"
              max="100"
              maxlength="3"
              v-model="hsb.brightness"
              @input="handleHsbInput" />
          </div>
        </div>
        <div class="color-picker-input-alpha">
          <input
            @blur="restoreSelection"
            @mousedown="onMouseDownInput"
            @keydown.stop
            @keyup.stop
            @keypress.stop
            type="number"
            name="alpha"
            min="0"
            max="100"
            maxlength="3"
            v-model="alpha" />
        </div>
      </div>
    </div>
    <div class="color-picker-buttons">
      <button class="color-picker-cancel">Cancel</button>
      <button class="color-picker-apply">Apply</button>
    </div>
  </div>
</template>

<style scoped>
  div {
    margin: 0;
    padding: 0;
  }

  button {
    all: unset;
    padding: 6px 16px;
    border-radius: 4px;
  }

  .color-picker-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 12px;
  }

  .color-picker-cancel {
    border: 1px solid var(--divider, #e0e0e0);
  }

  .color-picker-apply {
    color: white;
    background-color: var(--primary, #1677ff);
  }

  input {
    all: unset;
    border: 1px solid var(--divider, #e0e0e0);
    height: 28px;
    border-radius: 4px;
    padding: 0 6px;
    color: #2b5672;
  }

  input[type='number'] {
    -moz-appearance: textfield;
  }
  input:focus {
    border-color: var(--primary, #1677ff);
  }
  /* Chrome, Safari, Edge, Opera */
  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  .color-picker-panel {
    display: flex;
    flex-direction: column;
    width: 240px;
  }

  .color-picker-select {
    margin-bottom: 12px;
  }

  .color-picker-palette {
    overflow: hidden;
    border-radius: 4px;
    min-height: 160px;
    position: relative;
  }

  .color-picker-handler {
    position: absolute;
    width: 16px;
    height: 16px;
    border: 2px solid #ffffff;
    position: relative;
    border-radius: 50%;
    cursor: pointer;
    box-shadow: inset 0 0 1px 0 rgba(0, 0, 0, 0.25),
      0 0 0 1px rgba(0, 0, 0, 0.06);
  }

  .color-picker-saturation {
    position: absolute;
    border-radius: inherit;
    box-shadow: inset 0 0 1px 0 rgba(0, 0, 0, 0.25);
    inset: 0;
  }

  .color-picker-slider-container {
    box-sizing: border-box;
    display: flex;
    gap: 12px;
  }

  .color-picker-slider-group {
    flex: 1;
  }

  .color-picker-slider-hue {
    border-radius: 4px;
    margin-bottom: 12px;
  }

  .color-picker-slider-hue .color-picker-palette {
    position: relative;
    height: 8px;
    min-height: 8px;
  }

  .color-picker-slider-hue .color-picker-handler {
    width: 12px;
    height: 12px;
  }

  .color-picker-slider-hue .color-picker-gradient {
    position: absolute;
    inset: 0px;
    background: linear-gradient(
      to right,
      rgb(255, 0, 0),
      rgb(255, 255, 0),
      rgb(0, 255, 0),
      rgb(0, 255, 255),
      rgb(0, 0, 255),
      rgb(255, 0, 255),
      rgb(255, 0, 0)
    );
  }

  .color-picker-slider-alpha {
    background-image: conic-gradient(
      rgba(0, 0, 0, 0.06) 0 25%,
      transparent 0 50%,
      rgba(0, 0, 0, 0.06) 0 75%,
      transparent 0
    );
    background-size: 8px 8px;
    border-radius: 4px;
    margin-bottom: 12px;
  }

  .color-picker-slider-alpha .color-picker-palette {
    position: relative;
    height: 8px;
    min-height: 8px;
  }

  .color-picker-slider-alpha .color-picker-handler {
    width: 12px;
    height: 12px;
  }

  .color-picker-slider-alpha .color-picker-opacity {
    position: absolute;
    inset: 0px;
    background: linear-gradient(
      to right,
      rgba(255, 0, 4, 0),
      rgb(22, 133, 255)
    );
  }

  .color-picker-color-block {
    position: relative;
    border-radius: 4px;
    width: 26px;
    height: 26px;
    box-shadow: inset 0 0 1px 0 rgba(0, 0, 0, 0.25);
    background-image: conic-gradient(
      rgba(0, 0, 0, 0.06) 0 25%,
      transparent 0 50%,
      rgba(0, 0, 0, 0.06) 0 75%,
      transparent 0
    );
    background-size: 50% 50%;
  }

  .color-picker-color-block-inner {
    width: 100%;
    height: 100%;
    border: 1px solid rgba(0, 0, 0, 0.06);
    border-radius: inherit;
  }

  .color-picker-selector {
    display: flex;
  }

  .color-picker-selector {
    margin-bottom: 12px;
  }

  .color-picker-selector-item {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 40px;
    border-bottom: 2px solid var(--divider, #e1e5e9);
    cursor: pointer;
    color: #2b5672;
  }

  .color-picker-selector-item-actived {
    border-bottom: 2px solid var(--primary, #1677ff);
  }

  .color-picker-input {
    display: flex;
    justify-content: space-between;
  }

  .color-picker-input-rgb,
  .color-picker-input-hsb {
    display: flex;
    gap: 4px;
  }
</style>
