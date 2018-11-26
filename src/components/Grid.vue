<template>
  <div class="grid">
    <samp v-if="data.isProbe && data.isReal"
      :class="data.isTread ? 'probe tread' : 'probe'">💣</samp>

    <samp v-if="data.isProbe && !data.isReal && data.num"
      class="probe"
      @mousedown="handleMousedown"
      @mouseup="handleMouseup"
      @mouseleave="handleMouseleave" >{{ data.num }}</samp>

    <samp v-if="data.isProbe && !data.isReal && !data.num" class="probe"></samp>

    <samp v-if="!data.isProbe && data.isMark"
      class="no-probe mark"
      @mouseup="handleMark">🚩</samp>

    <samp v-if="!data.isProbe && !data.isMark"
      :class="data.isActive ? 'no-probe active' : 'no-probe'"
      @mouseup="handleMark"
      @click="handleProbe"></samp>
  </div>
</template>

<script>
export default {
  name: 'grid',
  props: {
    data: {
      type: Object,
      default: () => {
        return {
          // 下标
          index: '',
          // 行
          row: 1,
          // 列
          col: 1,
          // 是否踩开
          isProbe: false,
          // 是否雷
          isReal: false,
          // 是否标记
          isMark: false,
          // 是否踩中雷
          isTread: false,
          // 周围雷数
          num: 0,
          // 周围格左右键同时按下状态
          isActive: false
        }
      }
    },
  },
  data: () => {
    return {
      // 按住左键
      isHoleDownLeft: false,
      // 按住右键
      isHoleDownRight: false,
      // 同时按住左右键
      isHoleDownLeftAndRight: false,
      // 非同时松开时，0.5秒后同时按住状态失效
      timeout: () => {}
    }
  },
  methods: {
    handleMark (event) {
      // 右击做标记
      if (event && event.which === 3) {
        this.data.isMark = !this.data.isMark
        this.$emit('handleMark', this.data)
      }
    },

    // 踩开
    handleProbe () {
      this.$emit('handleProbe', this.data)
    },

    // 数字格按住左右键事件
    handleMousedown (event) {
      clearTimeout(this.timeout)
      if (event && event.which === 1) {
        // 按住左键
        this.isHoleDownLeft = true
      } else if (event && event.which === 3) {
        // 按住右键
        this.isHoleDownRight = true
      }
      // 同时按住左右键
      if (this.isHoleDownLeft && this.isHoleDownRight) {
        this.isHoleDownLeftAndRight = true
        // 变更周围的格样式
        this.$emit('handleActive', this.data, true)
      }
    },

    // 数字格松开左右键事件
    handleMouseup (event) {
      // 停止按住状态失效监控
      clearTimeout(this.timeout)
      // 变更周围的格样式
      this.$emit('handleActive', this.data, false)

      if (event && event.which === 1) {
        // 松开左键
        this.isHoleDownLeft = false
      } else if (event && event.which === 3) {
        // 松开右键
        this.isHoleDownRight = false
      }

      // 非同时松开时，0.5秒后同时按住状态失效
      if (this.isHoleDownLeftAndRight) {
        this.timeout = setTimeout(() => {
          this.isHoleDownLeftAndRight = false
        }, 500)
      }

      // 同时松开
      if (this.isHoleDownLeftAndRight && !this.isHoleDownLeft && !this.isHoleDownRight) {
        // 停止按住状态失效监控
        clearTimeout(this.timeout)
        this.isHoleDownLeftAndRight = false
        // 变更周围的格样式
        this.$emit('handleMouseup', this.data)
      }
    },

    // 移开失效
    handleMouseleave () {
      if (!this.isHoleDownLeftAndRight) {
        return
      }

      this.isHoleDownLeftAndRight = false
      // 变更周围的格样式
      this.$emit('handleActive', this.data, false)
    }
  }
}
</script>

<style lang="scss">
.grid {
  border-right: 1px solid gray;
  width: 40px;
  height: 40px;

  .probe,
  .no-probe {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }

  .no-probe {
    cursor: pointer;

    &:hover {
      background-color: #FFF2E2;
    }

    &.active,
    &:active {
      background-color: #FDE6E0;
    }
  }

  .probe {
    box-shadow: -1px -1px 8px #888888 inset;
    background-color: #DCE2F1;
  }

  .tread {
    background-color: red;
  }
}
</style>
