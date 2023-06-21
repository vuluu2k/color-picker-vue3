<script>
  export default {
    props: {
      value: {
        type: String,
        default: '#1677ff',
      },
    },
    data() {
      return {
        selector: 'hsb',
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
        hex: '',
        alpha: 1,
        clientXGradient: 0,
      }
    },
    mounted() {
      // const gradientRect = this.$refs.pickerGradient.getBoundingClientRect()
      const selectRect = this.$refs.pickerSelect.getBoundingClientRect()
      this.clientX = -8
      this.clientY = selectRect.height - 8
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
      backgroundBlock() {
        return `rgb(${this.rgb.red},${this.rgb.green},${this.rgb.blue})`
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
          alpha ? h & 0x000000ff : '',
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

        ;[this.hsb.hue, this.hsb.saturation, this.hsb.brightness] =
          this.rgbToHsb(red, green, blue)
        this.hex = `#${this.rgbToHex(red, green, blue)}`
      },
      handleHexInput(event) {
        const value = event.target.value
        const getValue =
          value.length == 7
            ? value.slice(0, 6)
            : value.length >= 8
            ? value.slice(0, 8)
            : value
        const [red, green, blue, alpha] = this.hexToRgb(getValue)
        this.rgb = { red, green, blue }
        this.alpha = Math.round((alpha / 255) * 100)
      },
      onMouseDown(event) {
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

      onMouseDownProgressGradient(event) {
        const gradientRect = this.$refs.pickerGradient.getBoundingClientRect()
        this.clientXGradient = event.clientX - gradientRect.left - 8
        this.updateHue()
        document.addEventListener('mousemove', this.onMouseMoveProgressGradient)
        document.addEventListener('mouseup', this.onMouseUpProgressGradient)
      },
      onMouseMoveProgressGradient(event) {
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
      onMouseUpProgressGradient(event) {
        document.removeEventListener(
          'mousemove',
          this.onMouseMoveProgressGradient
        )
        document.removeEventListener('mouseup', this.onMouseUpProgressGradient)
      },
    },
    beforeUnmount() {
      document.removeEventListener('mousemove', onMouseMove)
      document.removeEventListener('mouseup', onMouseUp)
      document.removeEventListener(
        'mousemove',
        this.onMouseMoveProgressGradient
      )
      document.removeEventListener('mouseup', this.onMouseUpProgressGradient)
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
    },
  }
</script>

<template>
  <div class="color-picker-panel">
    <div class="color-picker-select">
      <div
        class="color-picker-palette"
        @mousedown="onMouseDown"
        ref="pickerSelect"
      >
        <div
          :style="{
            position: 'absolute',
            left: leftSelect,
            top: topSelect,
            zIndex: 1,
          }"
        >
          <div
            class="color-picker-handler"
            :style="{ background: backgroundBlock }"
          />
        </div>
        <div
          class="color-picker-saturation"
          ref="pickerSaturation"
          :style="{
            'background-color': backgroundGradient,
            'background-image':
              'linear-gradient(0deg, rgb(0, 0, 0), transparent), linear-gradient(90deg, rgb(255, 255, 255), rgba(255, 255, 255, 0))',
          }"
        />
      </div>
    </div>

    <div class="color-picker-slider-container">
      <div class="color-picker-slider-group">
        <div class="color-picker-slider-hue">
          <div
            class="color-picker-palette"
            @mousedown="onMouseDownProgressGradient"
            ref="pickerGradient"
          >
            <div
              :style="{
                position: 'absolute',
                left: `${this.clientXGradient}px`,
                top: '-2px',
                zIndex: 1,
              }"
            >
              <div
                class="color-picker-handler"
                :style="{ backgroundColor: backgroundGradient }"
              ></div>
            </div>
            <div class="color-picker-gradient"></div>
          </div>
        </div>
        <div class="color-picker-slider-alpha">
          <div class="color-picker-palette">
            <div
              :style="{
                position: 'absolute',
                left: '110px',
                top: '-2px',
                zIndex: 1,
              }"
            >
              <div class="color-picker-handler"></div>
            </div>
            <div class="color-picker-opacity"></div>
          </div>
        </div>
      </div>
      <div class="color-picker-color-block">
        <div
          class="color-picker-color-block-inner"
          :style="{ background: backgroundBlock }"
        ></div>
      </div>
    </div>

    <div class="color-picker-input-container">
      <div class="color-picker-selector">
        <div
          class="color-picker-selector-item"
          :class="{ 'color-picker-selector-item-actived': selector === 'hex' }"
          @click="selector = 'hex'"
        >
          HEX
        </div>
        <div
          class="color-picker-selector-item"
          :class="{ 'color-picker-selector-item-actived': selector === 'rgb' }"
          @click="selector = 'rgb'"
        >
          RGB
        </div>
        <div
          class="color-picker-selector-item"
          :class="{ 'color-picker-selector-item-actived': selector === 'hsb' }"
          @click="selector = 'hsb'"
        >
          HSB
        </div>
      </div>
      <div class="color-picker-input">
        <div class="color-picker-input-type">
          <div class="color-picker-input-hex" v-if="selector === 'hex'">
            <input
              type="text"
              v-model="hex"
              name="hex"
              @input="handleHexInput"
            />
          </div>
          <div class="color-picker-input-rgb" v-if="selector === 'rgb'">
            <input
              type="number"
              min="0"
              max="255"
              name="red"
              v-model="rgb.red"
              @input="handleRgbInput"
            />
            <input
              type="number"
              min="0"
              max="255"
              name="green"
              v-model="rgb.green"
              @input="handleRgbInput"
            />
            <input
              type="number"
              min="0"
              max="255"
              name="blue"
              v-model="rgb.blue"
              @input="handleRgbInput"
            />
          </div>
          <div class="color-picker-input-hsb" v-if="selector === 'hsb'">
            <input
              name="hue"
              type="number"
              min="0"
              max="360"
              v-model="hsb.hue"
              @input="handleHsbInput"
            />
            <input
              name="saturation"
              type="number"
              min="0"
              max="100"
              v-model="hsb.saturation"
              @input="handleHsbInput"
            />
            <input
              name="brightness"
              type="number"
              min="0"
              max="100"
              v-model="hsb.brightness"
              @input="handleHsbInput"
            />
          </div>
        </div>
        <div class="color-picker-input-alpha">
          <input type="number" value="100" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  div {
    margin: 0;
    padding: 0;
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
    width: 42px;
  }

  input:focus {
    border-color: var(--primary, #1677ff);
  }

  .color-picker-panel {
    display: flex;
    flex-direction: column;
    width: 240px;
    height: 236px;
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
    width: 12px;
    height: 12px;
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
    width: 8px;
    height: 8px;
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
    width: 8px;
    height: 8px;
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
