/* eslint-disable */
<template>
  <div class="game">
    <div class="title">扫雷</div>
    <Setting v-show="!isStart" @handleStart="handleStart" />
    <Plate v-show="isStart" v-model="isStart" :data="data"/>
  </div>
</template>

<script>
import Setting from './Setting.vue'
import Plate from './Plate.vue'

export default {
  name: 'game',
  data: () => {
    return {
      isStart: false,
      data: {}
    }
  },
  components: {
    Setting,
    Plate
  },
  methods: {
    handleStart (data) {
      // 设置雷区数据
      this.data = data
      // 开始
      this.isStart = true
    }
  }
}
</script>

<style lang="scss">
button {
  border-radius: 4px;
  padding: 5px 10px;
  cursor: pointer;

  &:hover {
    background-color: #FFF2E2;
  }

  &:active {
    background-color: #FDE6E0;
  }
}

.game {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.title {
  margin-bottom: 40px;
  text-align: center;
  font-size: 24px;
}
</style>
