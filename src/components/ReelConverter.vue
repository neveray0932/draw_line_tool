<script setup>
import { ref, onMounted, watch } from 'vue'

const rows = ref(3)
const cols = ref(3)
const isServer = ref(false)
const beautify = ref(false)
const inputArray = ref('')
const result = ref('')
const symbolConfig = ref({
    S1: 0,
    S2: 1,
    S3: 2,
    A: 3,
    K: 4,
    Q: 5,
    J: 6,
    SC: 7,
    WD: 8,
    BG: 9,
    EMPTY: 10
})

const showInstructions = ref(false)

const toggleInstructions = () => {
    showInstructions.value = !showInstructions.value
}

const getSymbolColor = (value) => {
    const colors = {
        'S1': '#FF0000',
        'S2': '#800080',
        'S3': '#0000FF',
        'S4': '#FFA500',
        'S5': '#008000',
        'A': '#FF0000',
        'K': '#800080',
        'Q': '#0000FF',
        'J': '#FFA500',
        'T': '#008000',
        'WD': '#d6cfcd',
        'SC': '#0af1f9',
        'SP': '#f90ad8',
        'SP2': '#f90ad8'
    }

    for (const [symbol, num] of Object.entries(symbolConfig.value)) {
        if (num === value) {
            return colors[symbol] || '#000000'
        }
    }
    return '#000000'
}

const getSymbolText = (value) => {
    for (const [symbol, num] of Object.entries(symbolConfig.value)) {
        if (num === value) {
            return symbol
        }
    }
    return value
}

const displayMatrices = ref([])

const updateDisplayMatrices = () => {
    const input = inputArray.value.trim()
    if (!input) {
        displayMatrices.value = [Array(rows.value).fill().map(() => Array(cols.value).fill(0))]
        return
    }
    try {
        let inputArrays
        try {
            inputArrays = JSON.parse(input)
            if (!Array.isArray(inputArrays[0])) {
                inputArrays = [inputArrays]
            }
        } catch (e) {
            inputArrays = [input.split(',').map(num => parseInt(num.trim()))]
        }
        displayMatrices.value = inputArrays.map(numbers => {
            if (numbers.length !== rows.value * cols.value) {
                return Array(rows.value).fill().map(() => Array(cols.value).fill(0))
            }
            const matrix = []
            if (!isServer.value) {
                for (let j = 0; j < cols.value; j++) {
                    for (let i = 0; i < rows.value; i++) {
                        if (!matrix[i]) matrix[i] = []
                        matrix[i][j] = numbers[j * rows.value + i]
                    }
                }
            } else {
                for (let i = 0; i < rows.value; i++) {
                    matrix[i] = []
                    for (let j = 0; j < cols.value; j++) {
                        matrix[i][j] = numbers[i * cols.value + j]
                    }
                }
            }
            return matrix
        })
    } catch (e) {
        displayMatrices.value = [Array(rows.value).fill().map(() => Array(cols.value).fill(0))]
    }
}

watch([inputArray, rows, cols, isServer], () => {
    updateDisplayMatrices()
}, { immediate: true })

const updateInputArray = () => {
    const matrices = displayMatrices.value;
    const arrs = matrices.map(matrix => {
        const arr = [];
        for (let idx = 0; idx < rows.value * cols.value; idx++) {
            let value;
            if (isServer.value) {
                // Server: row-major
                const row = Math.floor(idx / cols.value);
                const col = idx % cols.value;
                value = matrix[row][col];
            } else {
                // Client: col-major
                const row = idx % rows.value;
                const col = Math.floor(idx / rows.value);
                value = matrix[row][col];
            }
            arr.push(value);
        }
        return arr;
    });
    inputArray.value = arrs.length === 1 ? JSON.stringify(arrs[0]) : JSON.stringify(arrs);
}

