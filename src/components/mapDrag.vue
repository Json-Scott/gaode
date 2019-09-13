<template>
  <div class="m-map">
    <div id="map" class="map">正在加载数据 ...</div>
    <div id="address"></div>
    <div id="getLive"></div>
    <div id="weather"></div>
  </div>
</template>

<script>
import remoteLoad from "@/utils/remoteLoad.js";
import { MapKey, MapCityName } from "@/config/config";
export default {
  props: ["lat", "lng"],
  data() {
    return {
      AMapUI: null,
      AMap: null,
      mapZoom: 8, // 地图初始化缩放
      MLocation: 0, // 存储当前位置信息
      OLocation: 0, // 存储当前位置信息
      district: "", // 存储当前点击的区域
      weather: "" // 存储当前点击的区域天气信息
    };
  },
  methods: {
    // 实例化地图
    initMap() {
      let self = this;
      // 加载PositionPicker，loadUI的路径参数为模块名中 'ui/' 之后的部分
      let AMapUI = (this.AMapUI = window.AMapUI);
      let AMap = (this.AMap = window.AMap);
      AMapUI.loadUI(["misc/PositionPicker"], PositionPicker => {
        let mapConfig = {
          zoom: 16,
          cityName: MapCityName
        };
        // 创建实例
        let map = new AMap.Map("map", mapConfig);
        map.setZoom(this.mapZoom);

        // 标记
        let marker = new AMap.Marker({
          map: map,
          position: [self.MLocation, self.OLocation],
          label: {
            offset: new AMap.Pixel(20, 20), //修改label相对于maker的位置
            content: ""
          }
        });
        AMap.plugin("AMap.Geolocation", function() {
          let geolocation = new AMap.Geolocation({
            enableHighAccuracy: true, //是否使用高精度定位，默认:true
            timeout: 10000, //超过10秒后停止定位，默认：无穷大
            maximumAge: 0, //定位结果缓存0毫秒，默认：0
            convert: true, //自动偏移坐标，偏移后的坐标为高德坐标，默认：true
            showButton: true, //显示定位按钮，默认：true
            showMarker: true, //定位成功后在定位到的位置显示点标记，默认：true
            panToLocation: true, //定位成功后将定位到的位置作为地图中心点，默认：true
            zoomToAccuracy: true //定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
          });
          geolocation.getCurrentPosition(function(state, data) {
            self.MLocation = data.position.M;
            self.OLocation = data.position.O;
          });
          AMap.event.addListener(geolocation, "complete", onComplete); //返回定位信息
          AMap.event.addListener(geolocation, "error", onError); //返回定位出错信息
          function onComplete(data) {
            // data是具体的定位信息
          }
          function onError(data) {
            // console.log("出错了", data);
          }
        });
        // 点击事件
        let clickMap = map.on("click", function(e) {
          let lnglatXY = [e.lnglat.getLng(), e.lnglat.getLat()]; //已知点坐标
          map.setZoomAndCenter(14, lnglatXY); // 点击后放大当前位置, 可以解决点击之后maker不见的bug
          marker.setPosition(lnglatXY);
          regeocoder(lnglatXY);
          // 天气
          AMap.plugin("AMap.Weather", function() {
            //创建天气查询实例
            let weather = new AMap.Weather();
            //执行实时天气信息查询, 获取位置信息
            map.getCity(function(info) {
              weather.getLive(info.district, function(err, data) {
                self.district = info.district;
                self.weather = data.weather;
                document.getElementById("getLive").innerHTML = 'getLive传入的信息: ' + self.district;
                document.getElementById("weather").innerHTML = '天气: ' + self.weather;
              });
            });
          });
        });
        //逆地理编码
        function regeocoder(lnglatXY) {
          AMap.plugin("AMap.Geocoder", function() {
            let geocoder = new AMap.Geocoder({
              radius: 1000,
              extensions: "all"
            });
            geocoder.getAddress(lnglatXY, function(status, result) {
              if (status === "complete" && result.info === "OK") {
                let address = result.regeocode.formattedAddress; //返回地址描述
                document.getElementById("address").innerHTML = '地址: ' + address;
              }
            });
          });
        }
        // 启用工具条
        AMap.plugin(["AMap.ToolBar"], function() {
          map.addControl(
            new AMap.ToolBar({
              position: "RB"
            })
          );
        });
        // 创建地图拖拽
        let positionPicker = new PositionPicker({
          mode: "dragMap", // 可选'dragMarker'，默认为'dragMap'
          map: map // 依赖地图对象
        });
        // 拖拽完成发送自定义 drag 事件
        positionPicker.on("success", positionResult => {
          this.$emit("drag", positionResult);
        });
        // 启动拖放
        positionPicker.start();
      });
    }
  },
  async created() {
    // 已载入高德地图API，则直接初始化地图
    if (window.AMap && window.AMapUI) {
      this.initMap();
      // 未载入高德地图API，则先载入API再初始化
    } else {
      await remoteLoad(`http://webapi.amap.com/maps?v=1.3&key=${MapKey}`);
      await remoteLoad("http://webapi.amap.com/ui/1.0/main.js");
      this.initMap();
    }
  }
};
</script>

<style lang="stylus" scoped>
@import '~@/assets/styles/stylus.styl';
.m-map
  width px2rem(375)
  min-height 300px
  position relative
  .map 
    width: 100%
    height: 100%
.removeMaker 
  display none
</style>
