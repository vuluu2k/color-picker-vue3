<script>
  import ColorCustom from './ColorCustom.vue'
  import ColorPicker from './ColorPicker.vue'
  import InputProgress from './InputProgress.vue'

  export default {
    emits: ['update:value', 'change'],
    components: {
      ColorCustom,
      ColorPicker,
      InputProgress,
    },
    props: {
      value: {
        type: String,
        default: '#000000',
      },
      more: {
        type: Boolean,
        default: false,
      },
      placement: {
        type: String,
        default: 'bottomRight',
      },
    },
    mounted() {
      this.initColor(this.value)
    },
    data() {
      return {
        show: false,
        view: 'ColorCustom',
        pickerLocation: {
          top: 0,
          left: 0,
        },
        startY: 0,
        startX: 0,
        alpha: 100,
        selection: null,
      }
    },
    methods: {
      async toggle(event) {
        this.show = !this.show
        await this.$nextTick()
        const picker = this.$refs.picker.getBoundingClientRect()
        if (this.show) {
          if (this.placement == 'bottomRight') {
            this.pickerLocation.top = picker.bottom + 2
            this.pickerLocation.left = picker.left
          } else if (this.placement == 'leftTop') {
            const pickerOverlay =
              this.$refs.pickerOverlay.getBoundingClientRect()
            this.pickerLocation.top = picker.bottom - pickerOverlay.height
            this.pickerLocation.left = picker.left - pickerOverlay.width - 10
          }
        }
      },
      async setView(view, value) {
        this.view = view
        await this.$nextTick()
      },
      handleColorChange(color, selection) {
        this.$emit('update:value', color)
        this.$emit('change', color, selection)
      },
      onMouseDownLocation(event) {
        const headerRect = this.$refs.pickerHeader.getBoundingClientRect()
        this.startY = event.clientY - headerRect.top
        this.startX = event.clientX - headerRect.left
        document.addEventListener('mousemove', this.onMouseMoveLocation)
        document.addEventListener('mouseup', this.onMouseUpLocation)
      },
      onMouseMoveLocation(event) {
        this.pickerLocation.top = event.clientY - this.startY
        this.pickerLocation.left = event.clientX - this.startX
      },
      onMouseUpLocation(event) {
        document.removeEventListener('mousemove', this.onMouseMoveLocation)
        document.removeEventListener('mouseup', this.onMouseUpLocation)
      },
      handleChangeHex(event) {
        const value = event.target.value
        const replaceValue = value.replaceAll('#', '')
        const getValue =
          replaceValue.length == 7
            ? replaceValue.slice(0, 6)
            : replaceValue.length >= 8
            ? replaceValue.slice(0, 8)
            : replaceValue
        const [_red, _green, _blue, alpha] = this.hexToRgb(getValue)
        this.alpha = Math.round((alpha / 255) * 100)
        this.handleColorChange(`#${replaceValue}`, this.selection)
      },
      handleChangeAlpha(value, selection) {
        const hexOpacity = Math.round((value / 100) * 255)
          .toString(16)
          .padStart(2, '0')
        const hex = `#${this.value.replace('#', '').slice(0, 6)}${hexOpacity}`
        this.handleColorChange(hex, selection)
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
      rgbaToHex(r, g, b, a) {
        const alpha = Math.round(a * 255)
          .toString(16)
          .padStart(2, '0')
        const hex = ((r << 16) + (g << 8) + b).toString(16).padStart(6, '0')
        return `#${hex}${alpha}`
      },
      savedSelection(event) {
        const sel = window.getSelection()
        if (
          sel.rangeCount !== 0 &&
          !sel.focusNode?.className.startsWith('picker-more')
        ) {
          this.selection = window.getSelection().getRangeAt(0)
        }
      },
      restoreSelection() {
        if (this.selection) {
          const sel = window.getSelection()
          sel.removeAllRanges()
          sel.addRange(this.selection)
          this.selection = null
        }
      },
      initColor(newValue) {
        if (newValue.startsWith('#')) {
          const replaceValue = newValue.replace('#', '')
          const getValue =
            replaceValue.length == 7
              ? replaceValue.slice(0, 6)
              : replaceValue.length >= 8
              ? replaceValue.slice(0, 8)
              : replaceValue
          const [_red, _green, _blue, alpha] = this.hexToRgb(getValue)
          this.alpha = Math.round((alpha / 255) * 100)
        } else if (this.more && newValue.startsWith('rgb')) {
          const rgbaRegex = /rgba?\((\d+),\s*(\d+),\s*(\d+),?\s*([\d\.]+)?\)/i
          const matches = newValue.match(rgbaRegex)

          if (matches) {
            const red = parseInt(matches[1])
            const green = parseInt(matches[2])
            const blue = parseInt(matches[3])
            const alpha = matches[4] ? parseFloat(matches[4]) : 1
            const hex = this.rgbaToHex(red, green, blue, alpha)
            this.handleColorChange(hex)
          }
        }
      },
    },
    beforeUnmount() {
      document.removeEventListener('mousemove', this.onMouseMoveLocation)
      document.removeEventListener('mouseup', this.onMouseUpLocation)
    },
  }
</script>

<template>
  <div>
    <div :class="{ 'picker-more': more }" tabindex="-1">
      <div
        @click.stop.prevent
        @mousedown.stop.prevent="toggle"
        class="picker-wrapper"
        ref="picker">
        <slot>
          <div class="picker" :style="{ background: value }"></div>
        </slot>
      </div>
      <div class="picker-more-info" v-if="more">
        <input
          class="picker-more-text"
          id="hex-color"
          type="text"
          :value="value"
          @change="handleChangeHex"
          @keydown.stop
          @keyup.stop
          @keypress.stop
          @mousedown="savedSelection"
          @mouseup.prevent
          @blur="restoreSelection" />
        <InputProgress v-model:value="alpha" @change="handleChangeAlpha" />
      </div>
    </div>

    <Teleport to="body">
      <div
        class="color-picker-custom"
        v-if="show"
        ref="pickerOverlay"
        :style="{
          position: 'absolute',
          top: `${pickerLocation.top}px`,
          left: `${pickerLocation.left}px`,
        }"
        @click.stop>
        <div
          class="color-picker-custom-header"
          @mousedown.stop.prevent="onMouseDownLocation"
          ref="pickerHeader">
          <div class="color-picker-custom-title" v-if="view == 'ColorPicker'">
            <span
              class="color-picker-custom-back"
              @mousedown.stop.prevent="setView('ColorCustom')">
              <svg
                viewBox="0 0 24 24"
                fill="currentColor"
                width="24"
                height="24">
                <path
                  d="M13.5 17.914 7.086 11.5 13.5 5.086 14.914 6.5l-5 5 5 5-1.414 1.414Z"></path>
              </svg>
            </span>
            <span> Color Picker </span>
          </div>
          <div class="color-picker-custom-title" v-if="view == 'ColorCustom'">
            Color Custom
          </div>
          <div
            class="color-picker-custom-close"
            @mousedown.stop.prevent="toggle">
            <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
              <path
                d="m13.59 12 4.344-4.342a.226.226 0 0 0 0-.318L16.66 6.067a.223.223 0 0 0-.318 0L12 10.409 7.658 6.067a.223.223 0 0 0-.318 0L6.066 7.34a.226.226 0 0 0 0 .318L10.409 12l-4.343 4.343a.226.226 0 0 0 0 .318l1.274 1.273c.088.088.23.088.318 0L12 13.59l4.343 4.343c.088.088.23.088.318 0l1.273-1.273a.226.226 0 0 0 0-.318L13.59 12Z"></path>
            </svg>
          </div>
        </div>
        <div class="color-picker-custom-content">
          <ColorPicker
            :value="value"
            :alpha="alpha"
            @change="handleColorChange"
            v-if="view == 'ColorPicker'"
            @setView="setView" />
          <ColorCustom
            :value="value"
            @setView="setView"
            @change="handleColorChange"
            v-if="view == 'ColorCustom'" />
        </div>
      </div>
    </Teleport>
  </div>
