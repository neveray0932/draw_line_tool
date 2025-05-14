<script setup>
import { ref, onMounted, nextTick } from 'vue'
import Sidebar from './components/Sidebar.vue'
import DrawingTool from './components/DrawingTool.vue'
import ReelConverter from './components/ReelConverter.vue'
import IndexConverter from './components/IndexConverter.vue'

const currentTool = ref('drawing')

const handleToolSelect = (toolId) => {
  currentTool.value = toolId
}
</script>

<template>
  <div class="app-container">
    <Sidebar @select-tool="handleToolSelect" />
    <div class="main-content">
      <keep-alive>
        <DrawingTool v-if="currentTool === 'drawing'" />
        <ReelConverter v-else-if="currentTool === 'converter'" />
        <IndexConverter v-else-if="currentTool === 'index-converter'" />
      </keep-alive>
    </div>
  </div>
</template>

<style>
.app-container {
  display: flex;
  min-height: 100vh;
}

.main-content {
  flex: 1;
  margin-left: 250px;
  padding: 20px;
  background-color: #f5f5f5;
  transition: margin-left 0.3s ease;
}

/* 當側邊欄摺疊時，調整主內容區域的邊距 */
.sidebar.collapsed ~ .main-content {
  margin-left: 60px;
}

.coming-soon {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  color: #666;
}

body {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}
</style>
