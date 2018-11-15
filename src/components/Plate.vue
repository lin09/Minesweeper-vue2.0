<template>
  <div class="plate">
    <div class="info">
      <div class="time">时间：{{ timeText }}</div>
      <div class="sweeper">雷：{{ sweeperNum }}</div>
      <button class="quit" @click="handleQuit">退出</button>
      <button class="quit" @click="handleAgain">重来</button>
    </div>
    <div class="container">
      <div v-for="r in data.gridRow" :key="r" class="row">
        <Grid v-for="c in data.gridCol" :key="`${ r }-${ c }`" :data="gridData[`${ r }-${ c }`]" @handleMark="handleMark" @handleProbe="handleProbe" />
      </div>
    </div>
  </div>
</template>

<script>
import Grid from './Grid.vue'

export default {
  name: 'plate',
  props: {
    // 开始
    value: {
      type: Boolean,
      default: () => {
        return false
      }
    },
    // 格子数据
    data: {
      type: Object,
      default: () => {
        return {
          gridRow: 9,
          gridCol: 9,
          sweeper: 10
        }
      }
    }
  },
  components: {
    Grid
  },
  data: () => {
    return {
      // 计时秒
      second: 0,
      // 计时显示时间
      timeText: '00:00',
      // 计时器
      timeout: () => {},
      // 全部格的数据
      gridData: {},
      // 雷数
      sweeperNum: 0,
      // 踩开数
      treadNum: 0,
      // 棋标数
      markNum: 0
    }
  },
  watch: {
    value (val) {
      // 开始游戏
      if (val) {
        // 初始化
        this.init()
      }
    }
  },
  methods: {
    init () {
      // 初始化数据
      this.sweeperNum = this.data.sweeper
      this.treadNum = 0
      this.markNum = 0
      this.handleGridData()

      // 重置时间
      this.second = 0
      this.timeText = '00:00'
      // 开始计时
      this.handleTimeing()
    },

    // 退出
    handleQuit () {
      // 删除计时器
      clearTimeout(this.timeout)
      // 退出
      this.$emit('input', false)
    },

    // 重来
    handleAgain () {
      // 删除计时器
      clearTimeout(this.timeout)

      // 重新初始化数据
      this.init()
    },

    // 初始化数据
    handleGridData () {
      let gridData = {}

      for (let r = 1; r <= this.data.gridRow; r ++) {
        for (let c = 1; c <= this.data.gridCol; c ++) {
          let index = `${ r }-${ c }`
          gridData[index] = {
            // 下标
            index,
            // 行
            row: r,
            // 列
            col: c,
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
      }

      // 随机放雷
      for (let i = 0; i < this.data.sweeper;) {
        // 随机行标
        let row = this.handleRandom(this.data.gridRow)
        // 随机列标
        let col = this.handleRandom(this.data.gridCol)
        // 组装下标
        let index = `${ row }-${ col }`
        // 过滤已经放雷
        if (!gridData[index].isReal) {
          // 设置为雷
          gridData[index].isReal = true

          // 计算周围雷数
          // 上
          let top = row - 1
          let tHave = !!(top > 0)
          // 右
          let right = col + 1
          let rHave = !!(right <= this.data.gridCol)
          // 下
          let bottom = row + 1
          let bHave = !!(bottom <= this.data.gridRow)
          // 左
          let left = col - 1
          let lHave = !!(left > 0)

          // 上
          if (tHave) {
            gridData[`${ top }-${ col }`].num ++
          }
          // 右上
          if (tHave && rHave) {
            gridData[`${ top }-${ right }`].num ++
          }
          // 右
          if (rHave) {
            gridData[`${ row }-${ right }`].num ++
          }
          // 右下
          if (rHave && bHave) {
            gridData[`${ bottom }-${ right }`].num ++
          }
          // 下
          if (bHave) {
            gridData[`${ bottom }-${ col }`].num ++
          }
          // 左下
          if (lHave && bHave) {
            gridData[`${ bottom }-${ left }`].num ++
          }
          // 左
          if (lHave) {
            gridData[`${ row }-${ left }`].num ++
          }
          // 左上
          if (lHave && tHave) {
            gridData[`${ top }-${ left }`].num ++
          }

          // 放下一个雷
          i ++
        }
      }

      this.gridData = gridData
    },

    // 随机数
    handleRandom (max) {
      return Math.ceil(Math.random()*max)
    },

    // 计时器
    handleTimeing () {
      let timeing = () => {
        this.timeout = setTimeout(() => {
          ++ this.second
          // 分钟
          let minute = Math.floor(this.second / 60)
          // 秒
          let second = this.second % 60
          if (minute < 10) {
            // 补0
            minute = `0${ minute }`
          }
          if (second < 10) {
            // 补0
            second = `0${ second }`
          }
          // 分秒时间
          this.timeText = `${ minute }:${ second }`

          // 下一秒
          timeing()
        }, 1000)
      }
      // 开始计时
      timeing()
    },

    // game over
    gameOver () {
      // 删除计时器
      clearTimeout(this.timeout)

      // 全部开
      for (let grid in this.gridData) {
        this.gridData[grid].isProbe = true
      }
    },

    // 成功踩格
    handleTread (data) {
      // 累计踩开数
      this.treadNum ++
      // 踩开数 + 标记数 + 剩余雷数 = 格子数
      if (this.treadNum + this.markNum + this.sweeperNum === this.data.gridRow * this.data.gridCol) {
        // 成功
        this.gameOver()
      }
    },

    // 踩格
    probe (data) {
      if (!data.isProbe) {
        data.isProbe = true
        this.handleTread(data)

        // 周围雷数为0，自动踩开周围
        if (data.num === 0) {
          this.handleZeroGrid(data)
        }
      }
    },

    // 踩开周围
    handleZeroGrid (data) {
      let row = data.row
      let col = data.col

      // 上
      let top = row - 1
      let tHave = !!(top > 0)
      // 右
      let right = col + 1
      let rHave = !!(right <= this.data.gridCol)
      // 下
      let bottom = row + 1
      let bHave = !!(bottom <= this.data.gridRow)
      // 左
      let left = col - 1
      let lHave = !!(left > 0)

      // 上
      if (tHave) {
        this.probe(this.gridData[`${ top }-${ col }`])
      }
      // 右上
      if (tHave && rHave) {
        this.probe(this.gridData[`${ top }-${ right }`])
      }
      // 右
      if (rHave) {
        this.probe(this.gridData[`${ row }-${ right }`])
      }
      // 右下
      if (rHave && bHave) {
        this.probe(this.gridData[`${ bottom }-${ right }`])
      }
      // 下
      if (bHave) {
        this.probe(this.gridData[`${ bottom }-${ col }`])
      }
      // 左下
      if (lHave && bHave) {
        this.probe(this.gridData[`${ bottom }-${ left }`])
      }
      // 左
      if (lHave) {
        this.probe(this.gridData[`${ row }-${ left }`])
      }
      // 左上
      if (lHave && tHave) {
        this.probe(this.gridData[`${ top }-${ left }`])
      }
    },

    // 踩开一格
    handleProbe (data) {
      this.gridData[data.index] = data
      if (data.isTread) {
        // 踩雷结束
        this.gameOver()
      } else {
        this.probe(this.gridData[data.index])
      }
    },

    // 标记
    handleMark (data) {
      this.gridData[data.index] = data
      // 棋标变更，雷数变更
      if (data.isMark) {
        this.sweeperNum --
        this.markNum ++
      } else {
        this.sweeperNum ++
        this.markNum --
      }
    }
  }
}
</script>

<style lang="scss">
.plate {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.info {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
  width: 100%;
  min-width: 400px;
  max-width: 100vw;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: #EAEAEF;
  box-shadow: 1px 1px 14px;

  .row {
    display: flex;
    border-top: 1px solid gray;
    border-left: 1px solid gray;
    &:last-of-type {
      border-bottom: 1px solid gray;
    }
  }

  .col {
    display: flex;
    width: 40px;
    height: 40px;
  }
}
</style>
