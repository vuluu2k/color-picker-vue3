<script>
  export default {
    emits: ['change'],
    props: {
      value: {
        type: String,
        default: '#000000',
      },
    },
    mounted() {
      this.previewColor = this.value
      if (this.value.includes('gradient')) {
        this.$emit('change', '#FFFFFF')
      }
    },
    data() {
      return {
        themes: [
          [
            'rgba(255, 255, 255, 1)',
            'rgba(227, 227, 227, 1)',
            'rgba(151, 151, 151, 1)',
            'rgba(79, 79, 79, 1)',
            'rgba(0, 0, 0, 1)',
          ],
          [
            'rgba(232, 234, 237, 1)',
            'rgba(149, 158, 229, 1)',
            'rgba(56, 74, 211, 1)',
            'rgba(37, 49, 141, 1)',
            'rgba(19, 25, 70, 1)',
          ],
          [
            'rgba(242, 222, 204, 1)',
            'rgba(194, 178, 163, 1)',
            'rgba(145, 133, 122, 1)',
            'rgba(97, 89, 82, 1)',
            'rgba(48, 44, 41, 1)',
          ],
          [
            'rgba(209, 219, 195, 1)',
            'rgba(171, 184, 153, 1)',
            'rgba(127, 148, 99, 1)',
            'rgba(85, 99, 66, 1)',
            'rgba(42, 49, 33, 1)',
          ],
          [
            'rgba(199, 211, 213, 1)',
            'rgba(154, 169, 172, 1)',
            'rgba(103, 126, 130, 1)',
            'rgba(69, 84, 87, 1)',
            'rgba(34, 42, 43, 1)',
          ],
        ],
        colors: [
          'rgba(0,0,0, 1)',
          'rgba(255,255,255, 1)',
          'rgba(139,0,0, 1)',
          'rgba(186,218,85, 1)',
          'rgba(50,50,50, 1)',
          'rgba(162,163,233, 1)',
          'rgba(139, 140, 214, 1)',
          'rgba(228, 50, 146, 1)',
          'rgba(237, 69, 160, 1)',
          'rgba(174, 239, 197, 1)',
          'rgba(56, 198, 139, 1)',
          'rgba(29, 44, 243, 1)',
          'rgba(180, 90, 211, 1)',
          'rgba(142, 58, 171, 1)',
        ],
        previewColor: '#000000',
      }
    },
    methods: {
      onMouseOverColor(event) {
        this.previewColor = event.target.getAttribute('data-color')
      },
      onMouseLeaveColor() {
        this.previewColor = this.value
      },
      onAdd() {
        this.$emit('setView', 'ColorPicker')
      },
      onSelectColor(color) {
        this.$emit('change', color)
      },
    },
  }
</script>

<template>
  <div class="color-custom">
    <div class="color-custom-container">
      <div class="color-custom-add">
        <div class="color-custom-add-title">Theme colors</div>
        <div class="color-custom-add-action">Edit</div>
      </div>

      <div class="color-custom-themes">
        <div
          class="color-custom-theme"
          v-for="(theme, index) in themes"
          :key="index"
        >
          <div
            class="color-custom-theme-color"
            v-for="(color, index) in theme"
            @mouseover="onMouseOverColor"
            @mouseleave="onMouseLeaveColor"
            @mousedown.stop.prevent="onSelectColor(color)"
            :data-color="color"
            :key="index"
            :style="{ background: color }"
          ></div>
        </div>
      </div>

      <div class="color-custom-add">
        <div class="color-custom-add-title">My colors</div>
        <div class="color-custom-add-action" @mousedown.prevent="onAdd">
          + Add
        </div>
      </div>
      <div class="color-custom-colors">
        <div
          v-for="(color, index) in colors"
          class="color-custom-item"
          :key="index"
        >
          <div
            @mouseover="onMouseOverColor"
            @mouseleave="onMouseLeaveColor"
            @mousedown.stop.prevent="onSelectColor(color)"
            :data-color="color"
            class="color-custom-item-inner"
            :style="{ background: color }"
          ></div>
        </div>
      </div>
      <div class="color-custom-display">{{ previewColor }}</div>
    </div>
  </div>
</template>

<style scoped>
  .color-custom {
    width: 240px;
  }
  .color-custom-container {
    padding: 0px 12px;
  }
  .color-custom-add {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 12px;
    font-size: 13px;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  .color-custom-add-action {
    color: var(--primary, #1677ff);
    cursor: pointer;
  }
  .color-custom-colors {
    display: flex;
    flex-wrap: wrap;
    gap: 6px 0;
  }
  .color-custom-item {
    width: 28px;
    height: 28px;
    flex: 0 1 calc(100% / 6);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .color-custom-item-inner {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    box-shadow: inset 0 0 0 1px hsla(0, 0%, 78.4%, 0.5);
    transition: 0.2s;
  }
  .color-custom-item-inner:hover {
    width: 28px;
    height: 28px;
  }
  .color-custom-display {
    margin-top: 12px;
    text-align: right;
    height: 12px;
    font-size: 12px;
    color: #595d70;
  }
  .color-custom-theme {
    display: flex;
    margin-bottom: 10px;
  }
  .color-custom-theme-color {
    position: relative;
    width: 48px;
    height: 20px;
    box-sizing: border-box;
    cursor: pointer;
    box-shadow: 0 0 0 1px hsla(0, 0%, 78.4%, 0.5);
  }

  .color-custom-theme-color:hover:after {
    border-radius: 2px;
    z-index: 1;
    box-shadow: 0 0 7px 0 rgba(22, 45, 61, 0.44);
    content: '';
    position: absolute;
    top: -2px;
    left: -2px;
    width: calc(100% + 2px * 2);
    height: calc(100% + 2px * 2);
    background-color: inherit;
  }
</style>
