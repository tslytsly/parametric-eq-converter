<template>

  <div class="max-w-3xl mx-auto p-6 rounded-xl shadow-lg bg-white text-gray-900 dark:bg-gray-900 dark:text-white transition-colors">
    <h1 class="text-2xl font-bold text-center mb-6">Parametric EQ Converter</h1>

    <div class="space-y-6">

      <div>
        <label class="block font-semibold mb-1">Center Frequency (Hz):</label>
        <input
          v-model.number="frequency"
          type="number"
          class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
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
      </div>

      <div>
        <label class="block font-semibold mb-1">Q:</label>
        <input
          v-model.number="q"
          type="number"
          step="0.01"
          class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
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

    <div class="mt-8 flex flex-wrap gap-4">
      <div class="flex-1 min-w-[45%] p-4 border rounded-lg bg-gray-100 border-gray-300 dark:bg-gray-800 dark:border-gray-700 text-left">
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

      <div class="flex-1 min-w-[45%] p-4 border rounded-lg bg-gray-100 border-gray-300 dark:bg-gray-800 dark:border-gray-700 text-left">
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
import { ref, watch } from 'vue'

const darkMode = ref(true)
// Toggle the `dark` class on the <html> element
watch(darkMode, (enabled) => {
  document.documentElement.classList.toggle('dark', enabled)
})

const frequency = ref(1000)
const gain = ref(0)
const q = ref(1)
const bandwidth = ref(1)
const decimalPlaces = ref(2)

const calculatedQ = ref(q.value)
const calculatedBandwidth = ref(null)

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