const convert = () => {
    const input = inputArray.value.trim()

    try {
        let inputArrays
        if (input === '') {
            inputArrays = [new Array(rows.value * cols.value).fill(0)]
        } else {
            inputArrays = JSON.parse(input)

            if (!Array.isArray(inputArrays)) {
                result.value = '請輸入有效的陣列格式！'
                return
            }

            if (Array.isArray(inputArrays[0])) {
                for (const array of inputArrays) {
                    if (!Array.isArray(array) || array.length !== rows.value * cols.value) {
                        result.value = `每個盤面必須包含 ${rows.value * cols.value} 個數字！`
                        return
                    }
                }
            } else {
                if (inputArrays.length !== rows.value * cols.value) {
                    result.value = `請輸入 ${rows.value * cols.value} 個數字！`
                    return
                }
                inputArrays = [inputArrays]
            }
        }

        let output = ''
        inputArrays.forEach((numbers, arrayIndex) => {
            const inputMatrix = []
            if (!isServer.value) {
                for (let j = 0; j < cols.value; j++) {
                    for (let i = 0; i < rows.value; i++) {
                        if (!inputMatrix[i]) inputMatrix[i] = []
                        inputMatrix[i][j] = numbers[j * rows.value + i]
                    }
                }
            } else {
                for (let i = 0; i < rows.value; i++) {
                    inputMatrix[i] = []
                    for (let j = 0; j < cols.value; j++) {
                        inputMatrix[i][j] = numbers[i * cols.value + j]
                    }
                }
            }

            const outputArray = []
            if (!isServer.value) {
                for (let i = 0; i < rows.value; i++) {
                    for (let j = 0; j < cols.value; j++) {
                        outputArray.push(inputMatrix[i][j])
                    }
                }
            } else {
                for (let j = 0; j < cols.value; j++) {
                    for (let i = 0; i < rows.value; i++) {
                        outputArray.push(inputMatrix[i][j])
                    }
                }
            }

            output += `${arrayIndex > 0 ? '\n\n' : ''}\n${JSON.stringify(outputArray)},`
        })
        // output += `${arrayIndex > 0 ? '\n\n' : ''}${isServer.value ? 'Client' : 'Server'} 盤面結果：\n${JSON.stringify(outputArray)}`

        result.value = output
    } catch (e) {
        const numbers = input.split(',').map(num => parseInt(num.trim()))
        if (numbers.length !== rows.value * cols.value) {
            result.value = `請輸入 ${rows.value * cols.value} 個數字！`
            return
        }

        const inputMatrix = []
        if (!isServer.value) {
            for (let j = 0; j < cols.value; j++) {
                for (let i = 0; i < rows.value; i++) {
                    if (!inputMatrix[i]) inputMatrix[i] = []
                    inputMatrix[i][j] = numbers[j * rows.value + i]
                }
            }
        } else {
            for (let i = 0; i < rows.value; i++) {
                inputMatrix[i] = []
                for (let j = 0; j < cols.value; j++) {
                    inputMatrix[i][j] = numbers[i * cols.value + j]
                }
            }
        }

        const outputArray = []
        if (!isServer.value) {
            for (let i = 0; i < rows.value; i++) {
                for (let j = 0; j < cols.value; j++) {
                    outputArray.push(inputMatrix[i][j])
                }
            }
        } else {
            for (let j = 0; j < cols.value; j++) {
                for (let i = 0; i < rows.value; i++) {
                    outputArray.push(inputMatrix[i][j])
                }
            }
        }

        result.value = `${isServer.value ? 'Client' : 'Server'} 盤面結果：\n${JSON.stringify(outputArray)}`
    }
}
</script>

<template>
    <div class="board-converter">
        <h1>盤面轉換</h1>

        <div class="instructions" :class="{ 'collapsed': !showInstructions }">
            <h3 @click="toggleInstructions">使用說明</h3>
            <div class="instructions-content">
                <ol>
                    <li>設定盤面大小：
                        <ul>
                            <li>輸入行數(rows)和列數(columns)</li>
                            <li>預設為 3x3 的盤面</li>
                        </ul>
                    </li>
                    <li>選擇盤面類型：
                        <ul>
                            <li>Client 盤面：數字由上到下、由左到右排列</li>
                            <li>Server 盤面：數字由左到右、由上到下排列</li>
                        </ul>
                    </li>
                    <li>輸入方式：
                        <ul>
                            <li>在下方文字框輸入數字陣列（用逗號分隔）</li>
                        </ul>
                    </li>
                    <li>美化功能（選用）：
                        <ul>
                            <li>勾選「套用美化」可以將數字轉換為符號</li>
                            <li>可以自訂符號對應的數字</li>
                        </ul>
                    </li>
                </ol>
            </div>
        </div>

        <div class="dimension-inputs">
            <div class="dimension-input">
                <label>行數(rows)：</label>
                <input type="number" v-model.number="rows" min="1">
            </div>
            <div class="dimension-input">
                <label>列數(columns)：</label>
                <input type="number" v-model.number="cols" min="1">
            </div>
        </div>

        <div class="input-type-selector">
            <div class="input-type-option">
                <input type="radio" id="clientInput" v-model="isServer" :value="false">
                <label for="clientInput">Client 盤面</label>
            </div>
            <div class="input-type-option">
                <input type="radio" id="serverInput" v-model="isServer" :value="true">
                <label for="serverInput">Server 盤面</label>
            </div>
        </div>

        <div class="beautify-option">
            <input type="checkbox" id="beautify" v-model="beautify">
            <label for="beautify">套用美化</label>
        </div>

        <div class="input-group">
            <label>請輸入數字陣列（用逗號分隔）：</label>
            <textarea v-model="inputArray" placeholder="例如：[1,2,3,4,5,6,7,8,9]"></textarea>
        </div>

        <div class="grid-section" v-for="(matrix, mIdx) in displayMatrices" :key="mIdx">
            <div class="grid-label">輸入盤面顯示 {{ displayMatrices.length > 1 ? `#${mIdx + 1}` : '' }}：</div>
            <div class="grid-container" :style="{ gridTemplateColumns: `repeat(${cols}, 1fr)` }">
                <div v-for="(cell, idx) in matrix.flat()" :key="idx" class="grid-item">
                    <input type="text" :value="beautify ? getSymbolText(cell) : cell"
                        :style="{ color: beautify ? getSymbolColor(cell) : '#000' }" @input="(e) => {
                            const value = parseInt(e.target.value)
                            if (!isNaN(value)) {
                                const row = Math.floor(idx / cols)
                                const col = idx % cols
                                matrix[row][col] = value
                                updateInputArray()
                            }
                        }" />
                    <span class="index">
                        {{ isServer ? idx : ((idx % cols) * rows + Math.floor(idx / cols)) }}
                    </span>
                </div>
            </div>
        </div>

        <div class="button-group">
            <button @click="convert">轉換</button>
        </div>

        <div class="result" v-if="result">
            <h4 v-show="result.trim().length > 0">{{isServer ? 'Client' : 'Server'}} 盤面結果:</h4>
            <pre>{{ result }}</pre>
        </div>
    </div>
