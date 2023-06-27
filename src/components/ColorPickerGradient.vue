<script>
  import ColorCustom from './ColorCustom.vue'
  import ColorPicker from './ColorPicker.vue'
  import ColorGradient from './ColorGradient.vue'

  export default {
    emits: ['update:value', 'update:selection', 'change'],
    components: {
      ColorCustom,
      ColorPicker,
      ColorGradient,
    },
    props: {
      value: {
        type: String,
        default: '#000000',
      },
    },
    mounted() {
      const picker = this.$refs.picker.getBoundingClientRect()
      this.pickerLocation.top = picker.bottom + 2
      this.pickerLocation.left = picker.left
      if (this.value.includes('gradient')) {
        this.selected = 'gradient'
        this.gradient = this.value
      } else {
        this.selected = 'solid'
        this.solid = this.value
      }
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
        selected: 'solid',
        solid: '',
        gradient: '',
      }
    },
    methods: {
      toggle(event) {
        event.preventDefault()
        this.show = !this.show
      },
      async setView(view) {
        this.view = view
        await this.$nextTick()
      },
      handleColorPickerChange(color, selection) {
        console.log('object')
        this.$emit('update:value', color)
        this.$emit('change', color)
        if (selection) this.$emit('update:selection', selection)
      },
      handleSelectType(type) {
        this.selected = type
        if (type == 'solid') {
          this.gradient = this.value
          this.$emit('update:value', this.solid)
        } else if (type == 'gradient') {
          this.solid = this.value
          this.$emit('update:value', this.gradient)
        }
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
    },
    beforeUnmount() {
      document.removeEventListener('mousemove', this.onMouseMoveLocation)
      document.removeEventListener('mouseup', this.onMouseUpLocation)
    },
  }
</script>

<template>
  <div>
    <div
      @mousedown.stop.prevent="toggle"
      class="picker"
      ref="picker"
      :style="{ background: value }"
    ></div>

    <Teleport to="body">
      <div
        class="color-picker-custom"
        v-if="show"
        :style="{
          position: 'absolute',
          top: `${pickerLocation.top}px`,
          left: `${pickerLocation.left}px`,
        }"
      >
        <div
          class="color-picker-custom-header"
          @mousedown.stop.prevent="onMouseDownLocation"
          ref="pickerHeader"
        >
          <div class="color-picker-custom-title" v-if="view == 'ColorPicker'">
            <span
              class="color-picker-custom-back"
              @mousedown.stop.prevent="setView('ColorCustom')"
            >
              <svg
                viewBox="0 0 24 24"
                fill="currentColor"
                width="24"
                height="24"
              >
                <path
                  d="M13.5 17.914 7.086 11.5 13.5 5.086 14.914 6.5l-5 5 5 5-1.414 1.414Z"
                ></path>
              </svg>
            </span>
            <span> Color Picker </span>
          </div>
          <div class="color-picker-custom-title" v-if="view == 'ColorCustom'">
            Color Custom
          </div>
          <div
            class="color-picker-custom-close"
            @mousedown.stop.prevent="toggle"
          >
            <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
              <path
                d="m13.59 12 4.344-4.342a.226.226 0 0 0 0-.318L16.66 6.067a.223.223 0 0 0-.318 0L12 10.409 7.658 6.067a.223.223 0 0 0-.318 0L6.066 7.34a.226.226 0 0 0 0 .318L10.409 12l-4.343 4.343a.226.226 0 0 0 0 .318l1.274 1.273c.088.088.23.088.318 0L12 13.59l4.343 4.343c.088.088.23.088.318 0l1.273-1.273a.226.226 0 0 0 0-.318L13.59 12Z"
              ></path>
            </svg>
          </div>
        </div>
        <div class="color-picker-custom-select">
          <div
            class="color-picker-custom-select-item"
            :data-selected="selected == 'solid'"
            @mousedown.prevent="handleSelectType('solid')"
          >
            <div class="color-picker-custom-solid"></div>
          </div>
          <div
            class="color-picker-custom-select-item"
            :data-selected="selected == 'gradient'"
            @mousedown.prevent="handleSelectType('gradient')"
          >
            <div class="color-picker-custom-gradient"></div>
          </div>
        </div>
        <div class="color-picker-custom-content">
          <div v-if="selected == 'solid'">
            <ColorPicker
              :value="value"
              @change="handleColorPickerChange"
              v-if="view == 'ColorPicker'"
              @setView="setView"
            />
            <ColorCustom
              :value="value"
              @setView="setView"
              @change="handleColorPickerChange"
              v-if="view == 'ColorCustom'"
            />
          </div>
          <div v-if="selected == 'gradient'">
            <ColorGradient :value="value" @change="handleColorPickerChange" />
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<style scoped>
  .picker {
    width: 26px;
    height: 26px;
    border-radius: 4px;
    position: relative;
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
  }
  .color-picker-custom-select {
    display: flex;
    box-shadow: inset 0 -2px 0 0 rgba(0, 6, 36, 0.05);
    height: 54px;
  }
  .color-picker-custom-select-item {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    border-bottom: 2px solid var(--divider, #e0e0e0);
  }
  .color-picker-custom-select-item[data-selected='true'] {
    border-bottom: 2px solid var(--primary, #1677ff);
  }
  .color-picker-custom-solid {
    background-color: #000;
    width: 22px;
    height: 22px;
    border-radius: 50%;
  }
  .color-picker-custom-gradient {
    width: 22px;
    height: 22px;
    border-radius: 50%;
    background-image: linear-gradient(#fff, #000);
  }
</style>
