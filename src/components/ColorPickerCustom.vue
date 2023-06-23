<script>
  import ColorCustom from './ColorCustom.vue'
  import ColorPicker from './ColorPicker.vue'

  export default {
    emits: ['update:value'],
    components: {
      ColorCustom,
      ColorPicker,
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
    },
    data() {
      return {
        show: false,
        view: 'ColorCustom',
        pickerLocation: {
          top: 0,
          left: 0,
        },
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
      handleColorPickerChange(color) {
        this.$emit('update:value', color)
      },
      onMouseDownLocation(event) {
        document.addEventListener('mousemove', this.onMouseMoveLocation)
        document.addEventListener('mouseup', this.onMouseUpLocation)
      },
      onMouseMoveLocation(event) {
        this.pickerLocation.top = event.clientY
        this.pickerLocation.left = event.clientX
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
      @mousedown="toggle"
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
          @mousedown.prevent="onMousedownLocation"
        >
          <div
            class="color-picker-custom-title"
            v-if="view == 'ColorPicker'"
            @mousedown.stop.prevent="setView('ColorCustom')"
          >
            <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
              <path
                d="M13.5 17.914 7.086 11.5 13.5 5.086 14.914 6.5l-5 5 5 5-1.414 1.414Z"
              ></path>
            </svg>
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
        <div class="color-picker-custom-content">
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
      </div>
    </Teleport>
  </div>
</template>

<style scoped>
  .picker {
    width: 26px;
    height: 26px;
    border-radius: 4px;
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
</style>
