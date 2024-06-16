<template>
  <div id="map"></div>
</template>
<script setup lang="ts">
import { initMap, map } from '../home/initMap';
import { onMounted } from 'vue';

import OSM from 'ol/source/OSM.js';
import TileLayer from 'ol/layer/Tile.js';
import View from 'ol/View.js';
import WMTS from 'ol/source/WMTS.js';
import WMTSTileGrid from 'ol/tilegrid/WMTS.js';
import { get as getProjection } from 'ol/proj.js';
import { getTopLeft, getWidth } from 'ol/extent.js';

onMounted(() => {
  initMap();
  loadGeoserverWmtsLayer();
});

//封装加载geoserver-wmts图层(矢量瓦片)
const loadGeoserverWmtsLayer = () => {
  // create the WMTS tile grid in the google projection
  const projection = getProjection('EPSG:4326');
  const tileSizePixels = 256;
  const tileSizeMtrs = getWidth(projection.getExtent()) / tileSizePixels;
  const matrixIds = [];

  const resolutions = [];
  for (let i = 0; i <= 21; i++) {
    // 改动：原代码为matrixIds[i] =  i;
    matrixIds[i] = 'EPSG:4326:' + i;
    // 改动：原代码为resolutions[i] = tileSizeMtrs / Math.pow(2, i);
    resolutions[i] = tileSizeMtrs / Math.pow(2, i + 1);
  }
  const tileGrid = new WMTSTileGrid({
    origin: getTopLeft(projection.getExtent()),
    resolutions: resolutions,
    matrixIds: matrixIds,
  });

  const wmtsSource = new WMTS({
    url: 'http://localhost:8080/geoserver/gwc/service/wmts',
    layer: 'NanJing:njs2',
    format: 'image/png',
    matrixSet: 'EPSG:4326',

    tileGrid: tileGrid,
    style: '',
  });

  const wmtsLayer = new TileLayer({
    source: wmtsSource,
  });

  map.value.addLayer(wmtsLayer);
};
</script>
<style lang="scss" scoped>
#map {
  width: 100%;
  height: 100%;
}
</style>
