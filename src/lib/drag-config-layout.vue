<template>
  <div ref="drag_container">
    <div class="drag-content" :style="{width:spaceSizeX*dragMaxX+'px',height:spaceSizeY*dragMaxY+'px'}" v-if="isEdit">
      <draggable-resizable :class="[isEdit?'':'vdr_eye']" :parent="true" v-for="(item,index) in homeDragList" :key="index"
                              :canvas-number="[dragMaxX,dragMaxY]"
                              :active="item.isActive"
                              class-name="draggable_custom"
                              class-name-handle="draggable_custom_handle"
                              :drag-cancel="'.drag-cancel'"
                              :min-width="item.minWidth*spaceSizeX" :min-height="item.minHeight*spaceSizeY"
                              :max-width="item.maxWidth*spaceSizeX" :max-height="item.maxHeight*spaceSizeY"
                              :grid=[spaceSizeX,spaceSizeY]
                              :w="item.w*spaceSizeX"
                              :h="item.h*spaceSizeY"
                              :x="item.x*spaceSizeX"
                              :y="item.y*spaceSizeY"
                              :z="item.zIndex"
                              :handles="['br']"
                              :isConflictCheck="true"
                              :snap="true"
                              :draggable="isEdit"
                              :resizable="isEdit"
                              @refLineParams="getRefLineParams"
                              @resizing="resizechanging(arguments,item)"
                              @dragging="changing(arguments,item)"
                              @resizestop="resizestop(arguments,item)"
                              @dragstop = "dragstop(arguments,item)"
      >
        <div class="drag-box" :class="['drag-box_col--'+item.w , 'drag-box_row--'+item.h ,item.customClass]">
          <slot :name="item.slotName"></slot>
        </div>
        <div class="draggable_custom_closed drag-cancel"  v-if="item.canDel&&isEdit" @click="deleteDrag(item,index)">
          <img src="./img/icon-delete.png"/>
        </div>

        <div slot="br"><img src="./img/icon-drag.png"/></div>
      </draggable-resizable>
      <span class="ref-line v-line"
            v-for="item in vLine"
            v-show="item.display"
            :style="{ left: item.position, top: item.origin, height: item.lineLength}"
      />
      <span class="ref-line h-line"
            v-for="item in hLine"
            v-show="item.display"
            :style="{ top: item.position, left: item.origin, width: item.lineLength}"
      />
      <canvas id="drag_bg"></canvas>
    </div>
    <div class="drag-content" :style="{width:spaceSizeX*dragMaxX+'px',height:spaceSizeY*eyeYSpaceLength+'px'}" v-else>
      <div class="draggable_custom vdr_eye" v-for="(item,index) in homeDragList" :key="index" :style="eyeStyle(item)">
        <div class="drag-box" :class="['drag-box_col--'+item.w , 'drag-box_row--'+item.h ,item.customClass]">
          <slot :name="item.slotName"></slot>
        </div>
      </div>
    </div>

  </div>

</template>

