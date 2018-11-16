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
        <Grid v-for="c in data.gridCol" :key="`${ r }-${ c }`"
          :data="gridData[`${ r }-${ c }`]"
          @handleMark="handleMark"
          @handleProbe="handleProbe"
          @handleActive="handleActive"
          @handleMouseup="handleMouseup" />
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
      // 队列数据
      this.probe.queue = { items: {}, indexs: [] }
      // 队列处理中
      this.probe.processing = false
      // 队列中已处理的数量
      this.probe.treadNum = 0

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
          let item = {
            // 下标
            index,
            // 行
            row: r,
            // 列
            col: c,
            // 周围下标
            surrounding: [],
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
          this.handleSurrounding(item, (ind) => {
            // 添加周围下标
            item.surrounding.push(ind)
          })
          gridData[index] = item
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
        let item = gridData[index]
        if (!item.isReal) {
          // 设置为雷
          item.isReal = true
          for (let ind in item.surrounding) {
            gridData[item.surrounding[ind]].num ++
          }

          // 放下一个雷
          i ++
        }
      }

      this.gridData = gridData
    },

    // 查找周围
    handleSurrounding (data, callBack) {
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
        callBack(`${ top }-${ col }`)
      }
      // 右上
      if (tHave && rHave) {
        callBack(`${ top }-${ right }`)
      }
      // 右
      if (rHave) {
        callBack(`${ row }-${ right }`)
      }
      // 右下
      if (rHave && bHave) {
        callBack(`${ bottom }-${ right }`)
      }
      // 下
      if (bHave) {
        callBack(`${ bottom }-${ col }`)
      }
      // 左下
      if (lHave && bHave) {
        callBack(`${ bottom }-${ left }`)
      }
      // 左
      if (lHave) {
        callBack(`${ row }-${ left }`)
      }
      // 左上
      if (lHave && tHave) {
        callBack(`${ top }-${ left }`)
      }
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

    // 结算
    settlement () {
      // 踩开数 + 标记数 + 剩余雷数 = 格子数
      if (this.treadNum + this.markNum + this.sweeperNum === this.data.gridRow * this.data.gridCol) {
        // 成功
        this.gameOver()
      }
    },

    // 踩格
    probe () {
      // 正在处理中
      if (this.probe.processing) {
        return
      }
      this.probe.processing = true

      // 结算
      if (this.probe.queue.indexs.length <= 0) {
        // 累计踩开数
        this.treadNum += this.probe.treadNum
        this.probe.treadNum = 0
        this.settlement()
        this.probe.processing = false
        return
      }

      // 处理队列第一个格
      let index = this.probe.queue.indexs.shift()
      let data = this.probe.queue.items[index]

      if (!data.isProbe) {
        data.isProbe = true
        // 累计处理数量
        this.probe.treadNum ++

        // 处理周围的格
        for (let i in data.surrounding) {
          let item = this.gridData[data.surrounding[i]]
          let surroundingIndex = item.surrounding.indexOf(data.index)
          if (surroundingIndex > -1) {
            // 去掉周围格的记录
            item.surrounding.splice(surroundingIndex, 1)
          }
          if (!item.isProbe && data.num === 0) {
            // 雷数为0时，自动开周围的格
            this.AddProbeQueue(item)
          }
        }
      }

      // 删除当前处理格
      delete this.probe.queue.items[index]
      // 处理完成
      this.probe.processing = false
      // 下一个格
      this.probe()
    },
    // 添加到队列
    AddProbeQueue (data) {
      if (data.isReal) {
        data.isTread = true
        // 踩雷结束
        this.gameOver()
      }

      if (!data.isProbe && this.probe.queue.indexs.indexOf(data.index) === -1) {
        this.probe.queue.items[data.index] = data
        this.probe.queue.indexs.push(data.index)
        !this.probe.processing && this.probe()
      }
    },

    // 踩开一格
    handleProbe (data) {
      let item = this.gridData[data.index]
      item = data
      this.AddProbeQueue(item)
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
    },

    // 同时按住左右键
    handleActive (data, active) {
      for (let i in data.surrounding) {
        let item = this.gridData[data.surrounding[i]]
        if (!item.isProbe) {
          // 变更周围的格样式
          item.isActive = active
        }
      }
    },

    // 同时按住左右键同时松开
    handleMouseup (data) {
      // 累计周围棋标数
      let markNum = 0

      for (let i in data.surrounding) {
        let item = this.gridData[data.surrounding[i]]
        if (!item.isProbe && item.isMark) {
          markNum ++
        }
      }

      // 周围棋标数等于当前格雷数，自动踩开周围无标格
      if (markNum === data.num) {
        for (let i in data.surrounding) {
        let item = this.gridData[data.surrounding[i]]
        if (!item.isProbe && !item.isMark) {
          // 添到队列
          this.AddProbeQueue(item)
        }
      }
      }
    }
  }
}
</script>

<style lang="scss">
.plate {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 100%;
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
