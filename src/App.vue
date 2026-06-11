<template>
  <div class="app-wrapper">
    <!-- 顶部导航栏 -->
    <header class="top-header">
      <div class="header-inner">
        <!-- 左上角：天气信息 -->
        <div class="header-left-abs">
          <div class="location-weather">
            <div class="city-group">
              <el-icon :size="16" color="#00d4ff"><Location /></el-icon>
              <span class="location-text">广州</span>
            </div>
            <span class="weather-text">多云</span>
            <span class="weather-text">27 - 32 °C</span>
          </div>
        </div>
        <!-- 右上角：用户信息 -->
        <div class="header-right-abs">
          <span class="header-link" @click="handleSystemManage">系统管理</span>
          <el-dropdown trigger="click">
            <span class="user-info">
              <el-icon :size="18"><UserFilled /></el-icon>
              <span>Admin</span>
              <el-icon :size="12"><ArrowDown /></el-icon>
            </span>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item>个人设置</el-dropdown-item>
                <el-dropdown-item @click="handleLogout">退出登录</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          <span class="header-link" @click="handleLogout">退出登录</span>
        </div>
        <!-- 标题行 - 居中 -->
        <div class="title-row">
          <div class="logo-area">
            <span class="system-title">海洋要素可视化系统</span>
          </div>
        </div>
        <!-- 导航行 - 居中 -->
        <nav class="nav-row">
          <span
            v-for="item in navItems"
            :key="item.key"
            class="nav-item"
            :class="{ active: activeNav === item.key }"
            @click="activeNav = item.key"
          >
            {{ item.label }}
          </span>
        </nav>
      </div>
    </header>

    <!-- 主体内容 -->
    <div class="main-body">
      <!-- 左侧专题数据面板 -->
      <aside class="left-panel">
        <div class="data-type-list">
          <div
            v-for="item in dataTypes"
            :key="item.value"
            class="data-type-btn"
            :class="{ active: activeDataType === item.value }"
            @click="changeDataType(item.value)"
          >
            <span class="btn-text">专题数据-{{ item.label }}</span>
          </div>
        </div>
      </aside>

      <!-- 中央地图区域 -->
      <main class="center-area">
        <!-- 地图可视化 - Cesium 3D 地球 -->
        <div ref="mapWrapper" class="map-wrapper">
          <div ref="cesiumContainer" class="map-chart"></div>

          <!-- 地图工具栏 -->
          <div class="float-panel map-toolbar">
            <div class="toolbar-item" :class="{ active: isToolbarCollapsed }" @click="handleCurtain">
              <el-icon :size="22"><Operation /></el-icon>
              <span>卷帘</span>
            </div>
            <div v-show="!isToolbarCollapsed" class="toolbar-item" @click="handleZoomIn">
              <el-icon :size="22"><ZoomIn /></el-icon>
              <span>放大</span>
            </div>
            <div v-show="!isToolbarCollapsed" class="toolbar-item" @click="handleZoomOut">
              <el-icon :size="22"><ZoomOut /></el-icon>
              <span>缩小</span>
            </div>
            <div v-show="!isToolbarCollapsed" class="toolbar-item" @click="handleReset">
              <el-icon :size="22"><RefreshRight /></el-icon>
              <span>复位</span>
            </div>
            <div v-show="!isToolbarCollapsed" class="toolbar-item" @click="handleLegend">
              <el-icon :size="22"><List /></el-icon>
              <span>图例</span>
            </div>
            <div v-show="!isToolbarCollapsed" class="toolbar-item" @click="handleFullscreen">
              <el-icon :size="22"><FullScreen /></el-icon>
              <span>全屏</span>
            </div>
            <div v-show="!isToolbarCollapsed" class="toolbar-item" @click="handleSaveTheme">
              <el-icon :size="22"><FolderChecked /></el-icon>
              <span>保存专题</span>
            </div>
          </div>

          <!-- 左侧面板重开按钮 -->
          <div v-show="!isLeftPanelVisible && !isToolbarCollapsed" class="left-panel-reopen" @click="isLeftPanelVisible = true">
            <el-icon :size="18"><Setting /></el-icon>
          </div>

          <!-- 左侧控制面板 -->
          <div v-show="isLeftPanelVisible" class="float-panel left-float">
            <!-- 年份范围选择 -->
            <div class="year-range-row">
              <el-date-picker
                v-model="yearRange"
                type="yearrange"
                range-separator="—"
                start-placeholder="开始年份"
                end-placeholder="结束年份"
                size="small"
                class="year-range-picker"
                format="YYYY"
                value-format="YYYY"
              />
            </div>
            <!-- 当前数据图层 -->
            <div class="layer-select-row">
              <div class="data-type-display">
                <span class="data-type-label">EN4温盐格点数据(C14)</span>
              </div>
            </div>
            <div class="float-panel-header">
              <span class="panel-label var-label">变量</span>
              <div class="header-right-group">
                <span class="panel-label">廊线点</span>
                <el-switch v-model="showLayer" size="small" active-color="#00d4ff" />
              </div>
            </div>
            <!-- 变量字段按钮 -->
            <div class="variable-fields">
              <div class="variable-btn">
                <span>temperature</span>
              </div>
              <div class="variable-btn">
                <span>salinity</span>
              </div>
            </div>
            <div class="float-panel-body">
              <!-- 色带选择 -->
              <div class="section-title">色带</div>
              <div class="colorbar-select-wrapper">
                <div
                  class="colorbar-overlay"
                  :style="{ background: `linear-gradient(to right, ${currentColorScheme?.colors.join(',') ?? ''})` }"
                ></div>
                <el-select
                  v-model="selectedScheme"
                  class="colorbar-select"
                  popper-class="colorbar-popper"
                  placeholder="选择色带"
                  @change="onSchemeChange"
                >
                  <el-option
                    v-for="(scheme, idx) in colorSchemes"
                    :key="idx"
                    :label="' '"
                    :value="idx"
                  >
                    <div class="select-option-colorbar" :style="{ background: `linear-gradient(to right, ${scheme.colors.join(',')})` }"></div>
                  </el-option>
                </el-select>
              </div>

              <!-- 色带数值标注 -->
              <div class="colorbar-value-labels">
                <span
                  v-for="(tick, idx) in colorbarTicks"
                  :key="idx"
                  class="value-tick"
                >{{ tick }}</span>
              </div>

              <!-- 色带数值范围 -->
              <div class="range-inputs">
                <div class="range-row">
                  <el-input
                    v-model="colorMin"
                    size="small"
                    class="range-input"
                    placeholder="-5.0"
                    @change="onRangeChange"
                  />
                  <span class="range-label">最小值</span>
                </div>
                <div class="range-row">
                  <span class="range-label">最大值</span>
                  <el-input
                    v-model="colorMax"
                    size="small"
                    class="range-input"
                    placeholder="35.0"
                    @change="onRangeChange"
                  />
                </div>
              </div>

              <el-button type="primary" size="small" class="apply-btn" @click="applySettings">
                显示
              </el-button>
            </div>
          </div>

          <!-- 右上角经纬度 -->
          <div class="coord-text">
            <span>经度 {{ coordLng }}</span>
            <span>纬度 {{ coordLat }}</span>
          </div>

          <!-- 图例面板 -->
          <div v-show="showLegendPanel" class="float-panel legend-panel">
            <div class="legend-items">
              <div class="legend-item">
                <div class="legend-shape legend-rect-red"></div>
                <span class="legend-name">图例1</span>
              </div>
              <div class="legend-item">
                <div class="legend-shape legend-rect-green"></div>
                <span class="legend-name">图例2</span>
              </div>
              <div class="legend-item">
                <div class="legend-shape legend-circle-blue"></div>
                <span class="legend-name">图例3</span>
              </div>
            </div>
          </div>

          <!-- 折线图 -->
          <div v-show="showChartPanel" ref="chartContainer" class="chart-float"></div>
          <!-- 图表显示/隐藏按钮 -->
          <div class="chart-toggle-btn" @click="showChartPanel = !showChartPanel">
            <el-icon v-if="showChartPanel" :size="15"><Close /></el-icon>
            <el-icon v-else :size="15"><TrendCharts /></el-icon>
          </div>

          <!-- 局部放大图 -->
          <div ref="minimapContainer" class="minimap-wrapper"></div>
        </div>
      </main>


    </div>

  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed, nextTick, watch } from 'vue'
