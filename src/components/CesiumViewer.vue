<template>
  <div class="cesium-container">
    <div id="cesiumContainer" ref="cesiumContainer"></div>
    
    <!-- 控制面板 -->
    <div class="control-panel">
      <h3>场景控制</h3>
      
      <div class="control-section">
        <h4>相机控制</h4>
        <button @click="flyToBeijing">飞行到北京</button>
        <button @click="flyToShanghai">飞行到上海</button>
        <button @click="flyToChengdu">飞行到成都</button>
        <button @click="resetView">重置视图</button>
      </div>
      
      <div class="control-section">
        <h4>地形设置</h4>
        <label>
          <input type="checkbox" v-model="terrainEnabled" @change="toggleTerrain" />
          启用地形
        </label>
      </div>
      
      <div class="control-section">
        <h4>影像底图</h4>
        <select v-model="selectedImagery" @change="changeImagery">
          <option value="arcgis">ArcGIS 影像</option>
          <option value="天地图影像">天地图影像</option>
        </select>
      </div>
      
      <div class="control-section">
        <h4>实体添加</h4>
        <button @click="addBox">添加立方体</button>
        <button @click="addCylinder">添加圆柱</button>
        <button @click="addPoint">添加点标注</button>
        <button @click="clearEntities">清除所有实体</button>
      </div>
      
      <div class="control-section">
        <h4>当前视角信息</h4>
        <div class="info-text">
          <p>经度: {{ cameraInfo.longitude }}°</p>
          <p>纬度: {{ cameraInfo.latitude }}°</p>
          <p>高度: {{ cameraInfo.height }} m</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as Cesium from 'cesium'

const cesiumContainer = ref(null)
let viewer = null
let updateCameraHandler = null

const terrainEnabled = ref(true)
const selectedImagery = ref('天地图影像')
const cameraInfo = ref({
  longitude: 0,
  latitude: 0,
  height: 0
})

onMounted(() => {
  initCesium()
})

onBeforeUnmount(() => {
  if (updateCameraHandler) {
    updateCameraHandler()
  }
  if (viewer) {
    viewer.destroy()
  }
})

// 初始化Cesium
function initCesium() {
  // 设置Cesium ion访问令牌
  Cesium.Ion.defaultAccessToken = import.meta.env.VITE_CESIUM_ION_TOKEN
  
  viewer = new Cesium.Viewer(cesiumContainer.value, {
    imageryProvider: false,  // 先不加载任何影像
    terrain: Cesium.Terrain.fromWorldTerrain(),
    timeline: false,
    animation: false,
    baseLayerPicker: false,
    geocoder: false,
    homeButton: true,
    sceneModePicker: true,
    navigationHelpButton: false,
    infoBox: true,
    selectionIndicator: true,
    shouldAnimate: true
  })
  
  // 手动添加天地图影像作为默认底图
  const tiandituToken = import.meta.env.VITE_TIANDITU_TOKEN
  const defaultImagery = new Cesium.UrlTemplateImageryProvider({
    url: `https://t{s}.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=${tiandituToken}`,
    subdomains: ['0', '1', '2', '3', '4', '5', '6', '7'],
    maximumLevel: 18,
    credit: new Cesium.Credit('天地图全球影像服务')
  })
  viewer.imageryLayers.addImageryProvider(defaultImagery)
  
  // 启用光照效果
  viewer.scene.globe.enableLighting = false
  
  // 设置初始视角到中国
  viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(104.0, 35.0, 10000000.0),
    orientation: {
      heading: Cesium.Math.toRadians(0.0),
      pitch: Cesium.Math.toRadians(-90.0),
      roll: 0.0
    }
  })
  
  // 监听相机移动事件，更新相机信息
  updateCameraHandler = viewer.camera.changed.addEventListener(() => {
    updateCameraInfo()
  })
  
  // 初始更新一次相机信息
  updateCameraInfo()
  
  // 添加一些初始地标
  addInitialLandmarks()
  
  console.log('Cesium viewer initialized successfully')
}

// 更新相机信息
function updateCameraInfo() {
  const position = viewer.camera.positionCartographic
  cameraInfo.value = {
    longitude: Cesium.Math.toDegrees(position.longitude).toFixed(6),
    latitude: Cesium.Math.toDegrees(position.latitude).toFixed(6),
    height: position.height.toFixed(2)
  }
}

