# drag-config-layout

> This is a drag-and-drop configuration of the page layout,so that users can customize their own look
>
>当前插件基于vue 2+，无需任何其他依赖

## installation
```js
# install
npm install --save drag-config-layout
```
## vueDragInfinite 引入方式
``` js
//全局引入
import Vue from 'vue'
import dragConfigLayout from 'drag-config-layout'

Vue.use(dragConfigLayout)
```
## 页面使用方式
```html
<template>
  <div class="drag-page">
    <drag-config-layout
            ref="draggable_name"
            :is-edit="isEdit"
            :data-list="homeDragList"
            :space-size="spaceSize"
            :drag-max-y="dragMaxY"
            :drag-max-x="dragMaxX"
            @deldrag="deleteDrag"
            @changing="dragChanging"
            @change="dragChange">
      <div slot="waitName" style="height: 100%">
        <div class="waitname">我是一个等待的自定义组件</div>
      </div>
      <div slot="dateName" style="height: 100%">
        <div class="datename">我是一个时间的自定义组件</div>
      </div>

    </drag-config-layout>
    <div class="button-group">
      <button v-if="!isEdit" @click="isEdit = true">编辑</button>
      <div v-else>
        <button @click="save">保存</button>
        <button @click="addDrag(1,1)">新增1*1</button>
        <button @click="addDrag(3,2)">新增3*2</button>
        <button @click="addDrag(4,2)">新增4*2</button>
        <button @click="addDrag(2,5)">新增2*5</button>
        <button @click="addDrag(5,1)">新增5*1</button>
        <button @click="addDragBg(0,2)">新增2行画布</button>
        <button @click="addDragBg(2,0)">新增2列画布</button>
      </div>
    </div>
  </div>
</template>

<script>

export default {
    name: 'dragpage',
    data(){
        return{
          isEdit: true, // 是否为编辑模式
          dragMaxX: 12, // X轴方向可拖拽的高度
          dragMaxY: 12, // y轴方向可拖拽的高度
          spaceSize: 100, // 每一块占位的宽度
          homeDragList: [
              {
                isActive: false, // 是否默认选中 必填
                canDel: true, // 是否能删除当前模块 必填
                w: 4, // 模块宽度的占位 最大为12个 必填
                h: 4, // 模块高度占位 最大为12个 必填
                x: 0, // 模块距离 左上角的占位的数量 必填
                y: 0, // 模块距离 左上角的占位的数量 必填
                minWidth: 2, // 模块宽度最小占位数量 必填
                minHeight: 2, // 模块高度最小占位数量 必填
                maxWidth: 12, // 模块宽度最大占位数量 必填
                maxHeight: 12, // 模块宽度最大占位数量 必填
                customClass: 'waitName', // 自定义类名  非必填
                slotName: 'waitName'// solt 名称  必填 且唯一  需要跟slot一一对应
              },
              {
                isActive: false, // 是否默认选中  必填
                canDel: true, // 是否能删除当前模块 必填
                w: 2, // 模块宽度的占位 最大为12个 必填
                h: 3, // 模块高度占位 最大为12个 必填
                x: 4, // 模块距离 左上角的占位的数量 必填
                y: 0, // 模块距离 左上角的占位的数量 必填
                minWidth: 2, // 模块宽度最小占位数量 必填
                minHeight: 2, // 模块高度最小占位数量 必填
                maxWidth: 12, // 模块宽度最大占位数量 必填
                maxHeight: 12, // 模块宽度最大占位数量 必填
                customClass: 'dateName', // 自定义类名  非必填
                slotName: 'dateName'// solt 名称  必填 且唯一 需要跟slot一一对应
              }
          ]
        } 
    },
    methods: {
         /*
        * 拖拽放大过程中方法的回调 操作的当前对象
        * */
        dragChanging(item) {
          console.log(item, 'item')
        },
        /*
        * 拖拽放大结束后返回值  item 为当前操作的数据   data为整体数据
        * */
        dragChange(item, data) {
          this.homeDragList = data
        },
        /*
        * 保存校验是否有未填充的空间
        * 通过切片形式校验
        */
        save() {
          let resultJson = this.$refs.draggable_name.searchSpace()
          if (resultJson.isAre) { // 如果有空间 新增
            console.log('还有剩余空间未填写，是否继续保存')
          } else {
            console.log('保存成功！')
            this.isEdit = false
          }
        },
        // 新增俩行画布 编辑模式下才有当前按钮
        addDragBg(x, y) {
          this.dragMaxX += x
          this.dragMaxY += y
        },
        //  新增一条数据
        addDrag(w, h) {
          let newDragItem = {
            isActive: false, // 是否默认选中  必填
            canDel: true, // 是否能删除当前模块 必填
            w: w, // 模块宽度的占位 最大为12个 必填
            h: h, // 模块高度占位 最大为12个 必填
            x: 4, // 模块距离 左上角的占位的数量 必填
            y: 0, // 模块距离 左上角的占位的数量 必填
            minWidth: 1, // 模块宽度最小占位数量 必填
            minHeight: 1, // 模块高度最小占位数量 必填
            maxWidth: 12, // 模块宽度最大占位数量 必填
            maxHeight: 12, // 模块宽度最大占位数量 必填
            customClass: 'className_' + this.homeDragList.length, // 自定义类名  非必填
            slotName: 'ZlSaasItem_' + this.homeDragList.length// solt 名称  必填 且唯一 需要跟slot一一对应
          }
          /*
          * searchSpace方法 通过ref的形式 检索空白区域是否符合
          * 参数说明 w代表宽度  h 代表长度 默认为1
          * 返回值说明  isAre 是否有符合条件的空间
          *             sameXArr x轴方向符合的空间  起始点取第一个
          *             sameYArr y轴方向符合的空间  起始点取第一个
          * */
          let resultJson = this.$refs.draggable_name.searchSpace(w, h)
          if (resultJson.isAre) { // 如果有空间 新增
            newDragItem.x = resultJson.sameXArr[0]
            newDragItem.y = resultJson.sameYArr[0]
    
            this.homeDragList.push(newDragItem)
          } else { // 如果没有 提示
            console.log(`没有可用空间，请先预留好${w}*${h}位置`)
          }
        },
        /*
        * 删除的回调 item当前数据 data 总数据
        * */
        deleteDrag(item, index) {
          this.homeDragList.splice(index, 1)
        }
    }
}
</script>

<style>
.drag-page{
    padding: 20px;
    background-color: #efefef;
}
.button-group{
    padding: 20px 0px 0px;
    font-size: 20px;
}
</style>

```