import * as mars3d from 'mars3d'
import * as Cesium from 'mars3d-cesium'
import 'mars3d-cesium/Build/Cesium/Widgets/widgets.css'
import * as echarts from 'echarts'
import {
  Location, UserFilled, ArrowDown,
  Operation, ZoomIn, ZoomOut, RefreshRight, List, FullScreen, FolderChecked,
  Close, TrendCharts, Setting,

} from '@element-plus/icons-vue'

// ========== 导航 ==========
const navItems = [
  { key: 'home', label: '首页' },
  { key: 'ai', label: 'AI模型' },
  { key: 'satellite', label: '卫星数据' },
  { key: 'observation', label: '观测及其重构产品' },
  { key: 'reanalysis', label: '模式再分析数据' },
]
const activeNav = ref('home')

// ========== 系统操作 ==========
const handleSystemManage = () => {
  console.log('打开系统管理')
}

const handleLogout = () => {
  console.log('退出登录')
}

// ========== 地图工具栏 ==========
const handleCurtain = () => {
  isToolbarCollapsed.value = !isToolbarCollapsed.value
}
const handleZoomIn = () => {
  const camera = map?.camera
  if (!camera) return
  camera.zoomIn(500000)
}
const handleZoomOut = () => {
  const camera = map?.camera
  if (!camera) return
  camera.zoomOut(500000)
}
const handleReset = () => {
  map?.flyHome()
}
const handleLegend = () => {
  showLegendPanel.value = !showLegendPanel.value
}
const handleFullscreen = () => {
  if (!mapWrapper.value) return
  if (document.fullscreenElement) {
    document.exitFullscreen()
  } else {
    mapWrapper.value.requestFullscreen()
  }
}
const handleSaveTheme = () => {
  if (!map) return
  const now = new Date()
  const dateStr = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`
  map.expImage({ download: true, name: `专题图_${dateStr}` })
}

// ========== 折线图 ==========
const generateChartData = () => {
  const startYear = parseInt(yearRange.value[0])
  const endYear = parseInt(yearRange.value[1])
  const span = endYear - startYear
  const data: [string, number][] = []

  if (span > 3) {
    for (let year = startYear; year <= endYear; year++) {
      const base = Math.sin((year - 2010) * 0.65) * 0.12
      const noise = (Math.random() - 0.5) * 0.06
      const value = -0.05 + base + noise
      data.push([`${year}`, parseFloat(value.toFixed(3))])
    }
  } else {
    for (let year = startYear; year <= endYear; year++) {
      const endMonth = year === endYear ? 12 : 12
      for (let month = 1; month <= endMonth; month++) {
        const t = (year - startYear) + (month - 1) / 12
        const base = Math.sin(t * 1.8) * 0.1
        const noise = (Math.random() - 0.5) * 0.04
        const value = -0.05 + base + noise
        data.push([`${year}-${String(month).padStart(2, '0')}`, parseFloat(value.toFixed(3))])
      }
    }
  }

  return data
}

const getChartOption = (): echarts.EChartsOption => {
  const data = generateChartData()
  return {
    backgroundColor: 'rgba(0, 0, 0, 0.55)',
    title: {
      text: '洋流Uo随时间变化',
      left: 10,
      top: 8,
      textStyle: { color: '#e0e0e0', fontSize: 16, fontWeight: 500 },
    },
    graphic: [{
      type: 'group',
      left: 12,
      right: 12,
      top: 30,
      children: [{
        type: 'rect',
        shape: { x: 0, y: 0, width: 9999, height: 0.5 },
        style: { fill: '#555555' },
        silent: true,
      }],
    }],
    animation: false,
    grid: { top: 40, right: 20, bottom: 32, left: 48 },
    tooltip: { show: false },
    xAxis: {
      type: 'category',
      data: data.map(d => d[0]),
      axisLabel: { color: '#888888', fontSize: 10, rotate: data.length > 12 ? 45 : 0 },
      axisLine: { lineStyle: { color: '#444444' } },
      axisTick: { show: false },
    },
    yAxis: {
      type: 'value',
      min: -0.3,
      max: 0.2,
      interval: 0.1,
      splitLine: { show: true, lineStyle: { color: '#666666', type: 'solid', width: 1 } },
      axisLabel: { color: '#888888', fontSize: 10 },
      axisLine: { show: false },
      axisTick: { show: false },
    },
    series: [{
      type: 'line',
      data: data.map(d => d[1]),
      smooth: false,
      symbol: 'circle',
      symbolSize: 5,
      lineStyle: { color: '#4a90d9', width: 1.5 },
      itemStyle: {
        color: '#4a90d9',
        borderWidth: 0,
      },
    }],
  }
}

const initChart = () => {
  if (!chartContainer.value) return
  if (chartInstance) {
    chartInstance.dispose()
    chartInstance = null
  }
  chartInstance = echarts.init(chartContainer.value)
  chartInstance.setOption(getChartOption())
}

const initMinimap = () => {
  if (!minimapContainer.value || !map) return
  minimap = new mars3d.Map(minimapContainer.value, {
    scene: {
      center: { lat: 36, lng: 112, alt: 200000, heading: 0, pitch: -90 },
      sceneMode: 3,
      fxaa: true,
      globe: { baseColor: '#0d2b50', enableLighting: true },
      cameraController: { zoomFactor: 2, minimumZoomDistance: 100, maximumZoomDistance: 1000000 },
    },
    control: {
      baseLayerPicker: false, homeButton: false, sceneModePicker: false,
      navigationHelpButton: false, fullscreenButton: false, geocoder: false,
      infoBox: false, animation: false, timeline: false,
      attributionControl: false,
    },
    basemaps: [
      {
        name: '天地图影像',
        type: 'group',
        show: true,
        layers: [
          { name: '底图', type: 'tdt', layer: 'img_d' },
          { name: '注记', type: 'tdt', layer: 'img_z' },
        ],
      },
    ],
  })
  // 隐藏版权信息
  const credit = minimap.viewer.cesiumWidget.creditContainer as HTMLElement
  if (credit) credit.style.display = 'none'

  // 同步主地图中心到缩略图（放大视角）
  map.on(mars3d.EventType.cameraChanged, () => {
    if (!map || !minimap) return
    const center = map.getCenter()
    minimap.setCameraView({ lat: center.lat, lng: center.lng, alt: 200000, heading: 0, pitch: -90 })
  })
}

const updateChart = () => {
  if (!chartInstance) return
  chartInstance.setOption(getChartOption())
}

// ========== 专题数据 ==========
// ========== 专题数据 ==========
const dataTypes = [
  { value: 'temperature', label: '温度', icon: 'Sunny' },
  { value: 'salinity', label: '盐度', icon: 'Coin' },
  { value: 'waterLevel', label: '水位', icon: 'Histogram' },
  { value: 'meteorology', label: '气象', icon: 'WindPower' },
  { value: 'current', label: '海流', icon: 'Ship' },
  { value: 'ph', label: 'PH值', icon: 'Opportunity' },
  { value: 'biology', label: '生物', icon: 'Apple' },
  { value: 'disaster', label: '灾害', icon: 'WarnTriangleNotFilled' },
]
const activeDataType = ref('salinity')

// 各专题数据对应的色带与范围配置
const dataTypeConfigs: Record<string, { unit: string, min: number, max: number, schemeIndex: number }> = {
  temperature:  { unit: '°C',       min: -5,  max: 40,  schemeIndex: 4 },
  salinity:     { unit: 'PSU',      min: 30,  max: 38,  schemeIndex: 1 },
  waterLevel:   { unit: 'm',        min: -2,  max: 10,  schemeIndex: 0 },
  meteorology:  { unit: 'hPa',      min: 980, max: 1050, schemeIndex: 3 },
  current:      { unit: 'm/s',      min: 0,   max: 2.5, schemeIndex: 0 },
  ph:           { unit: '',         min: 6.5, max: 9.0, schemeIndex: 2 },
  biology:      { unit: 'mg/m³',    min: 0,   max: 30,  schemeIndex: 2 },
  disaster:     { unit: '级',       min: 0,   max: 12,  schemeIndex: 4 },
}

const changeDataType = (type: string) => {
  activeDataType.value = type
  const config = dataTypeConfigs[type]
  if (config) {
    selectedScheme.value = config.schemeIndex
    colorMin.value = String(config.min)
    colorMax.value = String(config.max)
  }
  updateMapVisualization()
}

// ========== Cesium 地球 ==========
const cesiumContainer = ref<HTMLDivElement | null>(null)
const mapWrapper = ref<HTMLDivElement | null>(null)
let map: mars3d.Map | null = null
const isToolbarCollapsed = ref(false)
const isLeftPanelVisible = ref(true)
const showLegendPanel = ref(true)
const showChartPanel = ref(true)
const chartContainer = ref<HTMLDivElement | null>(null)
const minimapContainer = ref<HTMLDivElement | null>(null)
let chartInstance: echarts.ECharts | null = null
let minimap: mars3d.Map | null = null
const yearRange = ref<string[]>(['2020', '2024'])
const showLayer = ref(true)

const coordLng = ref("106°27′30.00″")
const coordLat = ref("27°30′23.12″")

const toDMS = (deg: number): string => {
  const d = Math.floor(Math.abs(deg))
  const m = Math.floor((Math.abs(deg) - d) * 60)
  const s = ((Math.abs(deg) - d - m / 60) * 3600).toFixed(2)
  return `${d}°${String(m).padStart(2, '0')}′${s}″`
}

// 色带方案
const colorSchemes = [
  { name: '蓝-青-黄-红', colors: ['#000080', '#0044cc', '#0088ff', '#00ccff', '#00ff88', '#ffff00', '#ff8800', '#ff0000'] },
  { name: '深蓝-白-红', colors: ['#0000ff', '#3366ff', '#66aaff', '#ffffff', '#ffaa66', '#ff6633', '#ff0000'] },
  { name: '蓝-绿-黄', colors: ['#0022aa', '#0055dd', '#0088ff', '#00cc88', '#44ee44', '#ccff00', '#ffee00'] },
  { name: '紫-蓝-青', colors: ['#2d004b', '#54278f', '#756bb1', '#9e9ac8', '#bdd7e7', '#6baed6', '#2171b5'] },
  { name: '热力渐变', colors: ['#000000', '#330000', '#880000', '#cc4400', '#ff8800', '#ffcc00', '#ffff00', '#ffffff'] },
  { name: '海底地形', colors: ['#0a3d0a', '#1a6b1a', '#2d882d', '#55aa55', '#88cc88', '#cceecc', '#ffffff'] },
]
const selectedScheme = ref(0)
const colorMin = ref('-5.0')
const colorMax = ref('35.0')

const currentColorScheme = computed(() => colorSchemes[selectedScheme.value])

const colorbarTicks = computed(() => {
  const min = parseFloat(colorMin.value) || 0
  const max = parseFloat(colorMax.value) || 1
  const count = 6
  const ticks: string[] = []
  for (let i = 0; i <= count; i++) {
    const val = min + ((max - min) * i) / count
    ticks.push(Number.isInteger(val) ? val.toString() : val.toFixed(1))
  }
  return ticks
})

const onSchemeChange = () => {
  console.log('切换色带方案:', colorSchemes[selectedScheme.value]?.name)
}

const onRangeChange = () => {
  // 色带范围变化时触发
  console.log('色带范围:', colorMin.value, '~', colorMax.value)
}

const initMars3dMap = () => {
  if (!cesiumContainer.value) return

  map = new mars3d.Map(cesiumContainer.value, {
    scene: {
      center: { lat: 36, lng: 112, alt: 8000000, heading: 0, pitch: -90 },
      sceneMode: 3,
      fxaa: true,
      globe: {
        baseColor: '#0d2b50',
        showGroundAtmosphere: true,
        enableLighting: true,
      },
      cameraController: {
        zoomFactor: 3.0,
        minimumZoomDistance: 1,
        maximumZoomDistance: 50000000,
      },
    },
    control: {
      baseLayerPicker: false,
      homeButton: false,
      sceneModePicker: false,
      navigationHelpButton: false,
      fullscreenButton: false,
      geocoder: false,
      infoBox: false,
      animation: false,
      timeline: false,
    },
    basemaps: [
      {
        name: '天地图影像',
        type: 'group',
        show: true,
        layers: [
          { name: '底图', type: 'tdt', layer: 'img_d' },
          { name: '注记', type: 'tdt', layer: 'img_z' },
        ],
      },
    ],
  })

  // 隐藏 Cesium 版权信息
  const creditContainer = map.viewer.cesiumWidget.creditContainer as HTMLElement
  if (creditContainer) creditContainer.style.display = 'none'

  // 鼠标移动经纬度监听（Mars3D 传入 event.cartesian，不是 cartographic）
  map.on(mars3d.EventType.mouseMove, (event: any) => {
    if (event.cartesian) {
      const cartographic = Cesium.Cartographic.fromCartesian(event.cartesian)
      if (cartographic) {
        const lng = Cesium.Math.toDegrees(cartographic.longitude)
        const lat = Cesium.Math.toDegrees(cartographic.latitude)
        coordLng.value = toDMS(lng)
        coordLat.value = toDMS(lat)
      }
    }
  })
}
const updateMapVisualization = () => {
  // Cesium 场景已自动渲染，切换数据类型时可在此添加图层逻辑
  console.log('切换到数据类型:', activeDataType.value)
}

const applySettings = () => {
  updateMapVisualization()
  isLeftPanelVisible.value = false
}

// ========== 折线图 watch ==========
watch(yearRange, () => {
  updateChart()
}, { deep: true })

watch(showChartPanel, (visible) => {
  if (visible) {
    nextTick(() => {
      initChart()
    })
  }
})

let chartResizeObserver: ResizeObserver | null = null

const handleWindowResize = () => {
  chartInstance?.resize()
}

// ========== 生命周期 ==========
onMounted(() => {
  nextTick(() => {
    initMars3dMap()
    initChart()
    initMinimap()
    // 监听折线图容器大小变化
    if (chartContainer.value) {
      chartResizeObserver = new ResizeObserver(() => {
        chartInstance?.resize()
      })
      chartResizeObserver.observe(chartContainer.value)
    }
  })
  window.addEventListener('resize', handleWindowResize)
})

onBeforeUnmount(() => {
  chartResizeObserver?.disconnect()
  minimap?.destroy()
  map?.destroy()
  chartInstance?.dispose()
  window.removeEventListener('resize', handleWindowResize)
})
</script>

<style lang="scss">
// ===== 全局重置 =====
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  height: 100%;
  overflow: hidden;
  font-family: 'Microsoft YaHei', 'PingFang SC', 'Helvetica Neue', sans-serif;
  background: #020b1a;
  color: #c8e0f0;
}

// ===== 应用容器 =====
.app-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: radial-gradient(ellipse at 30% 40%, #0a1e3d 0%, #020b1a 70%);
}

// ===== 顶部导航 =====
.top-header {
  flex-shrink: 0;
  background: linear-gradient(180deg, rgba(10,30,60,0.95) 0%, rgba(5,20,45,0.9) 100%);
  border-bottom: 1px solid rgba(0,180,255,0.2);
  backdrop-filter: blur(10px);
  z-index: 100;
  position: relative;

  .header-inner {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 12px 24px 8px;
  }

  .header-left-abs {
    position: absolute;
    left: 24px;
    top: 16px;
    display: flex;
    align-items: center;

    .location-weather {
      display: flex;
      align-items: center;
      gap: 14px;
      font-size: 13px;
      color: #7eb8da;

      .city-group {
        display: flex;
        align-items: center;
        gap: 3px;
      }

      .location-text {
        color: #00d4ff;
        font-weight: 500;
      }
    }
  }

  .header-right-abs {
    position: absolute;
    right: 24px;
    top: 16px;
    display: flex;
    align-items: center;
    gap: 20px;

    .header-link {
      font-size: 13px;
      color: #7eb8da;
      cursor: pointer;
      transition: color 0.2s;

      &:hover {
        color: #00d4ff;
      }
    }

    .user-info {
      display: flex;
      align-items: center;
      gap: 6px;
      color: #7eb8da;
      cursor: pointer;
      font-size: 13px;

      &:hover {
        color: #00d4ff;
      }
    }
  }

  .title-row {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 4px 0;
  }

  .logo-area {
    display: flex;
    align-items: center;
    gap: 12px;

    .system-title {
      font-size: 34px;
      font-weight: 700;
      color: #ffffff;
      letter-spacing: 4px;
      text-shadow: 0 0 24px rgba(0,180,255,0.5);
    }
  }

  .nav-row {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
    padding: 8px 0 8px;

    .nav-item {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      height: 36px;
      padding: 0 18px;
      border-radius: 6px;
      font-size: 14px;
      line-height: 1;
      color: #ffffff;
      cursor: pointer;
      transition: all 0.25s;
      white-space: nowrap;

      &:hover {
        color: #00d4ff;
        background: rgba(0,180,255,0.1);
      }

      &.active {
        color: #00d4ff;
        background: rgba(0,180,255,0.15);
        box-shadow: 0 0 12px rgba(0,180,255,0.15);
      }
    }
  }
}

// ===== 主体布局 =====
.main-body {
  display: flex;
  flex: 1;
  overflow: hidden;
}

// ===== 左侧面板 =====
.left-panel {
  width: clamp(170px, 12vw, 250px);
  flex-shrink: 0;
  background: transparent;
  display: flex;
  flex-direction: column;
  padding: clamp(6px, 0.8vw, 14px) 0;

  .panel-title-wrapper {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 0 16px 12px;
    border-bottom: 1px solid rgba(0,150,255,0.15);
    margin-bottom: 8px;

    .panel-title {
      font-size: 16px;
      font-weight: 600;
      color: #00d4ff;
    }
  }

  .data-type-list {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 22px;
    padding: 0 12px;
    overflow-y: auto;
  }

  .data-type-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.25s;
    background: #0b2a55;
    border: 3.5px solid #3a9ad8;

    .btn-text {
      font-size: 13px;
      color: #ffffff;
      white-space: nowrap;
      font-weight: 500;
      letter-spacing: 0.5px;
    }

    &:hover {
      background: #0e3468;
      border-color: #5ab8e8;
      box-shadow: 0 0 10px rgba(58,154,216,0.25);

      .btn-text {
        color: #ffffff;
      }
    }

    &.active {
      background: #103c78;
      border-color: #00d4ff;
      box-shadow: 0 0 14px rgba(0,212,255,0.3);

      .btn-text {
        color: #00d4ff;
        font-weight: 600;
      }
    }
  }
}

// ===== 中央区域 =====
.center-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding: 10px 10px 0 10px;
}

.map-wrapper {
  flex: 1;
  position: relative;
  border: 1px solid rgba(0,150,255,0.15);
  border-radius: 6px;
  overflow: hidden;
  background: #020b1a;
}

.map-chart {
  width: 100%;
  height: 100%;

  .cesium-viewer {
    width: 100%;
    height: 100%;
  }

  .cesium-widget {
    width: 100%;
    height: 100%;
  }
}

.float-panel {
  position: absolute;
  background: rgba(128, 128, 128, 0.5);
  border: 1px solid rgba(160, 160, 160, 0.3);
  border-radius: 8px;
  backdrop-filter: blur(12px);
  z-index: 10;
}

.map-toolbar {
  top: clamp(8px, 1vw, 16px);
  left: clamp(8px, 1vw, 16px);
  display: flex;
  flex-direction: row;
  gap: clamp(2px, 0.3vw, 4px);
  padding: clamp(4px, 0.5vw, 8px) clamp(6px, 0.7vw, 12px);
  background: rgba(255, 255, 255, 0.95);
  border: 1px solid rgba(100, 180, 220, 0.7);
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
  backdrop-filter: none;

  .toolbar-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: clamp(1px, 0.15vw, 2px);
    padding: clamp(3px, 0.4vw, 6px) clamp(5px, 0.6vw, 10px);
    cursor: pointer;
    border-radius: 4px;
    color: #1a3a5c;
    transition: background 0.2s;

    &:hover {
      background: rgba(0, 150, 255, 0.1);
    }

    &.active {
      background: rgba(0, 180, 255, 0.15);
      box-shadow: 0 0 0 1px rgba(0, 180, 255, 0.4);
      color: #005599;
    }

    span {
      font-size: clamp(9px, 0.65vw, 11px);
      white-space: nowrap;
    }
  }
}

.left-float {
  top: clamp(60px, 5vw, 100px);
  left: clamp(8px, 1vw, 16px);
  width: clamp(260px, 19vw, 380px);
  padding: clamp(12px, 1.4vw, 24px) clamp(10px, 1vw, 18px);
  max-height: calc(100% - 32px);
  overflow-y: auto;

  .year-range-row {
    margin-bottom: 10px;

    .year-range-picker {
      width: 100%;
      --el-input-bg-color: rgba(5, 18, 42, 0.95) !important;
      background: rgba(5, 18, 42, 0.95) !important;
      border-color: rgba(0, 150, 255, 0.3) !important;
      height: 36px !important;
    }
  }

  .layer-select-row {
    margin-bottom: 12px;

    .layer-select-inner {
      width: 100%;

      .el-select__wrapper {
        --el-input-bg-color: rgba(5, 18, 42, 0.95) !important;
        background: rgba(5, 18, 42, 0.95) !important;
        border-color: rgba(0, 150, 255, 0.3) !important;
        box-shadow: none !important;
        height: 36px !important;
      }

      .el-select__placeholder {
        color: #7eb8da;
      }

      .el-select__selected-item {
        color: #c8e0f0;
      }
    }

    .data-type-display {
      display: flex;
      align-items: center;
      justify-content: flex-start;
      gap: 4px;
      padding: 6px 10px;
      background: rgba(5, 18, 42, 0.95);
      border: 1px solid rgba(0, 150, 255, 0.35);
      border-radius: 4px;
      height: 36px;
      width: 85%;

      .data-type-label {
        color: #e0f0ff;
        font-size: 13px;
        font-weight: 600;
      }

      .data-type-unit {
        color: #7eb8da;
        font-size: 12px;
      }
    }
  }

  .float-panel-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    margin-bottom: 10px;

    .panel-label {
      font-size: 14px;
      font-weight: 500;
      color: #00d4ff;
    }

    .var-label {
      color: #ffffff;
    }

    .header-right-group {
      display: flex;
      align-items: center;
      gap: 6px;
    }
  }

  .variable-fields {
    display: flex;
    gap: 8px;
    margin-bottom: 10px;

    .variable-btn {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 4px 12px;
      background: rgba(128, 128, 128, 0.5);
      border: 1px solid rgba(160, 160, 160, 0.3);
      border-radius: 4px;
      cursor: pointer;
      transition: border-color 0.2s;

      &:hover {
        border-color: #00d4ff;
      }

      span {
        font-size: 13px;
        color: #c8e0f0;
        font-family: 'Consolas', 'Courier New', monospace;
      }
    }
  }

  .float-panel-body {
    display: flex;
    flex-direction: column;
    .section-title {
      font-size: 14px;
      font-weight: 500;
      color: #ffffff;
      margin-bottom: 6px;
    }

    .colorbar-select-wrapper {
      position: relative;
      width: 100%;
      margin-bottom: 10px;

      .colorbar-overlay {
        position: absolute;
        top: 6px;
        left: 10px;
        right: 30px;
        bottom: 6px;
        border-radius: 3px;
        pointer-events: none;
        z-index: 1;
      }
    }

    .colorbar-select {
      width: 100%;

      .el-input__wrapper,
      .el-select__wrapper {
        --el-input-bg-color: rgba(5, 18, 42, 0.95) !important;
        background: rgba(5, 18, 42, 0.95) !important;
        border-color: rgba(0, 150, 255, 0.3) !important;
        box-shadow: none !important;
        height: 36px;
      }

      .el-input__inner,
      .el-select__placeholder,
      .el-select__selected-item {
        caret-color: transparent !important;
        color: transparent !important;
        user-select: none !important;
      }

      .el-input__inner::placeholder {
        color: transparent !important;
        opacity: 0 !important;
      }

      .el-input__suffix,
      .el-select__suffix {
        z-index: 2;
      }

      .el-input__suffix-inner,
      .el-select__caret {
        color: rgba(200, 224, 240, 0.7);
      }
    }

    .colorbar-value-labels {
      display: flex;
      justify-content: space-between;
      margin-bottom: 10px;
      padding-left: 10px;
      padding-right: 30px;

      .value-tick {
        font-size: 11px;
        color: #7eb8da;
        font-family: 'Consolas', 'Courier New', monospace;
        line-height: 1;
      }
    }

    .range-inputs {
      display: flex;
      justify-content: space-between;
      margin-bottom: 8px;

      .range-row {
        display: flex;
        align-items: center;
        gap: 4px;

        .range-label {
          font-size: 11px;
          color: #7eb8da;
          white-space: nowrap;
        }

        .range-input {
          width: 70px;
          flex: none;

          .el-input__wrapper {
            --el-input-bg-color: rgba(5, 18, 42, 0.95) !important;
            padding: 0 8px;
            background: rgba(5, 18, 42, 0.95) !important;
            border-color: rgba(0, 150, 255, 0.3) !important;
            box-shadow: none !important;
          }

          .el-input__inner {
            font-size: 12px;
            color: #c8e0f0;
            text-align: center;
            height: 26px;
          }
        }
      }
    }

    .slider-row {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-bottom: 8px;

      .slider-label {
        font-size: 12px;
        color: #7eb8da;
        white-space: nowrap;
        min-width: 28px;
      }

      .slider-value {
        font-size: 12px;
        color: #00d4ff;
        min-width: 36px;
        text-align: right;
      }
    }

    .apply-btn {
      width: auto;
      margin-top: 6px;
      margin-left: auto;
      background: rgba(5, 18, 42, 0.95);
      border: 1px solid rgba(0, 150, 255, 0.3);
      border-radius: 4px;
      font-size: 13px;

      &:hover {
        background: rgba(5, 30, 60, 0.95);
        border-color: rgba(0, 180, 255, 0.5);
      }
    }
  }
}

.coord-text {
  position: absolute;
  top: clamp(1px, 0.15vw, 3px);
  right: clamp(8px, 1vw, 16px);
  width: 42%;
  display: flex;
  justify-content: center;
  gap: clamp(10px, 1.2vw, 20px);
  font-size: clamp(10px, 0.75vw, 13px);
  font-weight: 600;
  color: #ffffff;
  font-family: 'Consolas', 'Courier New', monospace;
  z-index: 10;
  text-shadow: 0 0 4px rgba(0, 0, 0, 0.9), 0 0 8px rgba(0, 0, 0, 0.6);
}

// ===== Element Plus 暗色覆写 =====
.el-slider__bar {
  background: #00d4ff !important;
}
.el-slider__button {
  border-color: #00d4ff !important;
}
// Element Plus 输入框 CSS 变量覆盖
.el-input__wrapper {
  --el-input-bg-color: rgba(5, 18, 42, 0.95) !important;
  background: var(--el-input-bg-color) !important;
  border-color: rgba(0, 150, 255, 0.3) !important;
  box-shadow: none !important;
}
.el-input__inner {
  color: #c8e0f0 !important;
}
.el-select .el-input__wrapper {
  --el-input-bg-color: rgba(5, 18, 42, 0.95) !important;
  background: var(--el-input-bg-color) !important;
}

// Element Plus 全局填充色覆盖
:root {
  --el-fill-color-blank: rgba(5, 18, 42, 0.95);
  --el-bg-color: rgba(5, 18, 42, 0.95);
  --el-bg-color-overlay: rgba(5, 18, 42, 0.97);
}
.el-button--primary {
  --el-button-bg-color: #0055aa;
  --el-button-border-color: #0088cc;
}

// 色带下拉选项样式
.select-option-colorbar {
  width: 100%;
  height: 20px;
  border-radius: 3px;
}

// 色带下拉面板 - 定制间距
.colorbar-popper {
  background: rgba(5, 18, 42, 0.97) !important;
  border: 1px solid rgba(0, 160, 255, 0.25) !important;
  backdrop-filter: blur(12px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5) !important;

  .el-select-dropdown__item {
    padding: 5px 20px 5px 10px !important;
    color: #c8e0f0;
    font-size: 12px;
    background: transparent !important;

    &:hover {
      background: rgba(0, 150, 255, 0.2) !important;
      color: #fff;
    }

    &.selected {
      color: #00d4ff !important;
      background: rgba(0, 180, 255, 0.15) !important;
    }
  }

  .el-popper__arrow::before {
    background: rgba(5, 18, 42, 0.97) !important;
    border-color: rgba(0, 160, 255, 0.25) !important;
  }
}

// 下拉面板暗色主题（全局覆盖所有 el-select 下拉）
.el-select-dropdown,
.el-select-dropdown.is-light {
  background: rgba(5, 18, 42, 0.97) !important;
  border: 1px solid rgba(0, 160, 255, 0.25) !important;
  backdrop-filter: blur(12px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5) !important;

  .el-select-dropdown__item {
    color: #c8e0f0;
    font-size: 12px;
    background: transparent !important;

    &:hover {
      background: rgba(0, 150, 255, 0.2) !important;
      color: #fff;
    }

    &.selected {
      color: #00d4ff !important;
      background: rgba(0, 180, 255, 0.15) !important;
      font-weight: 600;
    }
  }

  .el-popper__arrow::before {
    background: rgba(5, 18, 42, 0.97) !important;
    border-color: rgba(0, 160, 255, 0.25) !important;
  }
}

// 日期选择器面板暗色
.el-picker-panel {
  background: rgba(5, 18, 42, 0.97) !important;
  border: 1px solid rgba(0, 160, 255, 0.25) !important;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5) !important;
  color: #c8e0f0;

  .el-date-picker__header-label,
  .el-year-table td .cell,
  .el-month-table td .cell {
    color: #c8e0f0;
  }

  .el-year-table td.current:not(.disabled) .cell,
  .el-month-table td.current:not(.disabled) .cell {
    color: #00d4ff;
  }

  .el-picker-panel__icon-btn {
    color: #7eb8da;
    &:hover { color: #00d4ff; }
  }

  .el-year-table td.today .cell,
  .el-month-table td.today .cell {
    color: #00d4ff;
  }
}

// ===== 滚动条 =====
::-webkit-scrollbar {
  width: 4px;
}
::-webkit-scrollbar-track {
  background: transparent;
}
::-webkit-scrollbar-thumb {
  background: rgba(0,150,255,0.25);
  border-radius: 2px;
}

// ===== 图例面板 =====
.legend-panel {
  bottom: clamp(8px, 1vw, 16px);
  left: clamp(8px, 1vw, 16px);
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: clamp(8px, 0.7vw, 12px) clamp(8px, 0.8vw, 14px);
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid rgba(130, 200, 235, 0.85);
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
  backdrop-filter: none;

  .legend-items {
    display: flex;
    flex-direction: column;
    gap: clamp(6px, 0.6vw, 10px);
  }

  .legend-item {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: clamp(4px, 0.5vw, 8px);
  }

  .legend-shape {
    width: clamp(20px, 1.6vw, 28px);
    height: clamp(20px, 1.6vw, 28px);
    border: 1px solid rgba(0, 0, 0, 0.15);
  }

  .legend-rect-red {
    background: #e04040;
    border-radius: 3px;
  }

  .legend-rect-green {
    background: #40a040;
    border-radius: 3px;
  }

  .legend-circle-blue {
    background: #4060e0;
    border-radius: 50%;
  }

  .legend-name {
    font-size: clamp(9px, 0.65vw, 11px);
    color: #1a3a5c;
    white-space: nowrap;
    text-align: center;
  }
}

// ===== 全屏适配 =====
.map-wrapper:fullscreen {
  .float-panel {
    z-index: 1000;
  }
  .map-toolbar {
    top: 20px;
    left: 20px;
  }
  .legend-panel {
    bottom: 20px;
    left: 20px;
  }
}

.map-wrapper:-webkit-full-screen {
  .float-panel {
    z-index: 1000;
  }
}

// ===== 局部放大图 =====
.minimap-wrapper {
  position: absolute;
  bottom: clamp(8px, 1vw, 16px);
  right: clamp(8px, 1vw, 16px);
  width: clamp(160px, 13vw, 260px);
  height: clamp(120px, 9.5vw, 190px);
  border: 2px solid rgba(130, 200, 235, 0.85);
  border-radius: 6px;
  overflow: hidden;
  z-index: 10;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

// ===== 左侧面板重开按钮 =====
.left-panel-reopen {
  position: absolute;
  top: clamp(60px, 5vw, 100px);
  left: clamp(8px, 1vw, 16px);
  width: clamp(28px, 2vw, 36px);
  height: clamp(28px, 2vw, 36px);
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(5, 18, 42, 0.9);
  border: 1px solid rgba(0, 150, 255, 0.3);
  border-radius: 6px;
  cursor: pointer;
  color: #00d4ff;
  z-index: 10;

  &:hover {
    background: rgba(5, 30, 60, 0.95);
    border-color: rgba(0, 180, 255, 0.5);
  }
}

// ===== 折线图 =====
.chart-float {
  position: absolute;
  top: clamp(16px, 1.5vw, 26px);
  right: clamp(8px, 1vw, 16px);
  width: 42%;
  height: 28%;
  min-width: clamp(240px, 18vw, 320px);
  min-height: clamp(130px, 10vw, 180px);
  z-index: 10;
  border-radius: 12px;
  overflow: hidden;
}

.chart-toggle-btn {
  position: absolute;
  top: clamp(16px, 1.5vw, 26px);
  right: clamp(8px, 1vw, 16px);
  width: clamp(26px, 2vw, 34px);
  height: clamp(26px, 2vw, 34px);
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(80, 80, 80, 0.8);
  border: 1px solid rgba(180, 180, 180, 0.4);
  border-radius: 4px;
  cursor: pointer;
  color: #cccccc;
  z-index: 11;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.4);

  &:hover {
    color: #ffffff;
    background: rgba(100, 100, 100, 0.9);
    border-color: rgba(200, 200, 200, 0.6);
  }
}

// ===== 响应式 =====
@media (max-width: 1400px) {
  .left-panel { width: 180px; }
  .header-nav .nav-item { padding: 4px 8px; font-size: 12px; }
}
</style>