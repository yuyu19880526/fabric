<template>
  <div class="hello">
    <button @click="addAnchor">添加</button>
    <button @click="getLocation">获取坐标</button>
    <button @click="remove">删除锚点</button>
    <button @click="e => createAtt(this.st, this.ed)">显示攻击</button>
    <button @click="clearAttackState">停止攻击</button>
    <canvas id="c">
      <img src="./路径@2x.png" ref="posImg"/>
    </canvas>
    <canvas class="bg1" id="attack-graph">
        <!-- <img src="./路径@2x.png" ref="posImg"/> -->
        <img class="attack-graph__img" ref="$attack" src="http://p0.qhimg.com/t01213fa53d9547084c.png" width="170" height="96" />
        <img class="attack-graph__img" ref="$beAttackAlert" src="http://p0.qhimg.com/t014c8c457d1f5c44d6.png" width="198" height="206" />
      </canvas>
    <!-- <img src="./形状结合.png" ref="deleteIcon"> -->
    <a-modal
      :visible="visible"
      @cancel="handleCancel"
      :mask="false"
      :footer="false"
    >
      <div v-if="showName">
        <p>{{ res.name }}</p>
        <a-button type="primary" @click="showName = false">
          添加下挂资产组
        </a-button>
        <a-button @click="remove">
          删除锚点
        </a-button>
      </div>
      <div v-if="!showName">
        添加下挂资产组
        <a-transfer
          :data-source="mockData"
          show-search
          :titles="['资产组列表', '下挂资产组']"
          :target-keys="targetKeys"
          :selected-keys="selectedKeys"
          :render="item => item.title"
          @change="handleChange"
          @selectChange="handleSelectChange"
          @scroll="handleScroll"
        />
        <div class="modalFooter">
          <a-button key="back" @click="handleCancel">
          取消
          </a-button>
          <a-button key="submit" type="primary" :loading="loading" @click="handleOk">
            确定
          </a-button>
        </div>
      </div>
    </a-modal>
  </div>
</template>

