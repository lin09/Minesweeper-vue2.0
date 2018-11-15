<template>
  <div class="grid">
    <samp v-if="data.isProbe && data.isReal" :class="data.isTread ? 'probe tread' : 'probe'">雷</samp>
    <samp v-if="data.isProbe && !data.isReal" class="probe">{{ data.num ? data.num : '' }}</samp>
    <samp v-if="!data.isProbe && data.isMark" class="no-probe mark" @mouseup="handleMark">标</samp>
    <samp v-if="!data.isProbe && !data.isMark" class="no-probe" @mouseup="handleMark" @click="handleProbe"></samp>
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
        }
      }
    },
  },
  methods: {
    handleMark (event) {
      // 右击做标记
      if (event && event.which === 3) {
        this.data.isMark = !this.data.isMark
        this.$emit('handleMark', this.data)
      }
    },
    handleProbe () {
      // 踩开
      this.data.isTread = this.data.isReal
      this.$emit('handleProbe', this.data)
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
