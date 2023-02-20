<template>
  <div class="page">
    <SeamlessScroll class="the-demo-list">
      <div class="the-demo-list__item">1</div>
      <div class="the-demo-list__item">2</div>
      <div class="the-demo-list__item">3</div>
      <div class="the-demo-list__item">4</div>
      <div class="the-demo-list__item">5</div>
      <div class="the-demo-list__item">6</div>
      <div class="the-demo-list__item">7</div>
      <div class="the-demo-list__item">8</div>
    </SeamlessScroll>
    <div ref="mapEl" class="map"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import maplibreGl from 'maplibre-gl';
import * as turf from '@turf/turf'
import SeamlessScroll from './components/SeamlessScroll.vue';

const mapEl = ref(null)
let map

const initOptions = {
  style: {
    "version": 8,
    "id": "43f36e14-e3f5-43c1-84c0-50a9c80dc5c7",
    "glyphs": "mapbox://fonts/mapbox/{fontstack}/{range}.pbf",
    "sources": {},
    "layers": [
    ]
  },
}

const fetchGeoJSON = async (areaCode) => {
  const res = await fetch(`.temp/merged-geojson/${areaCode}.json`, {
    headers: {
      'Content-Type': 'application/json'
    }
  })

  return res.json()
}

// 自动放大
const fitBounds = (feature, options = {}) => {
  const bboxResult = turf.bbox(feature)
  const [a, b, c, d] = bboxResult
  map.fitBounds([
    [a, b],
    [c, d]
  ], options)
}

const loadArea = async (code) => {
  const geoJSON = await fetchGeoJSON(code)
  map.getSource('bound-source').setData(geoJSON)
  fitBounds(geoJSON)
  // 更新标注
  map.setFilter('area-names', ['==', 'parentCode', Number(code)])
}

const emptyGeoJSON = { "type": "FeatureCollection", "features": [] }
onMounted(async () => {
  map = new maplibreGl.Map({
    container: mapEl.value,
    ...initOptions
  })
  map.on('style.load', async () => {
    map.addSource('bound-source', {
      type: 'geojson',
      data: emptyGeoJSON
    })
    map.addLayer({
      id: 'areas-surface',
      type: 'fill',
      source: 'bound-source',
      layout: {},
      paint: {
        'fill-color': '#0357aa',
        'fill-opacity': 1
      }
    })

    // 高亮图层
    map.addLayer({
      id: 'areas-surface-hight',
      type: 'fill',
      source: 'bound-source',
      layout: {},
      paint: {
        "fill-color": 'orange',
        "fill-opacity": 1
      },
      filter: ['==', 'adcode', '']
    })

    map.addLayer({
      id: 'areas-line',
      type: 'line',
      source: 'bound-source',
      paint: {
        'line-color': 'red',
        'line-width': 2
      }
    })

    // 点击
    map.on('click', 'areas-surface', (e) => {
      const feature = e.features[0]
      const code = feature.properties.adcode
      loadArea(code)
    })

    map.on('mousemove', 'areas-surface', (e) => {
      const feature = e.features[0]
      const code = feature.properties.adcode
      // 悬浮显示高亮图层
      map.setFilter('areas-surface-hight', ['==', 'adcode', Number(code)])
    })
    map.on('mouseout', 'areas-surface', (e) => {
      map.setFilter('areas-surface-hight', ['==', 'adcode', ''])
    })

    loadArea(420000)

    const areaNames = await fetchGeoJSON('area-names')
    map.addSource('names-source', {
      type: 'geojson',
      data: areaNames
    })
    map.addLayer({
      id: 'area-names',
      source: 'names-source',
      type: 'symbol',
      layout: {
        'text-field': '{name}',
        'text-size': 8,
      },
      paint: {
        'text-color': 'black',
        'text-halo-color': 'rgba(0, 0, 0, 0)'
      },
      filter: ['==', 'parentCode', 420000]
    })
  })

})
</script>

<style lang="scss">
.page {
  display: flex;
}
.the-demo-list {
  width: 100px;
  height: 180px;
  &__item {
    height: 32px;
    font-size: 16px;
    line-height: 32px;
  }
}
.map {
  width: 100vw;
  height: 100vh;
}
</style>