<script>
import { fabric } from 'fabric'
import _ from 'lodash'
const settings = {
  imageWidth: 198,
  imageHeight: 206
}
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    const mockData = [];
    for (let i = 0; i < 20; i++) {
      mockData.push({
        key: i.toString(),
        title: `content${i + 1}`,
        description: `description of content${i + 1}`,
        disabled: i % 3 < 1,
      });
    }
    const oriTargetKeys = mockData.filter(item => +item.key % 3 > 1).map(item => item.key);
    return {
      count: 1,
      showName: true,
      canvasObj: null,
      caWidth: 1380,
      loading: false,
      caHeight: 720,
      st: '1',
      ed: '3',
      imgcoordinate: [],
      currentAttackInstance: null,
      res: {},
      bgsrc: 'http://p0.qhimg.com/t01c582d25048128698.png',
      visible: false,
      mockData,
      targetKeys: oriTargetKeys,
      selectedKeys: ['1', '4'],
      config: {
        list: [
          {
              id: 1,
              name: '锚点1',
              x: 486, // 492
              y: 111, // 151,
              count: 10
            },
            {
              id: 2,
              name: '锚点2',
              x: 389,
              y: 242,
              count: 20
            },
            {
              id: 3,
              name: '锚点3',
              x: 666,
              y: 208,
              count: 30
            },
            {
              id: 4,
              name: '锚点4',
              x: 717,
              y: 242,
              count: 8
            }
        ]
      }
    }
  },
  mounted() {
    this.initCavas()
  },
  methods: {
    initCavas() {
      fabric.Object.prototype.hasControls = false
      fabric.Object.prototype.hasBorders = true
      fabric.Object.prototype.set({
        borderColor: '#00FCFF',
        borderScaleFactor: 1,
        padding: 0
      })
      this.canvasObj = new fabric.Canvas('c',{
        preserveObjectStacking:true // 禁止选中图层时自定置于顶部
      });
      this.canvasObj.setWidth(this.caWidth); //设置画布宽度
      this.canvasObj.setHeight(this.caHeight);
      this.canvasObj.setBackgroundImage(this.bgsrc, this.canvasObj.renderAll.bind(this.canvasObj), {
        width: this.canvasObj.width, 
        height: this.canvasObj.height, 
        originX: 'left', 
        originY: 'top',
        selectable: false
      });
    },
    addAnchor() {
      const circle1 = new fabric.Circle({
        radius: 12,
        fill: '#00FCFF',
        originX: 'center', //调整中心点的X轴坐标
        originY: 'center' //调整中心点的Y轴坐标
      })
      const imgInstance = new fabric.Image(this.$refs.posImg, {
        left: -6,
        top: -6,
        scaleX: 0.6,
        scaleY: 0.6,
        lockScalingFlip: false,
        lockUniScaling: false
      })
      const group1 = new fabric.Group([circle1, imgInstance], {
        left: 6,
        top: 0,
        angle: 0
      })
      const circle = new fabric.Circle({
        radius: 8,
        fill: '#f00',
        originX: 'center', //调整中心点的X轴坐标
        originY: 'center' //调整中心点的Y轴坐标
      });
      const text = new fabric.Text('0', { //绘制文本
        fontSize: 10,
        originX: 'center',
        originY: 'center',
        fill: '#fff'
      })
      const rect = new fabric.Rect({
        fill:'#00FCFF',//填充的颜色
        width:40,//方形的宽度
        height:20//方形的高度
      })
      const text1 = new fabric.Text('锚点' + this.count, { //绘制文本
        fontSize: 11,
        top: 10,
        left: 16,
        originX: 'center',
        originY: 'center',
        fill: '#262626'
      })
      const group3 = new fabric.Group([rect, text1], {
        left: 0,
        top: 30,
        angle: 0
      })
      const group2 = new fabric.Group([circle, text], {
        left: 24,
        top: -10,
        angle: 0
      })
      const groups = new fabric.Group([group3, group1, group2], {
        left: 100,
        top: 100,
        angle: 0
      })
      groups.id = this.count
      groups.name = '锚点' + this.count
      this.count ++
      groups.on('moving', () => {
        this.handleCancel()
      })
      groups.on('mousedown', (e) => {
        if(e.target) {
          e.target.opacity = 0.5;
          this.canvasObj.renderAll()
        }
      })
      groups.on('mouseup', (e) => {
        if(e.target) {
          e.target.opacity = 1;
          this.canvasObj.renderAll()
          // animate()
          this.openModal()
        }
      })
      groups.on('selected', (e) => {
        if(e.target) {
          e.target.opacity = 1;
          this.canvasObj.renderAll()
          this.res = Object.assign({}, {
            name: e.target.name,
            id: e.target.id
          })
          // this.openModal()
        }
      })
      this.canvasObj.add(groups).setActiveObject(groups);
    },
    openModal() {
      this.showName = true
      this.visible = true
    },
    handleCancel() {
      this.visible = false
    },
    handleOk() {
      console.log(454545)
    },
    getLocation() {
      this.imgcoordinate = []
      let items = this.canvasObj.getObjects()
      items.forEach((item, index) => {
        // tl 代表左上角  tr表示右上角  bl表示左下角  br表示右下角
        let itemcoord = {
          index: index,
          id: item.id,
          name: item.name,
          bl: {
            x: item.aCoords.bl.x,
            y: item.aCoords.bl.y
          }
        }
        this.imgcoordinate.push(itemcoord)
      })
      console.log(this.imgcoordinate)
    },
    createAtt(st, ed) {
      // 攻击锚点绘制
      this.clearAttackState()
      const startConfig = _.find(this.config.list, s => s.id.toString() === st.toString())
      const endConfig = _.find(this.config.list, s => s.id.toString() === ed.toString())
      if (startConfig && endConfig) {
        // 攻击路径
        this.createAttackPath(startConfig, endConfig)
        this.drawAttackIcon({
          left: startConfig.x,
          top: startConfig.y,
          id: startConfig.id,
          name: startConfig.name
        })

        this.createBeAttacked({
          left: endConfig.x,
          top: endConfig.y,
          id: endConfig.id,
          name: endConfig.name
        })
      }
    },
    drawAttackIcon(config) {
      const top = config.top - 30
      const left = config.left - 30
      var imgInstance = new fabric.Image(
        this.$refs.$attack,
        Object.assign({
          left,
          top,
          angle: 0,
          x2: 0,
          y2: 0,
          opacity: 0.5,
          width: 170,
          height: 96,
          selectable: false
        })
      )
      const _t = this
      const anim = () => {
        if (imgInstance === this.currentAttackInstance) {
          imgInstance.opacity = 0.5
          imgInstance.animate(
            {
              opacity: 1
            },
            {
              duration: 500,
              abort: () => imgInstance !== _t.currentAttackInstance,
              onChange: this.canvasObj.renderAll.bind(this.canvasObj),
              onComplete: anim
            }
          )
        }
      }
      console.log(imgInstance)
      this.canvasObj.add(imgInstance)
      this.currentAttackInstance = imgInstance
      anim()
    },
    createAttackPath(startConfig, endConfig) {
      const start = {
        x: startConfig.x + settings.imageWidth / 10,
        y: startConfig.y + settings.imageWidth / 5
      }
      const end = {
        x: endConfig.x + settings.imageWidth / 14,
        y: endConfig.y + settings.imageWidth / 4
      }

      const ot = {
        x: start.x + (end.x - start.x) / 2,
        y: end.y > start.y ? start.y - 50 : end.y - 50
      }
      var gradient = new fabric.Gradient({
        type: 'linear',
        coords: { x1: 0, y1: 0, x2: Math.abs(end.x - start.x), y2: 0 },
        colorStops: [
          { offset: 0, color: '#FF2F26' },
          { offset: 1, color: '#FFCA00' }
        ]
      })

      var line = new fabric.Path(`M ${start.x} ${start.y} Q ${ot.x}, ${ot.y}, ${end.x}, ${end.y}`, {
        fill: '',
        stroke: gradient,
        strokeLineCap: 'round',
        strokeWidth: 3,
        // strokeDashArray: [10, 10],
        strokeDashOffset: 10,
        selectable: false,
        objectCaching: false
      })
      const _t = this
      const anim = () => {
        if (line === this.currentAttackPath) {
          // line.scaleX = 0
          line.strokeDashOffset = 10
          line.animate(
            {
              strokeDashOffset: 0
            },
            {
              duration: 500,
              abort: () => line !== _t.currentAttackPath,
              onChange: this.canvasObj.renderAll.bind(this.canvasObj),
              onComplete: anim
            }
          )
        }
      }
      this.canvasObj.add(line)
      this.currentAttackPath = line
      anim()
    },
    createBeAttacked(config) {
      const $beAttackAlert = new fabric.Image(
        this.$refs.$beAttackAlert,
        Object.assign({
          left: 45,
          top: -30,
          angle: 0,
          x2: 0,
          y2: 0,
          opacity: 1,
          width: 194,
          height: 194,
          scaleX: 0.5,
          scaleY: 0.5,
          selectable: false
        })
      )
      const group = new fabric.Group(
        [$beAttackAlert],
        Object.assign(config, {
          width: settings.imageWidth,
          height: settings.imageHeight,
          selectable: false,
          top: config.top - 70,
          left: config.left - 90
        })
      )

      this.currentBeAttackedInstance = group
      var _t = this
      this.canvasObj.add(group)
      const anim = () => {
        if (group === _t.currentBeAttackedInstance) {
          $beAttackAlert.opacity = 0.5
          $beAttackAlert.animate(
            {
              opacity: 1
            },

            {
              duration: 500,
              abort: () => group !== _t.currentBeAttackedInstance,
              onChange: this.canvasObj.renderAll.bind(this.canvasObj),
              onComplete: anim
            }
          )
        }
      }
      anim()
    },
    clearAttackState() {
      this.currentAttackPath && this.canvasObj.remove(this.currentAttackPath)
      this.currentAttackInstance && this.canvasObj.remove(this.currentAttackInstance)
      this.currentBeAttackedInstance && this.canvasObj.remove(this.currentBeAttackedInstance)
      this.currentAttackPath = null
      this.currentAttackInstance = null
      this.currentBeAttackedInstance = null
    },
    remove() {
      let t = this.canvasObj.getActiveObject()
      this.canvasObj.remove(t)
      this.canvasObj.requestRenderAll()
      this.visible = false
    },
    Attack() {

    },
    handleChange(nextTargetKeys, direction, moveKeys) {
      this.targetKeys = nextTargetKeys;
      console.log('targetKeys: ', nextTargetKeys);
      console.log('direction: ', direction);
      console.log('moveKeys: ', moveKeys);
    },
    handleSelectChange(sourceSelectedKeys, targetSelectedKeys) {
      this.selectedKeys = [...sourceSelectedKeys, ...targetSelectedKeys];

      console.log('sourceSelectedKeys: ', sourceSelectedKeys);
      console.log('targetSelectedKeys: ', targetSelectedKeys);
    },
    handleScroll(direction, e) {
      console.log('direction:', direction);
      console.log('target:', e.target);
    },
    handleDisable(disabled) {
      this.disabled = disabled;
    }
  }
}
</script>

<style scoped>
.attack-graph__img {
  display: none;
}
</style>
