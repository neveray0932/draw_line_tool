<script setup>
import { ref, onMounted, nextTick } from 'vue'
import DrawingBoard from './components/DrawingBoard.vue'
import * as XLSX from 'xlsx'

const rows = ref(3)
const cols = ref(5)
const count = ref(20)
const boards = ref([])
const isServer = ref(true)
const color = ref('#FFA500')

// 新增：用來存放每個畫圖板的 ref
const boardRefs = ref([])

const setBoardRef = (el, idx) => {
  if (el) {
    boardRefs.value[idx] = el
  }
}

const generateBoards = async () => {
  boards.value = Array(count.value).fill(null)
  await nextTick()
  boardRefs.value = Array(count.value).fill(null)
}

// 新增：產出所有畫圖板的選取 index
const allSelectedIndexes = ref([])
const outputIndexes = () => {
  allSelectedIndexes.value = boardRefs.value.map(ref => ref?.getSelectedIndexes?.() || [])
}

const resetAllBoards = () => {
  boardRefs.value.forEach(ref => ref?.resetBoard?.())
}

// 新增格式化函數
const formatIndexes = (arr) => {
  return '[\n' + arr.map(row => '  [' + row.join(', ') + ']').join(',\n') + '\n]'
}

const exportToExcel = () => {
  // 取得每個畫圖板的顏色矩陣
  const wb = XLSX.utils.book_new()
  boardRefs.value.forEach((ref, idx) => {
    const colorMatrix = ref?.getCellColorMatrix?.() || []
    // 產生空白工作表
    const ws = XLSX.utils.aoa_to_sheet(colorMatrix.map(row => row.map(() => '')))
    // 設定每個 cell 的背景色
    colorMatrix.forEach((row, r) => {
      row.forEach((color, c) => {
        if (color) {
          const cellAddr = XLSX.utils.encode_cell({ r, c })
          if (!ws[cellAddr]) ws[cellAddr] = { t: 's', v: '' }
          ws[cellAddr].s = {
            fill: { patternType: 'solid', fgColor: { rgb: color.replace('#', '').toUpperCase() } }
          }
        }
      })
    })
    XLSX.utils.book_append_sheet(wb, ws, `Board${idx + 1}`)
  })
  // 啟用 cell style 支援
  XLSX.writeFile(wb, 'boards.xlsx', { cellStyles: true })
}
</script>

<template>
  <div class="app">
    <h1>畫圖工具</h1>
    <div class="input-container">
      <div class="input-group">
        <label>行數 (Row):</label>
        <input type="number" v-model.number="rows" min="1" max="20" />
      </div>
      <div class="input-group">
        <label>列數 (Col):</label>
        <input type="number" v-model.number="cols" min="1" max="20" />
      </div>
      <div class="input-group">
        <label>數量:</label>
        <input type="number" v-model.number="count" min="1" max="50" />
      </div>
      <div class="input-group">
        <label>顏色:</label>
        <input type="color" v-model="color" />
      </div>
      <div class="radio-group">
        <label>
          <input type="radio" v-model="isServer" :value="true" />
          Server
        </label>
      </div>
      <button @click="generateBoards" class="generate-btn">產生畫圖板</button>
      <button @click="outputIndexes" class="generate-btn" style="background:#2196F3;margin-left:10px;">產出 index</button>
      <button @click="resetAllBoards" class="generate-btn" style="background:#f44336;margin-left:10px;">重置</button>
    </div>
    <div class="boards-container">
      <div v-for="(board, index) in boards" :key="index" class="board-wrapper">
        <h3>畫圖板 {{ index + 1 }}</h3>
        <DrawingBoard :rows="rows" :cols="cols" :is-server="isServer" :color="color" :ref="el => setBoardRef(el, index)" />
      </div>
    </div>
    <div v-if="allSelectedIndexes.length" style="margin-top:30px;text-align:left;max-width:900px;margin-left:auto;margin-right:auto;">
      <h2>產出結果：</h2>
      <pre>{{ formatIndexes(allSelectedIndexes) }}</pre>
    </div>
  </div>
</template>

<style>
.app {
  padding: 20px;
  text-align: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 20px;
}

.input-container {
  margin-bottom: 30px;
  display: flex;
  justify-content: center;
  gap: 20px;
  align-items: center;
  flex-wrap: wrap;
}

.input-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.input-group label {
  font-weight: bold;
  color: #2c3e50;
}

.radio-group {
  display: flex;
  gap: 15px;
}

.radio-group label {
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
}

input[type="number"] {
  width: 60px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.generate-btn {
  padding: 8px 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

.generate-btn:hover {
  background-color: #45a049;
}

.boards-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  padding: 20px;
}

.board-wrapper {
  border: 1px solid #ddd;
  padding: 15px;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.board-wrapper h3 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #2c3e50;
}
</style>
