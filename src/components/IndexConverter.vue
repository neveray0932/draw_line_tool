<script setup>
import { ref } from 'vue'

const rows = ref(3)
const cols = ref(3)
const inputArray = ref('')
const mode = ref('client') // 'client' or 'server'
const result = ref('')

const convert = () => {
  let arr
  try {
    arr = JSON.parse(inputArray.value)
    if (!Array.isArray(arr)) throw new Error()
  } catch {
    arr = inputArray.value.split(',').map(x => parseInt(x.trim()))
  }
  if (!Array.isArray(arr) || arr.some(x => Array.isArray(x) ? x.some(isNaN) : isNaN(x))) {
    result.value = '請輸入有效的數字陣列'
    return
  }
  // 支援二維陣列
  if (Array.isArray(arr[0])) {
    // 多組陣列
    const results = arr.map(subArr => {
      if (mode.value === 'client') {
        return toClientPoses(subArr)
      } else {
        return toProbPoses(subArr)
      }
    })
    result.value = JSON.stringify(results)
  } else {
    if (mode.value === 'client') {
      result.value = JSON.stringify(toClientPoses(arr))
    } else {
      result.value = JSON.stringify(toProbPoses(arr))
    }
  }
}

const toClientReelsIndex = (rows, column, indices) => {
    // 不规则盘面
    if (Array.isArray(rows)) {
        // 获取客端盘面每列开头的索引
        const idxArray = [0];
        for (let i = 1; i < column; i++) {
            const lastIndex = i - 1;
            idxArray[i] = idxArray[lastIndex] + rows[lastIndex];
        }
        // 将概率盘面索引映射到客户端盘面上
        const flattened = [];
        const maxRow = Math.max(...rows);
        for (let iRow = 0; iRow < maxRow; iRow++) {
            for (let iReel = 0; iReel < column; iReel++) {
                if (iRow < rows[iReel]) {
                    flattened[flattened.length] = idxArray[iReel] + iRow;
                }
            }
        }

        return indices !== undefined ? indices.map(v => flattened[v]) : flattened;
    }

    if (indices === undefined) {
        indices = [];
        const size = rows * column;
        // 将概率盘面索引映射到客户端盘面上
        for (let idx = 0; idx < size; idx++) {
            const rowPos = Math.floor(idx / column);
            indices[idx] = (idx % column) * rows + rowPos;
        }

        return indices;
    }

    // 将概率盘面索引映射到客户端盘面上
    return indices.map((idx) => {
        const rowPos = Math.floor(idx / column);

        return (idx % column) * rows + rowPos;
    });
};

const toProbReelsIndex = (rows, column, indices) => {
    // 不规则盘面
    if (Array.isArray(rows)) {
        const probToClientIndices = toClientReelsIndex(rows, column);
        if (indices !== undefined) {
            return indices.map(v => probToClientIndices.indexOf(v));
        }
        // 全客端盘面索引映射
        const len = rows.length;
        let total = 0;
        for (let i = 0; i < len; i++) {
            total += rows[i];
        }
        const flattened = [];
        for (let i = 0; i < total; i++) {
            flattened[i] = probToClientIndices.indexOf(i);
        }

        return flattened;
    }

    // swap row, column
    return toClientReelsIndex(column, rows, indices);
};

const toClientPoses = (poses) => {
  return poses.map(value => toClientReelsIndex(rows.value, cols.value, [value])[0]);
}

const toProbPoses = (poses) => {
  return poses.map(value => toProbReelsIndex(rows.value, cols.value, [value])[0]);
}

</script>

<template>
  <div class="index-converter">
    <h1>Index 轉換工具</h1>
    <div class="input-container">
      <div class="input-group">
        <label>行數 (row)：</label>
        <input type="number" v-model.number="rows" min="1" />
      </div>
      <div class="input-group">
        <label>列數 (col)：</label>
        <input type="number" v-model.number="cols" min="1" />
      </div>
      <div class="input-group">
        <label>數字陣列：</label>
        <textarea v-model="inputArray" placeholder="如 [0,1,2] 或 0,1,2"></textarea>
      </div>
      <div class="radio-group">
      <label>
        <input type="radio" value="client" v-model="mode" /> Client Index
      </label>
      <label>
        <input type="radio" value="server" v-model="mode" /> Server Index
      </label>
    </div>
    </div>
    <button class="convert-btn" @click="convert">轉換</button>
    <div v-if="result" class="result-block">
      <h2>結果：</h2>
      <pre>{{ result }}</pre>
    </div>
  </div>
</template>

<style scoped>
.index-converter {
  padding: 30px 20px;
  background: #f9f9f9;
  border-radius: 12px;
  max-width: 900px;
  margin: 30px auto;
  box-shadow: 0 2px 8px rgba(44,62,80,0.08);
  text-align: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 24px;
  font-size: 2em;
  letter-spacing: 1px;
}

.input-container {
  display: flex;
  justify-content: center;
  gap: 24px;
  flex-wrap: wrap;
  margin-bottom: 18px;
}

.input-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 8px;
}

.input-group label {
  font-weight: bold;
  color: #2c3e50;
}

input[type='number'] {
  width: 140px;
  padding: 7px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1em;
  background: #fff;
}

textarea {
  width: 350px;
  min-height: 250px;
  padding: 7px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1em;
  background: #fff;
  resize: none;
  box-sizing: border-box;
}

.radio-group {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 18px;
}

.radio-group label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-weight: 500;
  color: #2c3e50;
  cursor: pointer;
}

.convert-btn {
  padding: 10px 28px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  font-size: 1.1em;
  margin-bottom: 18px;
  transition: background 0.2s;
}
.convert-btn:hover {
  background-color: #45a049;
}

.result-block {
  margin-top: 18px;
  padding: 18px 10px;
  background: #fff;
  border-radius: 8px;
  border: 1px solid #e0e0e0;
  text-align: left;
  box-shadow: 0 1px 4px rgba(44,62,80,0.04);
}
.result-block h2 {
  margin: 0 0 8px 0;
  color: #2196F3;
  font-size: 1.1em;
}
pre {
  margin: 0;
  font-size: 1.1em;
  color: #333;
  background: none;
  border: none;
}
</style> 