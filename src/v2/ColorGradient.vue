<script>
  import { useSiteStore } from '@/stores/site'
  import ColorPickerCustom from './ColorPickerCustom.vue'

  export default {
    emits: ['change'],
    components: {
      ColorPickerCustom,
    },
    setup() {
      const siteStore = useSiteStore()
      return { siteStore }
    },
    props: {
      value: {
        type: String,
        default:
          'linear-gradient(90deg, rgba(145, 133, 122, 1) 0%, rgba(242, 222, 204, 1) 100%)',
      },
    },
    mounted() {
      this.handleChangeValue(this.value)
      if (!this.value.includes('gradient')) {
        this.$emit(
          'change',
          'linear-gradient(90deg, rgba(145, 133, 122, 1) 0%, rgba(242, 222, 204, 1) 100%)'
        )
      }
    },
    data() {
      return {
        degree: 0,
        centalPoint: {
          x: 0,
          y: 0,
        },
        gradientType: 'linear',
        gradientColorsDefault: [
          'linear-gradient(90deg, rgba(145, 133, 122, 1) 0%, rgba(242, 222, 204, 1) 100%)',
          'linear-gradient(50.4988deg, rgba(127, 148, 99, 1) 17.0105%, rgba(242, 222, 204, 1) 48.2697%, rgba(56, 74, 211, 1) 80.9021%)',
          'radial-gradient(circle at 50% 50%, rgba(56, 74, 211, 1) 15.0635%, rgba(232, 234, 237, 1) 80.2887%)',
          'radial-gradient(circle at 32% 26%, rgba(209, 219, 195, 1) 0%, rgba(127, 148, 99, 1) 36.0321%, rgba(37, 49, 141, 1) 75.6775%)',
          'conic-gradient(from 152deg at 50% 50%, rgba(0, 87, 225, 1) 0%, rgba(249, 197, 180, 1) 68%, rgba(0, 87, 225, 1) 100%)',
          'conic-gradient(from 136deg at 50% 0%, rgba(85, 14, 155, 1) 0%, rgba(128, 21, 232, 1) 6%, rgba(243, 167, 143, 1) 12%, rgba(85, 14, 155, 1) 28%)',
        ],
        selectedColorIndex: 0,
        processColors: [
          { color: 'rgba(145, 133, 122, 1)', x: 0 },
          { color: 'rgba(242, 222, 204, 1)', x: 128 },
        ],
        radialPointer: {
          x: 120,
          y: 60,
        },
        conicPointer: {
          degree: 0,
          x: 120,
          y: 60,
          key: '',
          central: {
            x: 0,
            y: 0,
          },
        },
      }
    },
    computed: {
      gradientColors() {
        return (this.siteStore.getSettings?.gradients || []).concat(
          this.gradientColorsDefault
        )
      },
      colorsPreview() {
        const colors = this.processColors.map((process) => {
          const processPercent = Math.round((process.x / 128) * 100)
          return `${process.color} ${processPercent}%`
        })

        return colors.join(', ').trim()
      },
      gradientPreview() {
        const gradient =
          this.gradientType == 'linear'
            ? `linear-gradient(${this.degree}deg`
            : this.gradientType == 'radial'
            ? `radial-gradient(circle at ${
                (this.radialPointer.x / 240) * 100
              }% ${(this.radialPointer.y / 120) * 100}%`
            : `conic-gradient(from ${this.conicPointer.degree}deg at ${
                (this.conicPointer.x / 240) * 100
              }% ${(this.conicPointer.y / 120) * 100}%`

        return `${gradient}, ${this.colorsPreview})`
      },
      alpha() {
        const alpha = Math.round(
          Number(
            this.processColors[this.selectedColorIndex].color
              .split(',')[3]
              .replace(')', '')
          ) * 100
        )
        return alpha
      },
    },
    methods: {
      onMouseDownGradient(event) {
        const previewRect = this.$refs.gradientPreview.getBoundingClientRect()
        if (this.gradientType == 'linear') {
          this.centalPoint.x = previewRect.left + previewRect.width / 2
          this.centalPoint.y = previewRect.top + previewRect.height / 2
          this.degree = this.handleRoundCentral(
            event.clientX,
            event.clientY,
            this.centalPoint.x,
            this.centalPoint.y
          )
        } else if (this.gradientType == 'radial') {
          this.radialPointer.x = event.clientX - previewRect.left
          this.radialPointer.y = event.clientY - previewRect.top
        } else if (this.gradientType == 'conic') {
          const conicPointerRect =
            this.$refs.conicPointer.getBoundingClientRect()
          if (event.target.id == 'conic-slider') {
            this.conicPointer.central.x =
              conicPointerRect.left + conicPointerRect.width / 2
            this.conicPointer.central.y =
              conicPointerRect.top + conicPointerRect.height / 2
            this.conicPointer.degree = this.handleRoundCentral(
              event.clientX,
              event.clientY,
              this.conicPointer.central.x,
              this.conicPointer.central.y
            )
            this.conicPointer.key = 'conic-slider'
          } else {
            this.conicPointer.x = event.clientX - previewRect.left
            this.conicPointer.y = event.clientY - previewRect.top
            this.conicPointer.key = 'conic-pointer'
          }
        }
        document.addEventListener('mousemove', this.onMouseMoveGradient)
        document.addEventListener('mouseup', this.onMouseUpGradient)
      },
      onMouseMoveGradient(event) {
        if (this.gradientType == 'linear') {
          this.degree = this.handleRoundCentral(
            event.clientX,
            event.clientY,
            this.centalPoint.x,
            this.centalPoint.y
          )
        } else if (this.gradientType == 'radial') {
          const previewRect = this.$refs.gradientPreview.getBoundingClientRect()
          if (
            previewRect.top < event.clientY &&
            event.clientY < previewRect.bottom
          ) {
            this.radialPointer.y = event.clientY - previewRect.top
          }
          if (
            previewRect.left < event.clientX &&
            event.clientX < previewRect.right
          ) {
            this.radialPointer.x = event.clientX - previewRect.left
          }
          if (previewRect.right <= event.clientX) {
            this.radialPointer.x = previewRect.width
          }
          if (previewRect.left >= event.clientX) {
            this.radialPointer.x = 0
          }
          if (previewRect.bottom <= event.clientY) {
            this.radialPointer.y = previewRect.height
          }
          if (previewRect.top >= event.clientY) {
            this.radialPointer.y = 0
          }
        } else if (this.gradientType == 'conic') {
          if (this.conicPointer.key == 'conic-slider')
            this.conicPointer.degree = this.handleRoundCentral(
              event.clientX,
              event.clientY,
              this.conicPointer.central.x,
              this.conicPointer.central.y
            )
          else if (this.conicPointer.key == 'conic-pointer') {
            const previewRect =
              this.$refs.gradientPreview.getBoundingClientRect()
            if (
              previewRect.top < event.clientY &&
              event.clientY < previewRect.bottom
            ) {
              this.conicPointer.y = event.clientY - previewRect.top
            }
            if (
              previewRect.left < event.clientX &&
              event.clientX < previewRect.right
            ) {
              this.conicPointer.x = event.clientX - previewRect.left
            }
            if (previewRect.right <= event.clientX) {
              this.conicPointer.x = previewRect.width
            }
            if (previewRect.left >= event.clientX) {
              this.conicPointer.x = 0
            }
            if (previewRect.bottom <= event.clientY) {
              this.conicPointer.y = previewRect.height
            }
            if (previewRect.top >= event.clientY) {
              this.conicPointer.y = 0
            }
          }
        }
      },
      onMouseUpGradient(event) {
        if (this.gradientType == 'conic') {
          this.conicPointer.key = ''
        }
        document.removeEventListener('mousemove', this.onMouseMoveGradient)
        document.removeEventListener('mouseup', this.onMouseUpGradient)
      },
      onMouseDownProcessColor(event) {
        const colorPickerCustom = this.$refs.colorPickerCustom
        if (event.target.id !== 'processPointer') {
          if(colorPickerCustom.show == false){
            colorPickerCustom.pickerLocation.left = event.clientX - 60
            colorPickerCustom.pickerLocation.top = event.clientY + 20
          }
          colorPickerCustom.show = true
          const processRect = processColor.getBoundingClientRect()
          const processX = event.clientX - processRect.left
          const newColor = { color: colorPickerCustom.value, x: processX }

          let insertIndex = -1
          for (let i = 0; i < this.processColors.length; i++) {
            if (this.processColors[i].x === newColor.x) {
              insertIndex = i
              break
            } else if (this.processColors[i].x > newColor.x) {
              insertIndex = i
              break
            }
          }

          if (insertIndex === -1) {
            this.processColors.push(newColor)
            this.selectedColorIndex = this.processColors.length - 1
          } else {
            this.processColors.splice(insertIndex, 0, newColor)
            this.selectedColorIndex = insertIndex
          }
        } else {
          document.addEventListener('mousemove', this.onMouseMoveProcessColor)
          document.addEventListener('mouseup', this.onMouseUpProcessColor)
        }
      },
      onMouseMoveProcessColor(event) {
        const processRect = processColor.getBoundingClientRect()
        if (
          processRect.left <= event.clientX &&
          event.clientX <= processRect.right
        ) {
          const processX = event.clientX - processRect.left
          this.processColors[this.selectedColorIndex].x = processX
        }
        if (event.clientX <= processRect.left) {
          this.processColors[this.selectedColorIndex].x = 0
        }
        if (event.clientX >= processRect.right) {
          this.processColors[this.selectedColorIndex].x = processRect.width
        }
      },
      onMouseUpProcessColor(event) {
        document.removeEventListener('mousemove', this.onMouseMoveProcessColor)
        document.removeEventListener('mouseup', this.onMouseUpProcessColor)
      },
      onMouseDownOpacity(event) {
        const opacityRect = this.$refs.opacityColor.getBoundingClientRect()
        const alpha = (event.clientX - opacityRect.left) / opacityRect.width
        this.processColors[this.selectedColorIndex].color = this.replaceAlpha(
          this.processColors[this.selectedColorIndex].color,
          alpha.toFixed(2)
        )
        document.addEventListener('mousemove', this.onMouseMoveOpacity)
        document.addEventListener('mouseup', this.onMouseUpOpacity)
      },
      onMouseMoveOpacity(event) {
        const opacityRect = this.$refs.opacityColor.getBoundingClientRect()
        let alpha

        if (
          opacityRect.left <= event.clientX &&
          event.clientX <= opacityRect.right
        ) {
          alpha = (event.clientX - opacityRect.left) / opacityRect.width
        }
        if (event.clientX <= opacityRect.left) {
          alpha = 0
        }
        if (event.clientX >= opacityRect.right) {
          alpha = 1
        }
        this.processColors[this.selectedColorIndex].color = this.replaceAlpha(
          this.processColors[this.selectedColorIndex].color,
          alpha.toFixed(2)
        )
      },
      onMouseUpOpacity(event) {
        document.removeEventListener('mousemove', this.onMouseMoveOpacity)
        document.removeEventListener('mouseup', this.onMouseUpOpacity)
      },
      replaceAlpha(rgbaString, newAlpha) {
        return rgbaString.replace(/[^,]+(?=\))/, newAlpha)
      },
      handleRoundCentral(clientX, clientY, x, y) {
        const dx = clientX - x
        const dy = clientY - y
        let angle = Math.round((Math.atan2(dx, -dy) * 180) / Math.PI)
        if (angle < 0) angle += 360
        return angle
      },
      handleChangeColor(color) {
        this.processColors[this.selectedColorIndex].color = color
      },
      handleChangeAlpha(event) {
        const alpha = event.target.value / 100
        this.processColors[this.selectedColorIndex].color = this.replaceAlpha(
          this.processColors[this.selectedColorIndex].color,
          alpha.toFixed(2)
        )
      },
      handleFlipColor() {
        this.processColors = this.processColors
          .reverse()
          .map((process) => ({ ...process, x: 128 - process.x }))
      },
      handleDeleteColor() {
        if (this.processColors.length > 2) this.processColors.pop()
      },
      handleChangeValue(valueChange) {
        if (valueChange.startsWith('linear-gradient')) {
          const { degree, colors } = this.parseLinearGradient(valueChange)
          this.degree = degree
          this.processColors = colors
          this.gradientType = 'linear'
        } else if (valueChange.startsWith('radial-gradient')) {
          const { position, colors } = this.parseRadialGradient(valueChange)
          this.radialPointer = position
          this.processColors = colors
          this.gradientType = 'radial'
        } else if (valueChange.startsWith('conic-gradient')) {
          const { degree, position, colors } =
            this.parseConicGradient(valueChange)
          this.conicPointer.degree = degree
          this.conicPointer.x = position.x
          this.conicPointer.y = position.y
          this.processColors = colors
          this.gradientType = 'conic'
        }
      },
      handleAddTemplate() {
        const existing = this.gradientColors.find(
          (gradient) => gradient == this.gradientPreview
        )
        if (!existing)
          this.siteStore.updateSite(this.$route.params.site_id, {
            settings: {
              ...this.siteStore.getSettings,
              gradients: [this.gradientPreview].concat(
                this.siteStore.getSettings?.gradients || []
              ),
            },
          })
      },
      parseLinearGradient(gradientString) {
        const degree = gradientString.match(
          /-?\d+(\.\d+)?(deg|grad|rad|turn)/
        )[0]

        const colors = gradientString
          .match(/rgba?\([\d\s,]+\)\s\d+(\.\d+)?%/g)
          .map((colorString) => {
            const [color, percent] = colorString.match(
              /rgba?\([\d\s,]+\)|\d+(\.\d+)?%/g
            )
            return {
              color,
              x: (parseFloat(percent) / 100) * 128,
            }
          })

        return { degree: parseFloat(degree), colors }
      },
      parseRadialGradient(gradientString) {
        const [position, shape] = gradientString
          .match(/(circle|ellipse) at [\d.]+% [\d.]+%/)
          .map((str) => str.split(' '))

        const colors = gradientString
          .match(/rgba?\([\d\s,]+\)\s\d+(\.\d+)?%/g)
          .map((colorString) => {
            const [color, percent] = colorString.match(
              /rgba?\([\d\s,]+\)|\d+(\.\d+)?%/g
            )
            return {
              color,
              x: (parseFloat(percent) / 100) * 128,
            }
          })
        return {
          shape: shape[0],
          position: {
            x: (parseFloat(position[2]) / 100) * 240,
            y: (parseFloat(position[3]) / 100) * 120,
          },
          colors,
        }
      },
      parseConicGradient(gradientString) {
        const [[angle, _, x, y]] = gradientString
          .match(/-?\d+(\.\d+)?deg at [\d.]+% [\d.]+%/)
          .map((str) => str && str.split(' '))

        const colors = gradientString
          .match(/rgba?\([\d\s,]+\)\s\d+(\.\d+)?%/g)
          .map((colorString) => {
            const [color, percent] = colorString.match(
              /rgba?\([\d\s,]+\)|\d+(\.\d+)?%/g
            )
            return {
              color,
              x: (parseFloat(percent) / 100) * 128,
            }
          })

        return {
          degree: parseFloat(angle),
          position: { x: parseFloat(x), y: parseFloat(y) },
          colors,
        }
      },
    },
    watch: {
      gradientPreview() {
        this.$emit('change', this.gradientPreview)
      },
    },
    beforeUnmount() {
      document.removeEventListener('mousemove', this.onMouseMoveGradient)
      document.removeEventListener('mouseup', this.onMouseUpGradient)
      document.removeEventListener('mousemove', this.onMouseMoveProcessColor)
      document.removeEventListener('mouseup', this.onMouseUpProcessColor)
      document.removeEventListener('mousemove', this.onMouseMoveOpacity)
      document.removeEventListener('mouseup', this.onMouseUpOpacity)
    },
  }
