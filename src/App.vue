<script setup>
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <div>
    <a href="https://github.com/AI-ML-Lab" target="_blank">
      <img src="https://github.com/AI-ML-Lab/resources/blob/main/images/m_dimensiones.gif?raw=true" class="logo" alt="Dimensiones" />
    </a>
    <a href="https://ai-ml-lab.github.io/" target="_blank">
      <!-- <img src="https://raw.githubusercontent.com/AI-ML-Lab/resources/main/images/m_logo.svg" width=150 class="logo vue" alt="Minsait logo" /> -->
    </a>
  </div>
  <h1>Laboratorio de Inteligencia Artificial</h1>
  <HelloWorld msg="" />
</template>

<style scoped>

img {
  margin-left: 30px;
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
