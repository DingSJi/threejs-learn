<template>
  <div id="app">
    <!-- <HelloThree/> -->
    <!-- <TerrainRaycast/> -->
    <!-- <twoDHeatmap/> -->
    <!-- <threeDHeatmap/> -->
    <demoThreeVue/>
  </div>
</template>

<script>
import HelloThree from './components/HelloThree.vue'
import TerrainRaycast from '@/components/TerrainRaycast'
import twoDHeatmap from '@/components/twoDHeatmap'
import threeDHeatmap from '@/components/threeDHeatmap'
import demoThreeVue from './components/demoThree.vue'

export default {
  name: 'App',
  components: {
    HelloThree,
    TerrainRaycast,
    twoDHeatmap,
    threeDHeatmap,
    demoThreeVue
  }
}
</script>

<style>
body{
    margin: 0;
}
</style>