</script>

<template>
  <div class="color-gradient">
    <div class="color-gradient-select">
      <select
        v-model="gradientType"
        style="width: 196px; margin: 0 22px; padding: 0 12px">
        <option value="linear">Linear</option>
        <option value="radial">Radial</option>
        <option value="conic">Conic</option>
      </select>
    </div>
    <div class="color-gradient-container">
      <div
        class="color-gradient-preview"
        ref="gradientPreview"
        @mousedown.stop.prevent="onMouseDownGradient"
        :style="{
          'background-image': gradientPreview,
        }">
        <div
          class="color-gradient-preview-action"
          v-if="gradientType == 'linear'">
          <div class="color-gradient-preview-overlay"></div>
          <div
            class="color-gradient-preview-slider"
            :style="{ transform: `rotate(${degree}deg)` }"></div>
          <div class="color-gradient-preview-degree">{{ degree }}</div>
        </div>
        <div
          class="color-gradient-preview-radial"
          v-if="gradientType == 'radial'">
          <div
            class="color-gradient-preview-radial-pointer"
            :style="{
              transform: `translateX(${radialPointer.x}px) translateY(${radialPointer.y}px)`,
            }"></div>
        </div>
        <div
          class="color-gradient-preview-conic"
          v-if="gradientType == 'conic'"
          :style="{
            transform: `translateX(${conicPointer.x}px) translateY(${conicPointer.y}px)`,
          }">
          <div
            class="color-gradient-preview-conic-slider"
            :data-active="conicPointer.key == 'conic-slider'"
            id="conic-slider"></div>
          <div
            class="color-gradient-preview-conic-pointer"
            id="conic-pointer"
            ref="conicPointer"
            :style="{ transform: `rotate(${conicPointer.degree}deg)` }"></div>
        </div>
      </div>

      <div class="color-gradient-change">
        <div class="color-gradient-process">
          <div
            class="color-gradient-processing"
            :style="{
              'background-image': `linear-gradient(90deg, ${this.colorsPreview})`,
            }">
            <div
              class="color-gradient-processing-overlay"
              @mousedown.prevent="onMouseDownProcessColor"
              ref="processColor"
              id="processColor">
              <div
                class="color-gradient-processing-node"
                v-for="(processColor, index) in processColors"
                :style="{
                  '--handle-color': processColor.color,
                  '--handle-translate-x': `${processColor.x}px`,
                  '--handle-translate-y': 0,
                }"
                :data-selected="selectedColorIndex == index"
                :key="index"
                @mousedown.prevent="selectedColorIndex = index"
                id="processPointer" />
            </div>
          </div>
          <div
            class="color-gradient-flip-color"
            @mousedown.prevent="handleFlipColor">
            <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
              <path
                d="M7.70710678,8 L9.85355339,10.1464466 C10.0488155,10.3417088 10.0488155,10.6582912 9.85355339,10.8535534 C9.65829124,11.0488155 9.34170876,11.0488155 9.14644661,10.8535534 L5.79289322,7.5 L9.14644661,4.14644661 C9.34170876,3.95118446 9.65829124,3.95118446 9.85355339,4.14644661 C10.0488155,4.34170876 10.0488155,4.65829124 9.85355339,4.85355339 L7.70710678,7 L12,7 C15.3137085,7 18,9.6862915 18,13 L17,13 C17,10.2385763 14.7614237,8 12,8 L7.70710678,8 Z M16.2928932,16 L14.1464466,13.8535534 C13.9511845,13.6582912 13.9511845,13.3417088 14.1464466,13.1464466 C14.3417088,12.9511845 14.6582912,12.9511845 14.8535534,13.1464466 L18.2071068,16.5 L14.8535534,19.8535534 C14.6582912,20.0488155 14.3417088,20.0488155 14.1464466,19.8535534 C13.9511845,19.6582912 13.9511845,19.3417088 14.1464466,19.1464466 L16.2928932,17 L12,17 C8.6862915,17 6,14.3137085 6,11 L7,11 C7,13.7614237 9.23857625,16 12,16 L16.2928932,16 Z"></path>
            </svg>
          </div>
          <div
            class="color-gradient-delete-color"
            @mousedown.prevent="handleDeleteColor">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24">
              <g>
                <path
                  fill="#000000"
                  d="M16 16c0 1.104-.896 2-2 2h-4c-1.104 0-2-.896-2-2V8h8v8zm-6-9.5c0-.271.24-.5.525-.5h2.867c.301 0 .608.252.608.5V7h-4v-.5zm5 .5v-.5c0-.799-.752-1.5-1.608-1.5h-2.867C9.684 5 9 5.673 9 6.5V7H6v1h1v8c0 1.657 1.343 3 3 3h4c1.657 0 3-1.343 3-3V8h1V7h-3zm-2 9h1v-6h-1v6zm-3 0h1v-6h-1v6z"></path>
              </g>
            </svg>
          </div>
        </div>
      </div>

      <div
        class="color-gradient-custom"
        :style="{ '--color': processColors[selectedColorIndex].color }">
        <div
          ref="opacityColor"
          class="color-gradient-opacity"
          :style="{
            '--range-input-value': alpha / 100,
            '--range-input-width': '80px',
          }"
          @mousedown.prevent="onMouseDownOpacity">
          <span class="color-gradient-opacity-range"></span>
        </div>
        <div class="color-gradient-opacity-input">
          <input
            type="number"
            min="0"
            max="100"
            step="1"
            v-model="alpha"
            @input="handleChangeAlpha" />
          <span>%</span>
        </div>
        <ColorPickerCustom
          :value="processColors[selectedColorIndex].color"
          @change="handleChangeColor"
          output="rgb"
          ref="colorPickerCustom" />
      </div>

      <div class="color-gradient-add">
        <div class="color-gradient-add-title">
          <div class="color-gradient-add-title-text">My color gradients</div>
          <div
            class="color-gradient-add-title-add"
            @mousedown.prevent="handleAddTemplate">
            + Add
          </div>
        </div>

        <div class="color-gradient-add-preview">
          <div
            class="color-gradient-add-preview-item"
            v-for="(gradient, index) in gradientColors"
            :key="index"
            @mousedown.prevent="handleChangeValue(gradient)">
            <div
              class="color-gradient-add-preview-item-inner"
              :style="{ 'background-image': gradient }"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  select {
    outline: none;
    padding: 0 6px;
    height: 28px;
    border-radius: 4px;
    color: #2b5672;
  }
  select:focus {
    border-color: var(--primary, #1677ff);
  }

  input {
    -webkit-font-smoothing: antialiased;
    box-sizing: border-box;
    height: 32px;
    border: 1px solid transparent;
    padding: 0;
    outline: none;
    border-radius: 8px;
    color: #162d3d;
    text-align: center;
    text-overflow: clip;
    width: 100%;
    font-size: 14px;
    line-height: normal;
    padding-right: calc(14px * 1.3);
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

  .color-gradient {
    width: fit-content;
  }
  .color-gradient-select {
    display: flex;
    align-items: center;
    justify-content: center;
    width: fit-content;
    margin-bottom: 12px;
  }

  .color-gradient-preview {
    position: relative;
    width: 240px;
    height: 120px;
    border-radius: 2px;
  }

  .color-gradient-preview::before {
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

  .color-gradient-preview-action {
    width: 100%;
    height: 100%;
    -webkit-user-select: none;
    -moz-user-select: none;
    user-select: none;
    display: grid;
    grid-template: 1fr/1fr;
    grid-template-areas: 'content';
  }
  .color-gradient-preview-overlay {
    place-self: stretch;
    grid-area: content;
  }
  .color-gradient-preview-slider {
    place-self: center;
    grid-area: content;
    position: relative;
    width: calc(2 * 30px);
    height: calc(2 * 30px);
    border: 4px solid hsla(0, 0%, 100%, 0.6);
    box-shadow: var(
      0 0 2px 0 rgba(14, 15, 17, 0.3),
      inset 0 0 1px 0 rgba(14, 15, 17, 0.3)
    );
    background-color: rgba(23, 25, 28, 0.04);
    pointer-events: none;
    border-radius: 50%;
    transform: rotate(0deg);
  }
  .color-gradient-preview-slider::after {
    content: '';
    display: block;
    width: 16px;
    height: 16px;
    margin-left: 18px;
    margin-top: -9px;
    box-shadow: 0 1px 3px 0 rgba(14, 15, 17, 0.3);
    border: solid 4px #fff;
    background-color: var(--primary, #1f77ff);
    border-radius: 50%;
  }
  .color-gradient-preview-degree {
    color: var(--handle-text-color, #fff);
    text-shadow: var(--handle-text-shadow, 0 0 2px rgba(0, 0, 0, 0.5));
    place-self: center;
    grid-area: content;
    pointer-events: none;
    font: var(--handle-text-font, 1em sans-serif);
    font-weight: 900;
    margin-left: 0.33em;
  }
  .color-gradient-preview-degree::after {
    content: var(--handle-text-deg-sign, 'º');
    color: var(--handle-text-color, #fff);
    text-shadow: var(--handle-text-shadow, 0 0 2px rgba(0, 0, 0, 0.5));
  }

  .color-gradient-process {
    padding: 18px 24px 16px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .color-gradient-processing {
    border-radius: 6px;
    --height: 12px;
    --width: 128px;
  }
  .color-gradient-processing:first-child {
    margin: 0 14px 0 0;
    cursor: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' width='24' height='18.8' viewBox='5.5 4.3 26.6 20.9'%3E%3Cdefs%3E%3Cfilter id='a' width='158.3%25' height='209.8%25' x='-29.9%25' y='-46%25' filterUnits='objectBoundingBox'%3E%3CfeMorphology in='SourceAlpha' operator='dilate' radius='1' result='shadowSpreadOuter1'/%3E%3CfeOffset dy='1' in='shadowSpreadOuter1' result='shadowOffsetOuter1'/%3E%3CfeGaussianBlur in='shadowOffsetOuter1' result='shadowBlurOuter1' stdDeviation='1.5'/%3E%3CfeComposite in='shadowBlurOuter1' in2='SourceAlpha' operator='out' result='shadowBlurOuter1'/%3E%3CfeColorMatrix in='shadowBlurOuter1' values='0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.25 0'/%3E%3C/filter%3E%3Cpath id='b' d='M1.964 6.253l12 6.446-4.464 1.55 3.136 3.922-1.417 1.082-3.136-3.922-2.62 3.857L1.964 6.253zM19.93 7.885l.827 2.884 2.884-.827.275.961-2.884.827.827 2.884-.96.275-.829-2.883-2.883.827-.275-.962 2.884-.827-.827-2.883.96-.276z'/%3E%3C/defs%3E%3Cg fill='%23000' transform='rotate(16 8.825 35.1)' fill-rule='evenodd'%3E%3Cuse filter='url(%23a)' xlink:href='%23b'/%3E%3Cpath stroke='%23FFF' stroke-opacity='.8' d='M1.179 5.263l14.026 7.534-4.873 1.692 3.014 3.77-2.212 1.688-3.019-3.775-2.85 4.198L1.179 5.263zm19.093 2.004l.827 2.883 2.883-.827.552 1.923-2.885.827.828 2.884-1.923.55-.827-2.883-2.883.827-.552-1.923 2.884-.826-.827-2.884 1.923-.551z'/%3E%3C/g%3E%3C/svg%3E"),
      pointer;
  }
  .color-gradient-processing-overlay {
    width: 128px;
    height: 12px;
    position: relative;
  }
  .color-gradient-processing-node {
    z-index: 1;
    width: var(--handle-selected-width, 19px);
    height: var(--handle-selected-height, 19px);
    margin-left: calc(-1 * var(--handle-selected-width, 19px) / 2);
    margin-top: calc(-1 * var(--handle-selected-height, 19px) / 2);
    box-shadow: var(
      --handle-selected-shadow,
      0 1px 3px 0 rgba(23, 25, 28, 0.5)
    );
    border: var(--handle-selected-border, solid 4px #fff);
    transform: translate(
      var(--handle-translate-x, 0),
      var(--handle-translate-y, 0)
    );
    position: absolute;
    top: var(--handle-offset-top, 5px);
    box-sizing: border-box;
    width: var(--handle-width, 17px);
    height: var(--handle-height, 17px);
    margin-left: calc(-1 * var(--handle-width, 17px) / 2);
    margin-top: calc(-1 * var(--handle-height, 17px) / 2);
    background-color: #fff;
    box-shadow: var(--handle-shadow, 0 1px 3px 0 rgba(23, 25, 28, 0.3));
    border: var(--handle-border, solid 2px #fff);
    border-radius: 50%;
    cursor: grab;
  }
  .color-gradient-processing-node::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: var(--handle-color, #fff);
    border-radius: 50%;
  }
  .color-gradient-processing-node[data-selected='true']::after {
    content: '';
    position: absolute;
    top: 28px;
    left: -1px;
    width: 14px;
    height: 14px;
    background: #fff;
    transform: rotate(45deg);
    border-top: 1px solid #aec0cf;
    border-left: 1px solid #aec0cf;
    border-radius: 1px 0 0 0;
  }
  .color-gradient-flip-color,
  .color-gradient-delete-color {
    width: 24px;
    height: 24px;
    cursor: default;
    border-radius: 2px;
  }
  .color-gradient-delete-color {
    background-color: #f3f3f3;
    cursor: default;
  }
  .color-gradient-custom {
    display: flex;
    align-items: center;
    justify-content: space-between;
    --range-input-track-background: linear-gradient(
      90deg,
      transparent,
      var(--color)
    );
    border-top: 1px solid #d8e1e8;
    padding: 10px 24px;
    border-top: 1px solid #d8e1e8;
    border-bottom: 1px solid #d8e1e8;
  }
  .color-gradient-opacity {
    position: relative;
    box-sizing: border-box;
    width: var(--range-input-width);
    height: 8px;
    margin-right: 4px;
    border-radius: 4px;
    border: 1px solid #d5d5d5;
    cursor: pointer;
    box-shadow: inset 0 1px 0 0 rgba(0, 0, 0, 0.05);
    background: var(--range-input-track-background, #d6e6ff);
  }
  .color-gradient-opacity::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    z-index: -1;
    background-image: repeating-conic-gradient(
      from 0deg,
      #fff 0 25%,
      #ededed 0 50%
    );
    background-size: 8px 8px;
  }
  .color-gradient-opacity::after {
    width: calc(var(--range-input-value) * var(--range-input-width) - 8px);
    border-radius: 4px 0 0 4px;
    background-color: transparent;
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
  }
  .color-gradient-opacity-range {
    box-sizing: border-box;
    cursor: pointer;
    position: absolute;
    top: -5.33333px;
    left: 0;
    width: 16px;
    height: 16px;
    transform: translateX(
      calc(var(--range-input-value) * var(--range-input-width) - 8px)
    );
    border-radius: 50%;
    border: 4px solid #fff;
    background-color: #116dff;
    box-shadow: 0 1px 3px 0 rgba(43, 86, 114, 0.43);
  }
  .color-gradient-select-color {
    background-color: var(--color);
    width: 30px;
    height: 30px;
    border: 1px solid #b6c1cd;
    border-radius: 8px;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .color-gradient-add-title {
    grid-area: my-gradients-title;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 18px 22px 10px;
    gap: 1em;
    min-width: 0;
    font-size: 12px;
  }
  .color-gradient-add-title-add {
    color: var(--primary, #116dff);
    cursor: pointer;
  }
  .color-gradient-add-preview {
    grid-area: my-gradients;
    padding: 0 14px 0 18px;
    max-height: 96px;
    display: flex;
    flex-wrap: wrap;
    margin: 3px 0;
    gap: 6px 0;
  }
  .color-gradient-add-preview-item {
    width: 26px;
    height: 26px;
    text-align: center;
    flex: 0 1 calc(100% / 6);
  }
  .color-gradient-add-preview-item-inner {
    width: 22px;
    height: 22px;
    margin: 2px auto;
    border-radius: 50%;
    box-shadow: inset 0 0 0 1px hsla(0, 0%, 78.4%, 0.5);
    transition: 0.2s;
  }
  .color-gradient-add-preview-item-inner:hover {
    cursor: pointer;
    margin: 0 auto;
    width: 26px;
    height: 26px;
  }
  .color-gradient-opacity-input {
    display: flex;
    width: 50px;
    min-width: 50px;
    align-items: center;
    position: relative;
  }
  .color-gradient-opacity-input > span {
    position: absolute;
    margin-left: calc(var(--font-size) * -1.2);
    font-family: sans-serif;
    font-size: calc(var(--font-size) * 0.9);
    -webkit-font-smoothing: antialiased;
    pointer-events: none;
    right: 6px;
    top: 9px;
    font-size: 12px;
  }

  .color-gradient-preview-radial {
    width: 240px;
    height: 120px;
    position: relative;
    --handle-width: 12px;
    --handle-height: 12px;
    --handle-shadow: 0 1px 3px 0 rgba(43, 86, 114, 0.73);
    --handle-border: none;
    --handle-color: #fff;
  }
  .color-gradient-preview-radial-pointer {
    position: absolute;
    width: var(--handle-width, 17px);
    height: var(--handle-height, 17px);
    margin-left: calc(-1 * var(--handle-width, 17px) / 2);
    margin-top: calc(-1 * var(--handle-height, 17px) / 2);
    box-shadow: var(--handle-shadow, 0 2px 4px 0 rgba(0, 0, 0, 0.5));
    border: var(--handle-border, solid 2px #fff);
    background-color: var(--handle-color, #1f77ff);
    border-radius: 50%;
    box-sizing: border-box;
  }
  .color-gradient-preview-conic {
    width: var(--angle-slider-diameter, 60px);
    height: var(--angle-slider-diameter, 60px);
    background-color: transparent;
    position: absolute;
    top: calc(var(--angle-slider-diameter, 60px) / -2);
    left: calc(var(--angle-slider-diameter, 60px) / -2);
    display: grid;
    grid-template-areas: 'wrapper';
    place-items: center;
  }
  .color-gradient-preview-conic-slider {
    grid-area: wrapper;
    box-sizing: border-box;
    width: var(--angle-slider-diameter, 60px);
    height: var(--angle-slider-diameter, 60px);
    background-color: var(--angle-slider-bg-color, hsla(0, 0%, 100%, 0.2));
    border: var(--angle-slider-border, 3px solid hsla(0, 0%, 100%, 0.5));
    border-radius: 50%;
    box-shadow: var(
      --angle-slider-shadow,
      0 0 3px rgba(0, 0, 0, 0.3),
      inset 0 0 2px rgba(0, 0, 0, 0.1)
    );
    opacity: 0;
    transition: opacity var(--animation-enter, 0.2s ease-in-out 0.05s);
    cursor: grab;
    z-index: 1;
  }
  .color-gradient-preview-conic-slider[data-active='true'],
  .color-gradient-preview-conic-slider[data-active='true'] + ::after,
  .color-gradient-preview-conic-slider:hover,
  .color-gradient-preview-conic-slider:hover + ::after {
    opacity: 1;
    transition: opacity var(--animation-exit, 0.15s ease-in-out 0.1s);
  }
  .color-gradient-preview-conic-slider:active {
    cursor: grabbing;
  }
  .color-gradient-preview-conic-pointer {
    grid-area: wrapper;
    box-sizing: border-box;
    width: var(--hover-area-diameter, 36px);
    height: var(--hover-area-diameter, 36px);
    background-color: var(--position-slider-bg-color, #fff);
    border-radius: 50%;
    cursor: move;
    z-index: 3;
    background: radial-gradient(
      var(--position-slider-bg-color, #fff) calc(50% / 1.41),
      rgba(0, 0, 0, 0.1) calc(55% / 1.41),
      transparent calc(65% / 1.41)
    );
  }
  .color-gradient-preview-conic-pointer::before {
    background-color: var(--knob-pointer-inner-color-hover, #116dff);
    transition: background-color var(--animation-exit, 0.15s ease-in-out 0.1s);
    position: absolute;
    content: '';
    border-radius: 50%;
    left: calc(
      50% - calc(var(--knob-pointer-inner-diameter, 4px) * 1.6) +
        calc(var(--position-slider-diameter, 10px) / 2)
    );
    top: calc(50% - calc(var(--knob-pointer-inner-diameter, 12px) / 2));
    width: var(--knob-pointer-inner-diameter, 4px);
    height: var(--knob-pointer-inner-diameter, 4px);
  }
  .color-gradient-preview-conic-pointer::after {
    left: calc(
      50% + calc(var(--angle-slider-diameter, 60px) / 2) -
        calc(var(--knob-pointer-outer-diameter, 50px) / 1.5)
    );
    top: calc(50% - calc(var(--knob-pointer-outer-diameter, 64px) / 2));
    width: var(--knob-pointer-outer-diameter, 7px);
    height: var(--knob-pointer-outer-diameter, 7px);
    background-color: var(--knob-pointer-outer-color, #fff);
    pointer-events: none;
    opacity: 0;
    transition: opacity var(--animation-enter, 0.2s ease-in-out 0.05s);
    position: absolute;
    content: '';
    border-radius: 50%;
  }
</style>
