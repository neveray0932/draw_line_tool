<script setup>
import { ref } from 'vue'

const tools = ref([
  { id: 'drawing', name: '畫線工具', icon: '🎨' },
  { id: 'converter', name: '盤面轉換', icon: '🔄' },
  { id: 'index-converter', name: 'Index轉換工具', icon: '🔢' }
])

const isCollapsed = ref(false)
const emit = defineEmits(['select-tool'])

const selectTool = (toolId) => {
  emit('select-tool', toolId)
}

const toggleSidebar = () => {
  isCollapsed.value = !isCollapsed.value
}
</script>

<template>
  <div class="sidebar" :class="{ 'collapsed': isCollapsed }">
    <span class="collapse-btn" @click="toggleSidebar">
      <i  class="fi fi-br-menu-burger"></i>
    </span>
    <div class="sidebar-content">
      <div class="sidebar-header">
        <h2 v-show="!isCollapsed">工具選單</h2>
      </div>
      <div class="tools-list">
        <button
          v-for="tool in tools"
          :key="tool.id"
          class="tool-button"
          @click="selectTool(tool.id)"
          :title="isCollapsed ? tool.name : ''"
        >
          <span class="tool-icon">{{ tool.icon }}</span>
          <span class="tool-name" v-show="!isCollapsed">{{ tool.name }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.sidebar {
  width: 250px;
  height: 100vh;
  background-color: #2c3e50;
  color: white;
  padding: 20px 0;
  position: fixed;
  left: 0;
  top: 0;
  transition: width 0.3s ease;
  overflow: hidden;
}

.sidebar.collapsed {
  width: 60px;
}

.sidebar-content {
  width: 250px;
  transition: transform 0.3s ease;
}

.collapsed .sidebar-content {
  transform: translateX(-190px);
}

.collapse-btn {
  position: absolute;
  right: 5px;
  top: 20px;
  width: 25px;
  height: 25px;
  background-color: #2c3e50;
  /* border: 2px solid #fff; */
  border-radius: 10%;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  transition: all 0.3s ease;
  padding: 0;
  line-height: 1;
}

.collapse-btn:hover {
  background-color: #34495e;
  transform: scale(1.1);
}

.collapse-icon {
  font-size: 14px;
  transition: transform 0.3s ease;
  display: inline-block;
  line-height: 1;
}

.collapsed .collapse-icon {
  transform: rotate(180deg);
}

.sidebar-header {
  padding: 0 20px;
  margin-bottom: 20px;
}

.sidebar-header h2 {
  margin: 0;
  font-size: 1.5em;
  color: #fff;
  white-space: nowrap;
}

.tools-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.tool-button {
  display: flex;
  align-items: center;
  padding: 12px 20px;
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
  text-align: left;
  width: 100%;
  white-space: nowrap;
}

.tool-button:hover {
  background-color: #34495e;
}

.tool-icon {
  margin-right: 10px;
  font-size: 1.2em;
  min-width: 24px;
  text-align: center;
}

.tool-name {
  font-size: 1em;
}
</style> 