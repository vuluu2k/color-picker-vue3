<script>
  import ColorCustom from './ColorCustom.vue'
  import ColorPicker from './ColorPicker.vue'

  export default {
    setup() {
      return {}
    },
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
    mounted() {},
    data() {
      return {
        show: false,
        view: 'ColorCustom',
      }
    },
    methods: {
      toggle(event) {
        event.preventDefault()
        this.show = !this.show
      },
      setView(view) {
        this.view = view
      },
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
      <div class="color-picker-custom" v-if="show">
        <div class="color-picker-custom-header">
          <div class="color-picker-custom-title" v-if="view == 'ColorCustom'">
            Color Picker
          </div>
          <div class="color-picker-custom-title" v-if="view == 'ColorPicker'">
            Custom Color
          </div>
          <div class="color-picker-custom-close">X</div>
        </div>
        <div class="color-picker-custom-content">
          <component :is="view" @setView="setView" :value="value" />
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
    padding: 12px 22px;
    border-bottom: 1px solid var(--divider, #e0e0e0);
    display: flex;
    justify-content: space-between;
  }
  .color-picker-custom-close {
  }
  .color-picker-custom-title {
    font-weight: bold;
  }
  .color-picker-custom {
    border-radius: 8px;
    z-index: 1;
    background-color: #fff;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  }
</style>
