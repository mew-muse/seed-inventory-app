<template>
  <div class="min-h-screen bg-gradient-to-br">
    <header class="bg-white shadow-sm px-4 py-6">
      <div class="max-w-md mx-auto">
        <h1 class="text-2xl font-bold text-garden-green">
          🌱 Seed Inventory - Simple Test
        </h1>
        <p class="text-gray-600 text-sm mt-1">Testing if Vue is working</p>
      </div>
    </header>
    
    <main class="max-w-md mx-auto px-4 py-6">
      <div class="bg-white rounded-lg p-4 shadow-sm">
        <h2>Vue.js is working!</h2>
        <p>Count: {{ count }}</p>
        <button @click="count++" class="bg-garden-green text-white px-4 py-2 rounded">
          Increment
        </button>
      </div>
    </main>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'App',
  setup() {
    const count = ref(0)
    
    return {
      count
    }
  }
}
</script>