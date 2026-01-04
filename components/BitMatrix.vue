<script setup lang="ts">
import { ref } from 'vue'

// Sample data: 8 bytes, 8 bits each
// We'll use a simple pattern to make the transposition visible
const input = ref([
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 0: bit 0 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 1: bit 1 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 2: bit 2 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 3: bit 3 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 4: bit 4 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 5: bit 5 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 6: bit 6 is 1
  [0, 1, 1, 0, 0, 0, 0, 1], // Byte 7: bit 7 is 1
])

// Initialize output with null/0
const output = ref(Array(8).fill(0).map(() => Array(8).fill(0)))

const activeI = ref(-1) // Current Input Byte Index
const activeJ = ref(-1) // Current Bit Index
const isPlaying = ref(false)

const reset = () => {
  output.value = Array(8).fill(0).map(() => Array(8).fill(0))
  activeI.value = -1
  activeJ.value = -1
  isPlaying.value = false
}

const play = async () => {
  if (isPlaying.value) return
  reset()
  isPlaying.value = true
  
  // Loop i from 0 to 7 (Input Bytes)
  for (let i = 0; i < 8; i++) {
    activeI.value = i
    // Loop j from 0 to 7 (Bits)
    for (let j = 0; j < 8; j++) {
      activeJ.value = j
      
      // Logic: output[j] |= (input[i][j] << i)
      // So output[j][i] = input[i][j]
      // Note: Visual grid is [row][col]
      // Input Grid: row=i, col=j
      // Output Grid: row=j, col=i
      
      output.value[j][i] = input.value[i][j]
      
      await new Promise(r => setTimeout(r, 30)) // Animation speed
    }
  }
  
  activeI.value = -1
  activeJ.value = -1
  isPlaying.value = false
}
</script>

<template>
  <div class="flex flex-col items-center justify-center h-full p-4">
    <div class="flex items-start gap-8">
      
      <!-- Input Matrix -->
      <div class="flex flex-col items-center">
        <h3 class="mb-2 font-bold text-sm">Input Bytes (i)</h3>
        <div class="relative">
          <!-- Column Headers (j) -->
          <div class="flex mb-1 ml-6">
             <div v-for="j in 8" :key="'h-in-'+j" class="w-6 text-center text-[10px] text-gray-400">{{ j-1 }}</div>
          </div>
          
          <div class="flex" v-for="(row, i) in input" :key="'in-row-'+i">
            <!-- Row Header (i) -->
            <div class="w-6 flex items-center justify-center text-[10px] text-gray-400 mr-1">{{ i }}</div>
            
            <div 
              v-for="(bit, j) in row" 
              :key="'in-'+i+'-'+j"
              class="w-6 h-6 flex items-center justify-center text-xs border border-gray-200 dark:border-gray-700 transition-all duration-200"
              :class="{
                'bg-blue-500 text-white scale-110 z-10 shadow': activeI === i && activeJ === j,
                'bg-blue-100 dark:bg-blue-900/30': activeI === i && activeJ !== j,
                'bg-white dark:bg-gray-800': activeI !== i
              }"
            >
              {{ bit }}
            </div>
          </div>
        </div>
        <div class="mt-2 text-xs text-gray-500">Processing Byte i={{ activeI >= 0 ? activeI : '-' }}</div>
      </div>

      <!-- Animation Arrow -->
      <div class="flex flex-col justify-center items-center h-64 pt-6">
        <div class="text-2xl animate-pulse">➜</div>
        <div class="text-[10px] text-gray-400 mt-1">Transpose</div>
        <div class="text-[10px] text-gray-400">out[j][i] = in[i][j]</div>
      </div>

      <!-- Output Matrix -->
      <div class="flex flex-col items-center">
        <h3 class="mb-2 font-bold text-sm">Output Bytes (j)</h3>
        <div class="relative">
           <!-- Column Headers (i) -->
           <div class="flex mb-1 ml-6">
             <div v-for="i in 8" :key="'h-out-'+i" class="w-6 text-center text-[10px] text-gray-400">{{ i-1 }}</div>
          </div>

          <div class="flex" v-for="(row, j) in output" :key="'out-row-'+j">
            <!-- Row Header (j) -->
            <div class="w-6 flex items-center justify-center text-[10px] text-gray-400 mr-1">{{ j }}</div>

            <div 
              v-for="(bit, i) in row" 
              :key="'out-'+j+'-'+i"
              class="w-6 h-6 flex items-center justify-center text-xs border border-gray-200 dark:border-gray-700 transition-all duration-200"
              :class="{
                'bg-green-500 text-white scale-110 z-10 shadow': activeI === i && activeJ === j,
                'bg-green-100 dark:bg-green-900/30': output[j][i] !== undefined && (activeI > i || (activeI === i && activeJ >= j)),
                'bg-gray-50 dark:bg-gray-900': !(output[j][i] !== undefined && (activeI > i || (activeI === i && activeJ >= j)))
              }"
            >
              {{ bit }}
            </div>
          </div>
        </div>
        <div class="mt-2 text-xs text-gray-500">Filling Byte j={{ activeJ >= 0 ? activeJ : '-' }}</div>
      </div>
    </div>

    <button 
      @click="play" 
      class="mt-6 px-6 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-full text-sm font-medium transition-colors shadow-lg flex items-center gap-2"
      :disabled="isPlaying"
    >
      <span v-if="!isPlaying">▶ Play Animation</span>
      <span v-else>Running...</span>
    </button>
  </div>
</template>
