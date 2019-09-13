## 运行项目
```
# 安装依赖
npm install / cnpm install / yarn
# 运行项目
npm run serve
```

## 实现思路

* 创建一个 mapDrag 的公共组件
* 在组件的 created 生命周期，载入高德地图API
* API载入完成即开始执行地图初始化
* 地图API暴露全局变量 window.AMap 可以直接使用
* 监听地图拖放事件，获得数据后通知自定义事件，对组件外部提供事件接口

## created 生命周期载入高德地图API

import remoteLoad from "@/utils/remoteLoad.js";
import { MapKey, MapCityName } from "@/config/config";

## 将用到的配置文件引入子组件, 第一个是远程加载高德API, 第二个为使用高德的配置项

``` javascript
async created () {
  // 已载入高德地图API，则直接初始化地图
  if (window.AMap && window.AMapUI) {
    this.initMap()
  // 未载入高德地图API，则先载入API再初始化
  } else {
    await remoteLoad(`http://webapi.amap.com/maps?v=1.3&key=${MapKey}`)
    await remoteLoad('http://webapi.amap.com/ui/1.0/main.js')
    this.initMap()
  }
}
```
## 初始化地图
在 methods 中创建一个 initMap 的方法供载入地图API之后调用。这里就可以使用任意高德API
``` javascript
initMap () {
  // 加载PositionPicker，loadUI的路径参数为模块名中 'ui/' 之后的部分
  let AMapUI = this.AMapUI = window.AMapUI
  let AMap = this.AMap = window.AMap
  AMapUI.loadUI(['misc/PositionPicker'], PositionPicker => {
    let mapConfig = {
      zoom: 16,
      cityName: MapCityName
    }
    let map = new AMap.Map('map', mapConfig)
    // 工具条
    AMap.plugin(['AMap.ToolBar'], function () {
      map.addControl(new AMap.ToolBar({
        position: 'RB'
      }))
    })
    // 创建地图拖拽
    let positionPicker = new PositionPicker({
      mode: 'dragMap', // 'dragMarker'，默认为'dragMap'
      map: map // 依赖地图对象
    })
    // 拖拽完成发送自定义 drag 事件
    positionPicker.on('success', positionResult => {
      this.$emit('drag', positionResult)
    })
    // 启动拖拽
    positionPicker.start()
  })
}
```

## 调用
``` html
<mapDrag @drag="dragMap" />
```
## 总结

### 经过实际操作发现 vue-amap 坑比较多,所以最终决定直接引入原生的高德 API

### 缺点: 

#### 1. 对于高德 API 的了解不是很熟悉

#### 2. 子组件的业务逻辑的写法不是很精简

### 优点:

#### 1. 使用的时候直接引入组件即可,比较好维护

#### 2. 因为引入了原生 API , 故操作起来较为方便, 兼容性较好

### 对于我:

#### 第一次接触地图类的代码, 有些慌乱, 但经过阅读 API 学习之后, 踩了些坑, 增长了新的经验, 很符合自己的职业规划

#### 在此次 coding 同时, 我走了个近路, 没有去调用请求接口拿数据, 而是调用高德封装好的 API , 直接可以拿到数据, 但因此类项目没有接触过, 不知道这种做法是否合规?

#### 因为自己的经验较少, 感到有些失落, 望以后的自己变得更强

### 展望:

#### 未来将系统的学习高德 API , 将代码写的更加可维护且易读

#### 决定一次次的重构, 将代码变得更好