// 添加初始地标
function addInitialLandmarks() {
  // 北京天安门
  viewer.entities.add({
    name: '北京天安门',
    position: Cesium.Cartesian3.fromDegrees(116.3975, 39.9075, 100),
    point: {
      pixelSize: 10,
      color: Cesium.Color.RED,
      outlineColor: Cesium.Color.WHITE,
      outlineWidth: 2
    },
    label: {
      text: '北京',
      font: '14pt sans-serif',
      style: Cesium.LabelStyle.FILL_AND_OUTLINE,
      outlineWidth: 2,
      verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
      pixelOffset: new Cesium.Cartesian2(0, -10)
    }
  })
  
  // 上海东方明珠
  viewer.entities.add({
    name: '上海东方明珠',
    position: Cesium.Cartesian3.fromDegrees(121.4995, 31.2397, 100),
    point: {
      pixelSize: 10,
      color: Cesium.Color.BLUE,
      outlineColor: Cesium.Color.WHITE,
      outlineWidth: 2
    },
    label: {
      text: '上海',
      font: '14pt sans-serif',
      style: Cesium.LabelStyle.FILL_AND_OUTLINE,
      outlineWidth: 2,
      verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
      pixelOffset: new Cesium.Cartesian2(0, -10)
    }
  })
  
  // 成都
  viewer.entities.add({
    name: '成都',
    position: Cesium.Cartesian3.fromDegrees(104.0665, 30.5723, 100),
    point: {
      pixelSize: 10,
      color: Cesium.Color.GREEN,
      outlineColor: Cesium.Color.WHITE,
      outlineWidth: 2
    },
    label: {
      text: '成都',
      font: '14pt sans-serif',
      style: Cesium.LabelStyle.FILL_AND_OUTLINE,
      outlineWidth: 2,
      verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
      pixelOffset: new Cesium.Cartesian2(0, -10)
    }
  })
}

// 飞行到北京
function flyToBeijing() {
  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(116.3975, 39.9075, 50000.0),
    duration: 2.0
  })
}

// 飞行到上海
function flyToShanghai() {
  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(121.4995, 31.2397, 50000.0),
    duration: 2.0
  })
}

// 飞行到成都
function flyToChengdu() {
  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(104.0665, 30.5723, 50000.0),
    duration: 2.0
  })
}

// 重置视图
function resetView() {
  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(104.0, 35.0, 10000000.0),
    orientation: {
      heading: Cesium.Math.toRadians(0.0),
      pitch: Cesium.Math.toRadians(-90.0),
      roll: 0.0
    },
    duration: 2.0
  })
}

// 切换地形
function toggleTerrain() {
  if (terrainEnabled.value) {
    viewer.scene.setTerrain(Cesium.Terrain.fromWorldTerrain())
  } else {
    viewer.scene.setTerrain(new Cesium.Terrain(new Cesium.EllipsoidTerrainProvider()))
  }
}

// 切换影像底图
async function changeImagery() {
  viewer.imageryLayers.removeAll()
  
  let imageryProvider
  
  switch (selectedImagery.value) {
    case 'arcgis':
      // ArcGIS需要异步创建并等待准备就绪
      imageryProvider = await Cesium.ArcGisMapServerImageryProvider.fromUrl(
        'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer'
      )
      break
    case '天地图影像':
      // 使用UrlTemplateImageryProvider，这是更可靠的方式
      const tiandituToken = import.meta.env.VITE_TIANDITU_TOKEN || 'ba55e7dd64ebeeb9706159a0e2aa2e84'
      imageryProvider = new Cesium.UrlTemplateImageryProvider({
        url: `https://t{s}.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=${tiandituToken}`,
        subdomains: ['0', '1', '2', '3', '4', '5', '6', '7'],
        maximumLevel: 18,
        credit: new Cesium.Credit('天地图全球影像服务')
      })
      break
    default:
      const defaultTiandituToken = import.meta.env.VITE_TIANDITU_TOKEN || 'ba55e7dd64ebeeb9706159a0e2aa2e84'
      imageryProvider = new Cesium.UrlTemplateImageryProvider({
        url: `https://t{s}.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=${defaultTiandituToken}`,
        subdomains: ['0', '1', '2', '3', '4', '5', '6', '7'],
        maximumLevel: 18,
        credit: new Cesium.Credit('天地图全球影像服务')
      })
  }
  
  viewer.imageryLayers.addImageryProvider(imageryProvider)
  console.log('Imagery changed to:', selectedImagery.value)
}

// 添加立方体
function addBox() {
  const position = viewer.camera.pickEllipsoid(
    new Cesium.Cartesian2(
      viewer.canvas.clientWidth / 2,
      viewer.canvas.clientHeight / 2
    ),
    viewer.scene.globe.ellipsoid
  )
  
  if (position) {
    const cartographic = Cesium.Cartographic.fromCartesian(position)
    viewer.entities.add({
      name: '立方体',
      position: Cesium.Cartesian3.fromRadians(
        cartographic.longitude,
        cartographic.latitude,
        cartographic.height + 50000
      ),
      box: {
        dimensions: new Cesium.Cartesian3(100000.0, 100000.0, 100000.0),
        material: Cesium.Color.RED.withAlpha(0.5),
        outline: true,
        outlineColor: Cesium.Color.BLACK
      }
    })
  }
}

