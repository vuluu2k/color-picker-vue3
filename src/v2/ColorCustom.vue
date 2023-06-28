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
        this.$emit('change', 'rgba(0,0,0,1)')
      }
    },
    data() {
      return {
        themes: [
          ['#ffffff', '#e3e3e3', '#979797', '#4f4f4f', '#000000'],
          ['#e8eaed', '#959ee5', '#384ad3', '#25318d', '#131946'],
          ['#f2decc', '#c2b2a3', '#917a7a', '#615952', '#30243f'],
          ['#d1dbc3', '#abb899', '#7f9463', '#556342', '#2a3121'],
          ['#c7d3d5', '#9aa9ac', '#677e82', '#45575a', '#222a2b'],
        ],
        colors: [
          '#000000',
          '#ffffff',
          '#8b0000',
          '#bada55',
          '#323232',
          '#a2a3e9',
          '#8b8cd6',
          '#e43292',
          '#ed45a0',
          '#aeebc5',
          '#38c68b',
          '#1d2cf3',
          '#b45ad3',
          '#8e3aaa',
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
          :key="index">
          <div
            class="color-custom-theme-color"
            v-for="(color, index) in theme"
            @mouseover="onMouseOverColor"
            @mouseleave="onMouseLeaveColor"
            @mousedown.stop.prevent="onSelectColor(color)"
            :data-color="color"
            :key="index"
            :style="{ background: color }"></div>
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
          :key="index">
          <div
            @mouseover="onMouseOverColor"
            @mouseleave="onMouseLeaveColor"
            @mousedown.stop.prevent="onSelectColor(color)"
            :data-color="color"
            class="color-custom-item-inner"
            :style="{ background: color }"></div>
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