<script>
  import '@babel/polyfill';
  import DraggableResizable from './draggable-resizable.vue'
  import './draggable-resizable.css'
  import { deepClone } from './utils/fns'
  export default {
    name: 'drag-config-layout',
    components: { DraggableResizable },
    data() {
      return {
        homeDragList: [],
        vLine: [],
        hLine: [],
        allSliceArr: [],
        dragSliceArr: [],
        spaceSizeY: this.spaceSize,
        containerWidth: 1200

      }
    },
    props: {
      dataList: {
        type: Array,
        default: function() {
          return [
            {
              isActive: true, // 是否默认选中
              canDel: true, // 是否能删除当前模块
              type: 'one', // 模块的类型
              w: 4, // 模块宽度的占位 最大为12个
              h: 4, // 模块高度占位 最大为12个
              x: 0, // 模块距离 左上角的占位的数量
              y: 0, // 模块距离 左上角的占位的数量
              minWidth: 2, // 模块宽度最小占位数量
              minHeight: 2, // 模块高度最小占位数量
              maxWidth: 12, // 模块宽度最大占位数量
              maxHeight: 12, // 模块宽度最大占位数量
            }]
        }
      },
      isEdit: {
        type: Boolean,
        default: true
      },
      dragMaxY: {
        type: Number,
        default: 12
      },
      dragMaxX: {
        type: Number,
        default: 12
      },
      spaceSize: {
        type: Number,
        default: 100
      }
    },
    computed: {
      eyeYSpaceLength() {
        if (this.isEdit) {
          return this.dragMaxY
        } else {
          let boxHasArr = this.initDragSlice()
          let newYArr = []
          boxHasArr.forEach(item => {
            let yNumber = Number(item.split('_')[1])
            if (newYArr.indexOf(yNumber) === -1) {
              newYArr.push(yNumber)
            }
          })
          let max = newYArr.reduce(function(a, b) {
            return b > a ? b : a
          })
          return Number(max) + 1
        }
      },
      spaceSizeX() {
        let space = parseInt(this.containerWidth / this.dragMaxX)
        return space <= this.spaceSize ? this.spaceSize : space
      }
    },
    mounted() {
      this.homeDragList = deepClone(this.dataList)
      this.DrawDrid()
      this.allSliceArr = this.initAllSlice()

      let that = this
      that.setContainer()
      window.onresize = function() {
        that.setContainer()
      }
    },
    beforeDestroy: function() {
      window.onresize = null
    },
    watch: {
      dataList: {
        handler: function(newVal) {
          this.homeDragList = deepClone(newVal)
        },
        deep: true
      },
      isEdit(val) {
        this.$nextTick(() => {
          this.DrawDrid()
        })
      },
      spaceSizeX(val) {
        this.$nextTick(() => {
          this.DrawDrid()
        })
      },
      dragMaxY(val) {
        this.$nextTick(() => {
          this.DrawDrid()
          this.allSliceArr = this.initAllSlice()
        })
      },
      dragMaxX(val) {
        this.$nextTick(() => {
          this.DrawDrid()
          this.allSliceArr = this.initAllSlice()
        })
      }
    },
    methods: {
      setContainer() {
        this.containerWidth = this.$refs.drag_container.clientWidth
      },
      eyeStyle(item) {
        return {
          transform: `translate(${item.x * this.spaceSizeX}px, ${item.y * this.spaceSizeY}px)`,
          width: item.w * this.spaceSizeX + 'px',
          height: item.h * this.spaceSizeY + 'px',
          zIndex: 'auto'
        }
      },
      getRefLineParams(params) {
        const { vLine, hLine } = params
        this.vLine = vLine
        this.hLine = hLine
      },
      changing(arg, item) {
        item.zIndex = 99
        if (item.x === arg[0] / this.spaceSizeX &&
          item.y === arg[1] / this.spaceSizeY) {
          return
        }
        item.x = arg[0] / this.spaceSizeX
        item.y = arg[1] / this.spaceSizeY
        this.$emit('changing', item)
      },
      resizechanging(arg, item) {
        item.zIndex = 99
        if (item.x === arg[0] / this.spaceSizeX &&
          item.y === arg[1] / this.spaceSizeY &&
          arg[2] / this.spaceSizeX === item.w &&
          arg[3] / this.spaceSizeY === item.h) {
          return
        }
        item.x = arg[0] / this.spaceSizeX
        item.y = arg[1] / this.spaceSizeY
        item.w = arg[2] / this.spaceSizeX
        item.h = arg[3] / this.spaceSizeY
        this.$emit('changing', item)
      },
      resizestop(arg, item) {
        item.x = arg[0] / this.spaceSizeX
        item.y = arg[1] / this.spaceSizeY
        item.w = arg[2] / this.spaceSizeX
        item.h = arg[3] / this.spaceSizeY
        item.zIndex = 'auto'
        this.$emit('change', item, this.homeDragList)
      },
      dragstop(arg, item) {
        item.x = arg[0] / this.spaceSizeX
        item.y = arg[1] / this.spaceSizeY
        item.zIndex = 'auto'
        this.$emit('change', item, this.homeDragList)
      },

      initDragSlice() {
        let dragArr = []
        this.homeDragList.forEach(item => {
          for (let i = 0; i < item.w; i++) {
            for (let j = 0; j < item.h; j++) {
              dragArr.push(`${item.x + i}_${item.y + j}`)
            }
          }
        })
        return dragArr
      },
      initAllSlice() {
        let allArr = []
        for (let i = 0; i < this.dragMaxX; i++) {
          for (let j = 0; j < this.dragMaxY; j++) {
            allArr.push(`${i}_${j}`)
          }
        }
        return allArr
      },
      searchSpace(w = 1, h = 1) {
        this.dragSliceArr = this.initDragSlice()
        let newArray = this.allSliceArr.filter(item => {
          return this.dragSliceArr.indexOf(item) === -1
        })
        let findbox = { w: w, h: h }
        let resultJson = this.cumputerRest(newArray, findbox)
        return resultJson
      },
      cumputerRest(restArr, findbox) {
        let restJsonArr = []
        restArr.forEach(item => {
          let restJson = { x: item.split('_')[0], y: item.split('_')[1] }
          restJsonArr.push(restJson)
        })
        return this.filterRestData(findbox, restJsonArr)
      },
      filterRestData(findbox, restJsonArr) {
        let array = deepClone(restJsonArr)
        let arr = []
        let testYArr = []
        let num = 1
        for (let i = 0; i < array.length; i++) {
          testYArr.push(array[i].y)
          for (let j = i + 1; j < array.length; j++) {
            if (array[i].x === array[j].x) {
              testYArr.push(array[j].y)
              array.splice(j, 1)
              j--
              num++
            }
          }
          if (num >= findbox.h) {
            let arrItem = {
              x: array[i].x,
              yList: testYArr
            }
            arr.push(arrItem)
          }
          num = 1
          testYArr = []
        }

        let angeXList = []
        for (let i = 0; i < arr.length; i++) {
          arr[i].angeYList = this.arrange(arr[i].yList, findbox.h)
          if (arr[i].angeYList.length > 0) {
            angeXList.push(arr[i].x)
          }
        }

        angeXList = this.arrange(angeXList, findbox.w)
        let areList = []
        let sameXArr = []
        let comResult = { isAre: false, saveYArr: [] }
        for (let a = 0; a < angeXList.length; a++) {
          if (comResult.isAre) {
            break
          }
          for (let b = 0; b <= angeXList[a].length - findbox.w; b++) {
            comResult = { isAre: false, saveArr: [] }
            sameXArr = angeXList[a].slice(b, b + findbox.w)
            areList = []
            for (let c = 0; c < sameXArr.length; c++) {
              for (let all = 0; all < arr.length; all++) {
                if (Number(arr[all].x) === Number(sameXArr[c])) {
                  areList.push(arr[all].angeYList)
                }
              }
            }
            comResult = this.twoArrSameFind(areList, findbox.w, findbox.h)
            if (comResult.isAre) {
              break
            }
          }
        }
        return {
          isAre: comResult.isAre,
          sameYArr: deepClone(comResult.sameYArr),
          sameXArr: deepClone(sameXArr)
        }
      },
      twoArrSameFind(threeArr, lenx, leny) {
        let isAre = false
        let areNumber = 1
        let sameArr = []
        for (let t = 0; t <= threeArr.length - lenx; t++) {
          if (areNumber >= lenx && lenx > 1) {
            break
          }
          for (let s = 0; s < threeArr[t].length; s++) {
            if (areNumber >= lenx && lenx > 1) {
              break
            }
            for (let sd = 0; sd <= threeArr[t][s].length - leny; sd++) {
              sameArr = threeArr[t][s].slice(sd, sd + leny)
              areNumber = 1

              if (lenx <= 1) {
                isAre = true
                break
              } else {
                for (let l = t + 1; l < lenx; l++) {
                  for (let s2 = 0; s2 < threeArr[l].length; s2++) {
                    let newsameArr = sameArr.filter(item => threeArr[l][s2].indexOf(item) > -1)
                    if (this.isInclude(sameArr, newsameArr)) {
                      areNumber++
                      break
                    }
                  }
                }
                if (areNumber >= lenx) {
                  isAre = true
                  break
                }
              }
            }
          }
        }
        return { isAre: isAre, sameYArr: isAre ? sameArr : [] }
      },
      isInclude(aa, bb) {
        return aa.every((item) => {
          return bb.some((sub) => {
            return sub === item
          })
        })
      },
      arrange(source, number) {
        let t
        let ta
        let r = []
        for (let j = 0; j < source.length; j++) {
          let v = Number(source[j])
          if (v != null) {
            if (t === v) {
              ta.push(t)
              t++
              continue
            }
            ta = [v]
            t = v + 1
            r.push(ta)
          }
        }
        for (let i = r.length - 1; i >= 0; i--) {
          if (r[i].length < number) {
            r.splice(i, 1)
          }
        }
        return r
      },
      deleteDrag(item, index) {
        if (!item.canDel) {
          alert('当前模块不能删除')
          return
        }
        this.clearBoxActive()
        this.$emit('deldrag', item, index)
      },
      clearBoxActive() {
        this.homeDragList.forEach(item => {
          if (item.isActive) item.isActive = false
        })
      },
      DrawDrid() {
        if (!this.isEdit) return
        let canvas = document.getElementById('drag_bg')
        canvas.width = this.spaceSizeX * this.dragMaxX
        canvas.height = this.spaceSizeY * this.dragMaxY
        let paddingNumber = 10
        let ctx = canvas.getContext('2d')
        ctx.clearRect(0, 0, this.spaceSizeX * this.dragMaxX, this.spaceSizeY * this.dragMaxY)

        let spaceX = this.spaceSizeX
        let spaceY = this.spaceSizeY
        let x = 0; let y = 0
        ctx.setLineDash([5, 15])
        ctx.lineWidth = 2
        ctx.strokeStyle = '#efefef'
        for (y = spaceY; y < canvas.height; y += spaceY) {
          ctx.beginPath()
          ctx.moveTo(0, y)
          ctx.lineTo(canvas.width, y)
          ctx.stroke()
        }
        for (x = spaceX; x < canvas.width; x += spaceX) {
          ctx.beginPath()
          ctx.moveTo(x, 0)
          ctx.lineTo(x, canvas.height)
          ctx.stroke()
        }

        for (x = spaceX; x <= canvas.width; x += spaceX) {
          for (y = spaceY; y <= canvas.height; y += spaceY) {
            ctx.beginPath()
            ctx.lineWidth = 4
            ctx.strokeStyle = '#ededed'
            ctx.setLineDash([5, 5])
            ctx.moveTo(x - paddingNumber, y - paddingNumber)
            ctx.lineTo(x - paddingNumber, y - spaceY + paddingNumber)
            ctx.lineTo(x - spaceX + paddingNumber, y - spaceY + paddingNumber)
            ctx.lineTo(x - spaceX + paddingNumber, y - paddingNumber)
            ctx.closePath()
            ctx.stroke()
          }
        }
      }

    }
  }
