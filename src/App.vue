<template>
  <div class="min-h-screen bg-gradient-to-br from-green-50 to-emerald-50">
    <!-- Header -->
    <header class="bg-white shadow-sm px-4 py-6">
      <div class="max-w-md mx-auto">
        <h1 class="text-2xl font-bold text-garden-green flex items-center gap-2">
          🌱 Seed Inventory
        </h1>
        <p class="text-gray-600 text-sm mt-1">Track your seeds, grow your garden</p>
      </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-md mx-auto px-4 py-6">
      <!-- Stats Cards -->
      <div class="grid grid-cols-2 gap-4 mb-6">
        <div class="bg-white rounded-lg p-4 shadow-sm">
          <div class="text-2xl font-bold text-garden-green">{{ totalSeeds }}</div>
          <div class="text-sm text-gray-600">Total Seeds</div>
        </div>
        <div class="bg-white rounded-lg p-4 shadow-sm">
          <div class="text-2xl font-bold text-orange-500">{{ lowStockCount }}</div>
          <div class="text-sm text-gray-600">Low Stock</div>
        </div>
      </div>

      <!-- Add New Seed Button -->
      <button 
        @click="showAddForm = true"
        class="w-full bg-garden-green text-white py-4 px-6 rounded-lg font-semibold text-lg shadow-md hover:bg-green-600 transition-colors mb-6 flex items-center justify-center gap-2"
      >
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"/>
        </svg>
        Add New Seed
      </button>

      <!-- Seeds List -->
      <div class="space-y-4">
        <div v-if="seeds.length === 0" class="text-center py-12">
          <div class="text-6xl mb-4">🌱</div>
          <h3 class="text-lg font-medium text-gray-700 mb-2">No seeds yet!</h3>
          <p class="text-gray-500">Start by adding your first seed packet</p>
        </div>
        
        <div v-for="seed in seeds" :key="seed.id" class="bg-white rounded-lg p-4 shadow-sm">
          <div class="flex justify-between items-start mb-2">
            <h3 class="font-semibold text-gray-800">{{ seed.name }}</h3>
            <span class="text-sm text-gray-500">{{ seed.variety }}</span>
          </div>
          <div class="flex justify-between items-center">
            <div class="text-sm text-gray-600">
              Qty: {{ seed.quantity }} • Exp: {{ formatDate(seed.expiry) }}
            </div>
            <button 
              @click="plantSeed(seed)"
              class="bg-garden-green text-white px-3 py-1 rounded text-sm hover:bg-green-600 transition-colors"
            >
              Plant
            </button>
          </div>
        </div>
      </div>
    </main>

    <!-- Add Seed Modal (Simple for now) -->
    <div v-if="showAddForm" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-md">
        <h2 class="text-xl font-bold mb-4">Add New Seed</h2>
        <form @submit.prevent="addSeed">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">Name</label>
            <input 
              v-model="newSeed.name" 
              type="text" 
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
          </div>
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">Variety</label>
            <input 
              v-model="newSeed.variety" 
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
          </div>
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">Quantity</label>
            <input 
              v-model="newSeed.quantity" 
              type="number" 
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
          </div>
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-1">Expiry Date</label>
            <input 
              v-model="newSeed.expiry" 
              type="date"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
          </div>
          <div class="flex gap-3">
            <button 
              type="button" 
              @click="showAddForm = false"
              class="flex-1 px-4 py-2 text-gray-700 border border-gray-300 rounded-md hover:bg-gray-50 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              class="flex-1 px-4 py-2 bg-garden-green text-white rounded-md hover:bg-green-600 transition-colors"
            >
              Add Seed
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'App',
  setup() {
    const seeds = ref([])
    const showAddForm = ref(false)
    const newSeed = ref({
      name: '',
      variety: '',
      quantity: '',
      expiry: ''
    })

    const totalSeeds = computed(() => seeds.value.length)
    const lowStockCount = computed(() => seeds.value.filter(seed => seed.quantity < 10).length)

    const formatDate = (dateString) => {
      if (!dateString) return 'N/A'
      return new Date(dateString).toLocaleDateString()
    }

    const addSeed = () => {
      const seed = {
        id: Date.now(),
        name: newSeed.value.name,
        variety: newSeed.value.variety,
        quantity: parseInt(newSeed.value.quantity),
        expiry: newSeed.value.expiry,
        createdAt: new Date().toISOString()
      }
      
      seeds.value.push(seed)
      
      // Reset form
      newSeed.value = {
        name: '',
        variety: '',
        quantity: '',
        expiry: ''
      }
      showAddForm.value = false
      
      // Save to localStorage for now
      localStorage.setItem('seeds', JSON.stringify(seeds.value))
    }

    const plantSeed = (seed) => {
      // For now, just decrease quantity
      seed.quantity = Math.max(0, seed.quantity - 1)
      localStorage.setItem('seeds', JSON.stringify(seeds.value))
    }

    // Load seeds from localStorage on mount
    onMounted(() => {
      const stored = localStorage.getItem('seeds')
      if (stored) {
        seeds.value = JSON.parse(stored)
      }
    })

    return {
      seeds,
      showAddForm,
      newSeed,
      totalSeeds,
      lowStockCount,
      formatDate,
      addSeed,
      plantSeed
    }
  }
}
</script>