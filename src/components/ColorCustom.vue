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
    },
    data() {
      return {
        themes: [
          [
            'rgb(255, 255, 255)',
            'rgb(227, 227, 227)',
            'rgb(151, 151, 151)',
            'rgb(79, 79, 79)',
            'rgb(0, 0, 0)',
          ],
          [
            'rgb(232, 234, 237)',
            'rgb(149, 158, 229)',
            'rgb(56, 74, 211)',
            'rgb(37, 49, 141)',
            'rgb(19, 25, 70)',
          ],
          [
            'rgb(242, 222, 204)',
            'rgb(194, 178, 163)',
            'rgb(145, 133, 122)',
            'rgb(97, 89, 82)',
            'rgb(48, 44, 41)',
          ],
          [
            'rgb(209, 219, 195)',
            'rgb(171, 184, 153)',
            'rgb(127, 148, 99)',
            'rgb(85, 99, 66)',
            'rgb(42, 49, 33)',
          ],
          [
            'rgb(199, 211, 213)',
            'rgb(154, 169, 172)',
            'rgb(103, 126, 130)',
            'rgb(69, 84, 87)',
            'rgb(34, 42, 43)',
          ],
        ],
        colors: [
          'rgb(0,0,0)',
          'rgb(255,255,255)',
          'rgb(139,0,0)',
          'rgb(186,218,85)',
          'rgb(50,50,50)',
          'rgb(162,163,233)',
          'rgb(139, 140, 214)',
          'rgb(228, 50, 146)',
          'rgb(237, 69, 160)',
          'rgb(174, 239, 197)',
          'rgb(56, 198, 139)',
          'rgb(29, 44, 243)',
          'rgb(180, 90, 211)',
          'rgb(142, 58, 171)',
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
