<template>
  <div class="right flex flex-col">
    <p class="title">{{ currentProvider }}</p>
    <div @click="zoomIn" class="icon" title="放大"><Plus style="width: 2em; height: 2em" /></div>
    <div @click="zoomOut" class="icon" title="缩小"><Minus style="width: 2em; height: 2em" /></div>
    <div @click="zoomOut" class="icon" title="搜索地址"><Search style="width: 2em; height: 2em" /></div>
    <div @click="focusOn" class="icon" title="定位到本地"><Aim style="width: 2em; height: 2em" /></div>
    <div @click="isShowMoreBtn" class="icon" title="切换底图"><Switch style="width: 2em; height: 2em" /></div>
    <div v-if="isShowMore" class="btn-div">
      <div @click="switchToMap('Street_Map')" class="btn">街道地图</div>
      <div @click="switchToMap('Imagery')" class="btn">卫星地图</div>
      <div @click="switchToMap('Terrain')" class="btn">地形底图</div>
    </div>
  </div>
</template>
<script setup>
// 引用cesium
import * as Cesium from 'cesium/Build/Cesium';
import { ref } from 'vue';
const currentProvider = ref('底图');
let earthViewer;
let isShowMore = ref(false);
const intervalId = setInterval(() => {
  if (window.earthViewer) {
    earthViewer = window.earthViewer;
    clearInterval(intervalId);
  }
}, 1000);
const zoomIn = () => {
  earthViewer.scene.camera.zoomIn(1000); //相机向前移动1000米
};
const zoomOut = () => {
  earthViewer.scene.camera.zoomOut(1000); //相机向后移动1000米
};
const focusOn = () => {
  //获取当前设备的地理位置
  navigator.geolocation.getCurrentPosition(position => {
    const { latitude, longitude } = position.coords;
    earthViewer.camera.flyTo({
      destination: Cesium.Cartesian3.fromDegrees(longitude, latitude, 1000),
      duration: 3,
    });
  });
};
const isShowMoreBtn = () => {
  isShowMore.value = !isShowMore.value;
};
const switchToMap = type => {
  earthViewer.imageryLayers.removeAll();
  if (type === 'Terrain') {
    earthViewer.scene.setTerrain(new Cesium.Terrain(Cesium.CesiumTerrainProvider.fromIonAssetId(2426648)));
    currentProvider.value = type;
    return;
  }
  earthViewer.imageryLayers.addImageryProvider(
    new Cesium.UrlTemplateImageryProvider({
      url: `https://server.arcgisonline.com/ArcGIS/rest/services/World_${type}/MapServer/tile/{z}/{y}/{x}`,
      maximumLevel: 18,
    }),
  );
  currentProvider.value = type;
};
</script>
<style scoped>
.title {
  text-align: center;
  font-size: 1.5em;
  margin: 10px;
  color: #e0e0e0;
}
.right {
  width: 200px;
  height: calc(100% - 400px);
  /* background-color: rgba(0, 153, 204, 0.5); */
  position: absolute;
  right: 0;
  top: 200px;
  overflow: auto;
  box-sizing: border-box;
  align-items: flex-end;
}
.btn-div {
  position: absolute;
  right: 4em;
  bottom: 4em;
  .btn {
    width: 100px;
    height: 30px;
    background-color: #f0f0f0;
    margin: 10px 0;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
  }
}
</style>
