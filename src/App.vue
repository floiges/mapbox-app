<template>
  <div ref="mapEl" class="map"></div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import mapboxgl from 'maplibre-gl'
import 'maplibre-gl/dist/maplibre-gl.css'

const mapEl = ref(null)

const MY_KEY = '48114c84fb20020e58dc64e3cffbd76c'
const initOption = {
  style: {
    "version": 8,
    "id": "43f36e14-e3f5-43c1-84c0-50a9c80dc5c7",
    "glyphs": "mapbox://fonts/mapbox/{fontstack}/{range}.pbf",
    "sources": {
      "tdt-vec": {
        "type": "raster",
        "tiles": [`https://t0.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILECOL={x}&TILEROW={y}&TILEMATRIX={z}&tk=${MY_KEY}`],
        "tileSize": 256
      },
      "tdt-cva": {
        "type": "raster",
        "tiles": [`https://t0.tianditu.gov.cn/cia_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=cia&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILECOL={x}&TILEROW={y}&TILEMATRIX={z}&tk=${MY_KEY}`],
        "tileSize": 256
      }
    },
    "layers": [
      {
        "id": "tdt-tiles-layer",
        "type": "raster",
        "source": "tdt-vec",
        // "paint": {
        //   "raster-brightness-max": 0.7, // 最大亮度
        //   "raster-brightness-min": 0.3, // 最小亮度
        //   "raster-hue-rotate": 20, // 色相变换的角度
        //   "raster-saturation": 0.7 // 饱和度
        // }
      },
      {
        "id": "tdt-cva-layer",
        "type": "raster",
        "source": "tdt-cva",
      },
    ]
  },
}

const imgs = {
  zhuren: 'https://pic.zhangshichun.top/pic/20221129-12.png',
  bao: 'https://pic.zhangshichun.top/pic/20221129-10.png',
  nan: 'https://pic.zhangshichun.top/pic/20221129-11.png'
}

const boys = {
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "name": "德育处主任",
        "icon": "zhuren"
      },
      "geometry": {
        "coordinates": [
          114.34495622042738,
          30.51879704948628
        ],
        "type": "Point"
      }
    },
    {
      "type": "Feature",
      "properties": {
        "name": "战场小包",
        "icon": "bao"
      },
      "geometry": {
        "coordinates": [
          114.46248908403493,
          30.52385942598788
        ],
        "type": "Point"
      }
    },
    {
      "type": "Feature",
      "properties": {
        "name": "南方者",
        "icon": "nan"
      },
      "geometry": {
        "coordinates": [
          114.4188340204089,
          30.481906063384173
        ],
        "type": "Point"
      }
    }
  ]
}

// 注册图片的方法
const loadImages = async (map, imgs) => {
  await Promise.all(
    Object.entries(imgs).map(([key, url]) => new Promise(resolve => {
      map.loadImage(url, (error, res) => {
        if (error) throw error
        map.addImage(key, res)
        resolve(res)
      })
    }))
  )
}

onMounted(() => {
  const map = new mapboxgl.Map({
    container: mapEl.value,
    ...initOption
  })
  map.on('style.load', async () => {
    // 先加资源
    await loadImages(map, imgs)

    // 添加位置资源
    map.addSource('boys-source', {
      type: 'geojson',
      data: boys
    })

    // 添加 ICON 图册
    map.addLayer({
      id: 'boys-icon-layer',
      type: 'symbol',
      source: 'boys-source',
      layout: {
        'icon-image': '{icon}',
        'icon-size': 0.2,
        'icon-anchor': 'center',
        'icon-rotation-alignment': 'viewport',
        'icon-allow-overlap': true
      }
    })

    // 添加名字图册
    map.addLayer({
      id: 'boys-name-layer',
      type: 'symbol',
      source: 'boys-source',
      layout: {
        'text-field': '{name}',
        "text-size": 14,
        "text-offset": [0, 2.4],  // 名字要设置偏移，避免被头像挡住
        "text-allow-overlap": true
      },
      paint: {
        "text-color": 'white',
      }
    })
  })
})
</script>

<style>
.map {
  width: 600px;
  height: 400px;
}
</style>