</template>

<style scoped>
  input {
    all: unset;
  }
  .picker-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .picker {
    width: 22px;
    height: 22px;
    border-radius: 4px;
    position: relative;
    border: 1px solid var(--divider, #e5e5e5);
  }
  .picker::before {
    width: 100%;
    height: 100%;
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
    background-image: repeating-conic-gradient(
      from 0deg,
      #fff 0 25%,
      #ededed 0 50%
    );
    background-size: 10px 10px;
  }
  .picker-more {
    display: flex;
    border: 1px solid var(--more-border-color, rgba(255, 255, 255, 0));
    width: fit-content;
    padding: 0 3px;
    height: 32px;
    border-radius: 4px;
  }
  .picker-more:hover,
  .picker-more:hover .picker-more-input-progress {
    --more-border-color: var(--divider, #e0e0e0);
  }
  .picker-more .picker-more-input-progress {
    border-left: 1px solid var(--more-border-color, rgba(255, 255, 255, 0));
  }
  .picker-more:focus-within,
  .picker-more:focus-within .picker-more-input-progress {
    --more-border-color: var(--primary, #2196f3);
  }
  .picker-more-info {
    display: flex;
    margin-left: 10px;
  }
  .picker-more-text {
    padding-right: 5px;
    width: 60px;
    height: 100%;
  }
  .color-picker-custom-header {
    padding: 6px 8px;
    border-bottom: 1px solid var(--divider, #e0e0e0);
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: move;
  }
  .color-picker-custom-close {
    cursor: pointer;
  }
  .color-picker-custom-title {
    display: flex;
    align-items: center;
    font-weight: bold;
  }
  .color-picker-custom-back {
    display: flex;
    align-items: center;
    cursor: pointer;
  }
  .color-picker-custom-content {
    padding: 8px;
  }
  .color-picker-custom {
    border-radius: 8px;
    z-index: 1;
    background-color: #fff;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
    z-index: 1000;
  }
</style>