### Attributes
 | 参数 | 说明 |  类型 | 可选值 | 默认值 |
 | --------    | :----- | :----  |:----- | :----  |
 | spaceSize | 占位块的大小，宽度会跟随屏幕大小计算，最小值为当前值，高度为当前值 | Number | - | 100 |
 | isEdit | 是否为编辑模式 | Boolean | true/false | true |
 | dragMaxX | x轴方向可进行拖拽的块数 | Number | - | 12 |
 | dragMaxY | y轴方向可进行拖拽的块数 | Number | - | 12 |
 | dataList | 当前画布可操作的数据 | Object | - | [] |

### dataList Attributes

| 参数 | 说明 |  类型 | 可选值 | 默认值 | 是否必填 |
 | --------    | :----- | :----  |:----- | :----  | :---- |
 | isActive | 是否默认选中 | Boolean | true/false | - | 必填 |
 | canDel |  是否能删除当前模块 | Boolean | true/false | - | 必填 |
 | w | 模块宽度的占位 最大为12个 不超过 dragMaxX的大小 | Number | - | - | 必填 |
 | h | 模块宽度的占位 最大为12个 不超过 dragMaxY的大小 | Number | - |  -| 必填 |
 | x | 模块距离 左上角的占位的数量 x轴方向 | Number | - | - | 必填 |
 | y | 模块距离 左上角的占位的数量 y轴方向| Number | - | - | 必填 |
 | minWidth | 模块宽度最小占位数量| Number | - | - | 必填 |
 | minHeight | 模块高度最小占位数量| Number | - | - | 必填 |
 | maxWidth | 模块宽度最大占位数量| Number | - | - | 必填 |
 | maxHeight | 模块高度最大占位数量| Number | - | - | 必填 |
 | customClass | 自定义class类名| string | - | - | 非必填 |
 | slotName | solt 名称  必填 且唯一  需要跟slot一一对应| string | - | - | 必填 |

### Methods

| 方法名        | 说明    |  参数   |
| --------    | :----- | :---- |
| dragChanging  | 拖拽放大过程中方法的回调 操作的当前对象  |   (item) 操作的当前对象   |
| dragChange   |   移动放大结束后方法的回调  |   (item,data) item 为当前操作的数据   data为整体数据   |
| searchSpace   |  通过ref的形式 检索空白区域是否符合  |   参数说明 w代表宽度  h 代表长度 默认为1,返回值说明  isAre 是否有符合条件的空间,sameXArr x轴方向符合的空间  起始点取第一个,sameYArr y轴方向符合的空间  起始点取第一个 |
|deldrag| 删除当前模块的回调 | （item,index）item当前操作的模块，index索引值 |