// 添加圆柱
function addCylinder() {
  const position = viewer.camera.pickEllipsoid(
    new Cesium.Cartesian2(
      viewer.canvas.clientWidth / 2,
      viewer.canvas.clientHeight / 2
    ),
    viewer.scene.globe.ellipsoid
  )
  
  if (position) {
    const cartographic = Cesium.Cartographic.fromCartesian(position)
    viewer.entities.add({
      name: '圆柱',
      position: Cesium.Cartesian3.fromRadians(
        cartographic.longitude,
        cartographic.latitude,
        cartographic.height + 50000
      ),
      cylinder: {
        length: 100000.0,
        topRadius: 50000.0,
        bottomRadius: 50000.0,
        material: Cesium.Color.BLUE.withAlpha(0.5),
        outline: true,
        outlineColor: Cesium.Color.BLACK
      }
    })
  }
}

// 添加点标注
function addPoint() {
  const position = viewer.camera.pickEllipsoid(
    new Cesium.Cartesian2(
      viewer.canvas.clientWidth / 2,
      viewer.canvas.clientHeight / 2
    ),
    viewer.scene.globe.ellipsoid
  )
  
  if (position) {
    const cartographic = Cesium.Cartographic.fromCartesian(position)
    const lon = Cesium.Math.toDegrees(cartographic.longitude).toFixed(2)
    const lat = Cesium.Math.toDegrees(cartographic.latitude).toFixed(2)
    
    viewer.entities.add({
      name: `标注点 (${lon}, ${lat})`,
      position: position,
      point: {
        pixelSize: 15,
        color: Cesium.Color.YELLOW,
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2
      },
      label: {
        text: `(${lon}°, ${lat}°)`,
        font: '14pt sans-serif',
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        outlineWidth: 2,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
        pixelOffset: new Cesium.Cartesian2(0, -15)
      }
    })
  }
}

// 清除所有实体（除了初始地标）
function clearEntities() {
  const entities = viewer.entities.values
  const toRemove = []
  
  for (let i = 0; i < entities.length; i++) {
    const name = entities[i].name
    if (name !== '北京天安门' && name !== '上海东方明珠' && name !== '成都') {
      toRemove.push(entities[i])
    }
  }
  
  toRemove.forEach(entity => {
    viewer.entities.remove(entity)
  })
}
</script>

<style scoped>
.cesium-container {
  width: 100%;
  height: 100vh;
  position: relative;
}

#cesiumContainer {
  width: 100%;
  height: 100%;
}

.control-panel {
  position: absolute;
  top: 0;
  right: 0;
  background: rgba(42, 42, 42, 0.95);
  color: white;
  padding: 15px;
  border-radius: 0 0 0 8px;
  width: 280px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  z-index: 1000;
}

.control-panel h3 {
  margin: 0 0 15px 0;
  font-size: 18px;
  border-bottom: 2px solid #4CAF50;
  padding-bottom: 8px;
}

.control-section {
  margin-bottom: 20px;
}

.control-section h4 {
  margin: 0 0 10px 0;
  font-size: 14px;
  color: #4CAF50;
}

.control-panel button {
  display: block;
  width: 100%;
  margin: 5px 0;
  padding: 8px 12px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: background 0.3s;
}

.control-panel button:hover {
  background: #45a049;
}

.control-panel select {
  width: 100%;
  padding: 8px;
  margin: 5px 0;
  border-radius: 4px;
  border: 1px solid #555;
  background: #333;
  color: white;
  font-size: 13px;
}

.control-panel label {
  display: flex;
  align-items: center;
  cursor: pointer;
  font-size: 13px;
}

.control-panel input[type="checkbox"] {
  margin-right: 8px;
  cursor: pointer;
}

.info-text {
  background: rgba(0, 0, 0, 0.3);
  padding: 10px;
  border-radius: 4px;
  font-size: 12px;
}

.info-text p {
  margin: 5px 0;
  font-family: monospace;
}

/* 滚动条样式 */
.control-panel::-webkit-scrollbar {
  width: 8px;
}

.control-panel::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 4px;
}

.control-panel::-webkit-scrollbar-thumb {
  background: #4CAF50;
  border-radius: 4px;
}

.control-panel::-webkit-scrollbar-thumb:hover {
  background: #45a049;
}
</style>
