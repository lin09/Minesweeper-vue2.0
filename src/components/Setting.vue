/* eslint-disable */
<template>
  <div class="setting">
    <div class="grade">
      <button @click="handleGrade(0)" :class="grade === 0 ? 'active' : ''">初级</button>
      <button @click="handleGrade(1)" :class="grade === 1 ? 'active' : ''">中级</button>
      <button @click="handleGrade(2)" :class="grade === 2 ? 'active' : ''">高级</button>
      <button @click="handleGrade(3)" :class="grade === 3 ? 'active' : ''">自定义</button>
    </div>
    <div class="grid-num">
      <div>雷区大小：</div>
      <input type="number" v-model="gridCol" :disabled="grade !== 3 ? true : false" @blur="handleGridCol">
      <div class="x">x</div>
      <input type="number" v-model="gridRow" :disabled="grade !== 3 ? true : false" @blur="handleGridRow">
    </div>
    <div class="sweeper-num">
      <div>地雷数量：</div>
      <input type="number" v-model="sweeper" :disabled="grade !== 3 ? true : false" @blur="handleSweeper">
    </div>
    <button class="start" @click="handleStart">开始</button>
  </div>
</template>

<script>
export default {
  name: 'setting',
  data: () => {
    return {
      // 等级难度
      grade: 0,
      // 行数
      gridRow: 9,
      // 列数
      gridCol: 9,
      // 雷数
      sweeper: 10,
      // 最大雷数
      maxSweeper: 80,
    }
  },
  watch: {
    gridRow (val) {
      if (val > 1) {
        // 计算最大雷数
        this.maxSweeper = this.gridRow * this.gridCol - 1
        this.handleSweeper()
      }
    },
    gridCol (val) {
      if (val > 1) {
        // 计算最大雷数
        this.maxSweeper = this.gridRow * this.gridCol - 1
        this.handleSweeper()
      }
    },
  },
  methods: {
    handleGrade (grade) {
      // 处理等级难度
      this.grade = grade
      switch (grade) {
        case 0:
          this.gridRow = 9
          this.gridCol = 9
          this.sweeper = 10
          break
        case 1:
          this.gridRow = 16
          this.gridCol = 16
          this.sweeper = 40
          break
        case 2:
          this.gridRow = 16
          this.gridCol = 30
          this.sweeper = 99
          break
      }
    },
    handleGridRow () {
      // 自定义行数不能小于2
      if (this.gridRow * 1 < 2) {
        this.gridRow = 2
      }
    },
    handleGridCol () {
      // 自定义列数不能小于2
      if (this.gridCol * 1 < 2) {
        this.gridCol = 2
      }
    },
    handleSweeper () {
      let min = Math.ceil(this.maxSweeper / 10)
      if (this.sweeper * 1 < min) {
        // 自定义雷数不能小于最大雷数5分1
        this.sweeper = min
      } else if (this.sweeper * 1 > this.maxSweeper) {
        // 自定义雷数不能大于最大雷数
        this.sweeper = this.maxSweeper
      }
    },
    handleStart () {
      // 回传数据
      this.$emit('handleStart', {
        gridRow: this.gridRow * 1,
        gridCol: this.gridCol * 1,
        sweeper: this.sweeper * 1
      })
    }
  }
}
</script>

<style lang="scss">
.setting {
  display: flex;
  flex-direction: column;
  margin-top: 40px;

  > * {
    margin-bottom: 20px;
  }

  .grade {
    display: flex;
    justify-content: space-between;

    .active {
      background-color: #DCE2F1;
    }
  }

  .grid-num, .sweeper-num {
    display: flex;
    align-self: flex-start;
  }

  .x {
    margin: 0 10px;
  }
}
</style>