</template>

<style scoped>
.board-converter {
    padding: 20px;
    max-width: 900px;
    margin: 0 auto;
}

.dimension-inputs {
    display: flex;
    gap: 20px;
    margin-bottom: 10px;
}

.dimension-input {
    display: flex;
    flex-direction: column;
    gap: 5px;
}

input[type="number"] {
    width: 100px;
    padding: 5px;
}

textarea {
    width: 100%;
    height: 100px;
    padding: 10px;
    margin-bottom: 10px;
}

.button-group {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

button {
    padding: 10px 20px;
    cursor: pointer;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
}

button:hover {
    background-color: #45a049;
}

.result {
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: #f8f9fa;
    white-space: pre-wrap;
}

.instructions {
    background-color: #f8f9fa;
    padding: 15px;
    border-radius: 5px;
    margin-bottom: 20px;
    border: 1px solid #e9ecef;
}

.instructions h3 {
    margin: 0;
    color: #333;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.instructions h3:after {
    content: '▼';
    font-size: 12px;
    transition: transform 0.3s;
}

.instructions.collapsed h3:after {
    transform: rotate(-90deg);
}

.instructions-content {
    margin-top: 15px;
    transition: max-height 0.3s ease-out;
    overflow: hidden;
    max-height: 1000px;
}

.instructions.collapsed .instructions-content {
    max-height: 0;
    margin-top: 0;
}

.instructions ol {
    margin: 0;
    padding-left: 20px;
}

.instructions ul {
    margin: 5px 0;
    padding-left: 20px;
}

.instructions li {
    margin-bottom: 10px;
}

.instructions ul li {
    margin-bottom: 5px;
}

.input-type-selector {
    display: flex;
    gap: 20px;
    margin-bottom: 10px;
}

.input-type-option {
    display: flex;
    align-items: center;
    gap: 5px;
}

.beautify-option {
    display: flex;
    align-items: center;
    gap: 5px;
    margin-bottom: 10px;
}

.grid-section {
    margin: 20px 0;
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 5px;
}

.grid-label {
    text-align: center;
    margin-bottom: 10px;
    font-weight: bold;
}

.grid-container {
    display: grid;
    gap: 2px;
    margin: 0 auto;
    max-width: 445px;
}

.grid-item {
    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f0f0f0;
    border: 1px solid #ccc;
    font-size: 18px;
    font-weight: bold;
    min-width: 5px;
    min-height: 5px;
    color: #000;
    position: relative;
}

.grid-item input {
    width: 100%;
    height: 100%;
    border: none;
    background: transparent;
    text-align: center;
    font-size: 18px;
    font-weight: bold;
    padding: 0;
    margin: 0;
    outline: none;
}

.grid-item input:focus {
    background-color: #e0e0e0;
}

.grid-item .index {
    position: absolute;
    top: 1px;
    right: 1px;
    font-size: 12px;
    color: #f10c0c;
}
</style>