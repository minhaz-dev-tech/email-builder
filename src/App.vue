<script setup>
import TestEditorComponent from "./components/TestEditorComponent.vue";
</script>

<template>
  <div>
    <TestEditorComponent />
  </div>
</template>

<style scoped>
#app {
  width: 100% !important;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
