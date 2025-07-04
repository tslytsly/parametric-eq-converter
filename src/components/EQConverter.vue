<template>

  <div class="max-w-3xl mx-auto p-6 rounded-xl shadow-lg bg-white text-gray-900 dark:bg-gray-900 dark:text-white transition-colors">
    <h1 class="text-2xl font-bold text-center mb-6">Parametric EQ Converter</h1>


    <div class="flex space-x-4 mb-6">
      <button
        :class="[
          'px-4 py-2 rounded-t-lg font-semibold',
          activeTab === 'manual'
            ? 'bg-purple-600 text-white'
            : 'bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200'
        ]"
        @click="activeTab = 'manual'"
      >
        Manual Input
      </button>
      <button
        :class="[
          'px-4 py-2 rounded-t-lg font-semibold',
          activeTab === 'upload'
            ? 'bg-purple-600 text-white'
            : 'bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200'
        ]"
        @click="activeTab = 'upload'"
      >
        Upload REW Filter File
      </button>
    </div>

    <div v-if="activeTab === 'manual'" class="space-y-6">
      <div>
        <label class="block font-semibold mb-1">Center Frequency (Hz):</label>
        <input
          v-model.number="frequency"
          type="number"
          @input="updateSliderFromFreq"
          class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <input
          type="range"
          min="0"
          max="100"
          v-model.number="freqSlider"
          @input="updateFreqFromSlider"
          class="w-full mt-2"
        />
      </div>

      <div>
        <label class="block font-semibold mb-1">Gain (dB):</label>
        <input
          v-model.number="gain"
          type="number"
          step="0.01"
          class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <input
            v-model.number="gain"
            type="range"
            min="-12"
            max="12"
            step="0.1"
            class="w-full mt-2"
          />

      </div>

      <div>

        <label class="block font-semibold mb-1">Q:</label>
        <input
          v-model.number="q"
          type="number"
          step="0.01"
          @input="updateSliderFromQ"
          class="w-full p-2 border rounded-md"
        />
        <input
          type="range"
          min="0"
          max="100"
          v-model.number="qSlider"
          @input="updateQFromSlider"
          class="w-full mt-2"
        />
        <button
          @click="convertFromQ"
          class="mt-2 px-4 py-2 rounded transition
                bg-blue-600 text-white hover:bg-blue-700
                dark:bg-purple-600 dark:hover:bg-purple-700"
        >
          Convert from Q
        </button>
      </div>


      <div>
        <label class="block font-semibold mb-1">Bandwidth (Octaves):</label>
        <input
        v-model.number="bandwidth"
        type="number"
        step="0.01"
        class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <input
        v-model.number="bandwidth"
        type="range"
        min="0.1"
        max="5"
        step="0.01"
        class="w-full mt-2"
        />
        <button
        @click="convertFromBandwidth"
        class="mt-2 px-4 py-2 rounded transition
        bg-blue-600 text-white hover:bg-blue-700
        dark:bg-purple-600 dark:hover:bg-purple-700"
        >
        Convert from Bandwidth
      </button>
    </div>
  </div>
  <div v-if="activeTab === 'upload'" class="space-y-4">
    <div>
      <label class="block font-semibold mb-1">Upload REW Generic Filters:</label>
      <label
        for="file-upload"
        class="inline-block mt-4 px-4 py-2 rounded cursor-pointer transition
              bg-purple-600 text-white hover:bg-purple-700 dark:bg-purple-600 dark:hover:bg-purple-700"
      >
        Upload Filter File
      </label>
      <input
        id="file-upload"
        type="file"
        accept=".txt"
        @change="handleFileUpload"
        class="hidden"
      />

      <div v-if="filters.length" class="w-full p-4 border rounded-lg bg-gray-100 border-gray-300 dark:bg-gray-800 dark:border-gray-700 text-left mt-6">
        <h2 class="text-lg font-semibold mb-4">Parsed Filter Settings</h2>
        <table class="table-auto w-full text-left">
          <thead>
            <tr class="font-semibold">
              <th class="pr-4">Frequency (Hz)</th>
              <th class="pr-4">Gain (dB)</th>
              <th class="pr-4">Q</th>
              <th class="pr-4">Bandwidth (octaves)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(f, i) in filters" :key="i">
              <td class="pr-4">{{ f.frequency }}</td>
              <td class="pr-4">{{ f.gain }}</td>
              <td class="pr-4">{{ f.q }}</td>
              <td class="pr-4">{{ f.bandwidth.toFixed(decimalPlaces) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>


  <div v-if="activeTab === 'manual'" class="mt-8 flex flex-wrap gap-4">
      <div class="w-full md:w-[48%] p-4 border rounded-lg bg-gray-100 border-gray-300 dark:bg-gray-800 dark:border-gray-700 text-left">
        <h2 class="text-lg font-semibold mb-4">Q-Based Settings</h2>
        <table class="table-auto w-full text-left">
          <tbody>
            <tr>
              <th class="pr-4 font-semibold">Frequency:</th>
              <td>{{ frequency }} Hz</td>
            </tr>
            <tr>
              <th class="pr-4 font-semibold">Gain:</th>
              <td>{{ gain }} dB</td>
            </tr>
            <tr>
              <th class="pr-4 font-semibold">Q:</th>
              <td>{{ calculatedQ }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="w-full md:w-[48%] p-4 border rounded-lg bg-gray-100 border-gray-300 dark:bg-gray-800 dark:border-gray-700 text-left">
        <h2 class="text-lg font-semibold mb-4">Bandwidth-Based Settings</h2>
        <table class="table-auto w-full text-left">
          <tbody>
            <tr>
              <th class="pr-4 font-semibold">Frequency:</th>
              <td>{{ frequency }} Hz</td>
            </tr>
            <tr>
              <th class="pr-4 font-semibold">Gain:</th>
              <td>{{ gain }} dB</td>
            </tr>
            <tr>
              <th class="pr-4 font-semibold">Bandwidth:</th>
              <td>{{ calculatedBandwidth }} octaves</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div>
        <label class="block font-semibold mb-1">Decimal Places:</label>
        <select
          v-model.number="decimalPlaces"
          class="w-full p-2 border rounded-md text-gray-900 dark:text-white bg-white dark:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option v-for="n in 11" :key="n-1" :value="n-1">{{ n - 1 }}</option>
        </select>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'

const darkMode = ref(true)
// Toggle the `dark` class on the <html> element
watch(darkMode, (enabled) => {
  document.documentElement.classList.toggle('dark', enabled)
})

const activeTab = ref('manual') // 'manual' or 'upload'

const frequency = ref(1000)
const gain = ref(0)
const q = ref(1)
const bandwidth = ref(1)
const decimalPlaces = ref(2)

const calculatedQ = ref(q.value)
const calculatedBandwidth = ref(null)


// Slider range (linear)
const qSlider = ref(50) // midpoint
const freqSlider = ref(50) // midpoint

// Logarithmic mapping
const minQ = 0.01
const maxQ = 120
const minFreq = 20
const maxFreq = 20000

const logMinQ = Math.log10(minQ)
const logMaxQ = Math.log10(maxQ)
const logMinFq = Math.log10(minFreq)
const logMaxFq = Math.log10(maxFreq)

const updateQFromSlider = () => {
  const logValue = logMinQ + (qSlider.value / 100) * (logMaxQ - logMinQ)
  q.value = parseFloat((10 ** logValue).toFixed(decimalPlaces.value))
}

const updateSliderFromQ = () => {
  const logValue = Math.log10(q.value)
  qSlider.value = ((logValue - logMinQ) / (logMaxQ - logMinQ)) * 100
}

const updateFreqFromSlider = () => {
  const logValue = logMinFq + (freqSlider.value / 100) * (logMaxFq - logMinFq)
  frequency.value = parseFloat((10 ** logValue).toFixed(0))
}

const updateSliderFromFreq = () => {
  const logValue = Math.log10(frequency.value)
  freqSlider.value = ((logValue - logMinFq) / (logMaxFq - logMinFq)) * 100
}

// Sync slider when Q changes
watch(q, updateSliderFromQ)

const filters = ref([])

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const text = e.target.result
    parseFilterFile(text)
  }
  reader.readAsText(file)
}

const parseFilterFile = (text) => {
  const lines = text.split('\n')
  const parsedFilters = []

  const regex = /Filter\s+\d+:\s+ON\s+PK\s+Fc\s+([\d.]+)\s+Hz\s+Gain\s+([-\d.]+)\s+dB\s+Q\s+([\d.]+)/

  for (const line of lines) {
    const match = line.match(regex)
    if (match) {
      const [_, fc, gain, q] = match.map(Number)
      const bw = convertQtoBandwidth(q)
      parsedFilters.push({ frequency: fc, gain, q, bandwidth: bw })
    }
  }

  filters.value = parsedFilters
}

const convertQtoBandwidth = (q) => {
  const sqrtTerm = Math.sqrt(4 * q ** 2 + 1)
  const numerator = sqrtTerm + 1
  const denominator = sqrtTerm - 1
  return Math.log2(numerator / denominator)
}


const convertFromQ = () => {
  if (q.value) {
    const sqrtTerm = Math.sqrt(4 * q.value ** 2 + 1)
    const numerator = sqrtTerm + 1
    const denominator = sqrtTerm - 1
    const bwOctaves = Math.log2(numerator / denominator)
    calculatedBandwidth.value = +bwOctaves.toFixed(decimalPlaces.value)
    calculatedQ.value = +q.value.toFixed(2)
  }
}

const convertFromBandwidth = () => {
  if (bandwidth.value) {
    const powTerm = Math.pow(2, bandwidth.value)
    const numerator = Math.sqrt(powTerm)
    const denominator = powTerm - 1
    const qVal = numerator / denominator
    calculatedQ.value = +qVal.toFixed(decimalPlaces.value)
    calculatedBandwidth.value = +bandwidth.value.toFixed(decimalPlaces.value)
  }
}
</script>

