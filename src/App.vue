<script setup>
import CesiumViewer from './components/CesiumViewer.vue'
</script>

<template>
  <CesiumViewer />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  overflow: hidden;
}

#app {
  width: 100%;
  height: 100vh;
}
</style>
