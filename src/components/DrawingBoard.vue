<template>
  <div class="drawing-board">
    <div class="grid-container" :style="gridStyle">
      <div
        v-for="(cellIndex, displayIndex) in getDisplayOrder"
        :key="cellIndex"
        class="grid-cell"
        :class="{ 'colored': cellStates[cellIndex] }"
        @click="toggleCell(cellIndex)"
        :style="cellStates[cellIndex] ? { backgroundColor: props.color } : {}"
      >
        <span class="cell-index">{{ displayIndex }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, defineExpose } from 'vue'

const props = defineProps({
  rows: {
    type: Number,
    required: true
  },
  cols: {
    type: Number,
    required: true
  },
  isServer: {
    type: Boolean,
    default: false
  },
  color: {
    type: String,
    default: '#FFA500'
  }
})

const cells = computed(() => new Array(props.rows * props.cols).fill(false))

const gridStyle = computed(() => ({
  display: 'grid',
  gridTemplateColumns: `repeat(${props.cols}, 1fr)`,
  gap: '1px',
  backgroundColor: '#ccc',
  padding: '1px'
}))

const cellStates = ref([])
const initCellStates = () => {
  cellStates.value = new Array(props.rows * props.cols).fill(false)
}

// 初始化
initCellStates()

// 監聽 rows/cols/isServer 變動時重建 cellStates
watch([() => props.rows, () => props.cols, () => props.isServer], initCellStates)

const toggleCell = (index) => {
  cellStates.value[index] = !cellStates.value[index]
}

const getDisplayOrder = computed(() => {
  const total = props.rows * props.cols
  const order = []
  if (!props.isServer) {
    // client: 直向遞增
    for (let col = 0; col < props.cols; col++) {
      for (let row = 0; row < props.rows; row++) {
        order.push(row + col * props.rows)
      }
    }
  } else {
    // server: 橫向遞增
    for (let i = 0; i < total; i++) {
      order.push(i)
    }
  }
  return order
})

const getSelectedIndexes = () => {
  // 回傳所有被選取的 index
  return cellStates.value
    .map((v, i) => (v ? i : null))
    .filter(i => i !== null)
}

const resetBoard = () => {
  initCellStates()
}

const getCellColorMatrix = () => {
  const matrix = []
  for (let r = 0; r < props.rows; r++) {
    const row = []
    for (let c = 0; c < props.cols; c++) {
      const idx = r * props.cols + c
      row.push(cellStates.value[idx] ? props.color : null)
    }
    matrix.push(row)
  }
  return matrix
}

defineExpose({ getSelectedIndexes, resetBoard, getCellColorMatrix })
</script>

<style scoped>
.drawing-board {
  display: inline-block;
  border: 1px solid #999;
  background-color: white;
}

.grid-cell {
  width: 30px;
  height: 30px;
  background-color: white;
  cursor: pointer;
  transition: background-color 0.2s;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.grid-cell:hover {
  background-color: #f0f0f0;
}

.grid-cell.colored {
  transition: background-color 0.2s;
}

.cell-index {
  font-size: 10px;
  color: #666;
  position: absolute;
  top: 2px;
  left: 2px;
}
</style> 