</script>

<style scoped lang='scss'>
    .drag-content{
      border: 0px solid #ddd;
      background-color: #fefefe;
      position: relative;
      margin: auto;
      .draggable_custom{
        position: absolute;
        padding: 10px;
        box-sizing: border-box;
        &.vdr_eye{
          .drag-box{
            cursor: auto;
          }
        }
        .draggable_custom_closed{
          display: none;
          position: absolute;
          width: 20px;
          height: 20px;
          right: 20px;
          top:20px;
          color: red;
          cursor: pointer;
          font-size: 0px;
          img{
            width: 100%;
            height: 100%;
          }
          i{
            font-size: 20px;
          }
        }
        .drag-box{
          display: block;
          height: 100%;
          width: 100%;
          border: 2px solid transparent;
          box-sizing: border-box;
          border-radius: 5px;
          background-color:rgba(83, 168, 255,0.2);
          transition: all 0.4s;
          cursor: move;
        }
        &.active{
          &.vdr_eye .drag-box{
            box-shadow:0px  0px  8px transparent;
            border-color: transparent;
          }
          .draggable_custom_closed{
            display: block;
          }
          .drag-box{
            border: 2px solid #ccc;
            box-shadow: 0px 0px 8px rgba(0,0,0,0.2);
          }
        }
      }


    }
</style>
