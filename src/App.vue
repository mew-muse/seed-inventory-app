<template>
  <div class="min-h-screen bg-gradient-to-br from-green-50 to-emerald-50">
    <!-- Header -->
    <header class="bg-white shadow-sm px-4 py-6">
      <div class="max-w-md mx-auto">
        <h1 class="text-2xl font-bold text-garden-green flex items-center gap-2">
          🌱 Seed Inventory
        </h1>
        <p class="text-gray-600 text-sm mt-1">{{ getPageSubtitle() }}</p>
      </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-md mx-auto px-4 py-6 pb-20">
      <!-- Dashboard View -->
      <div v-if="currentView === 'dashboard'">
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

        <!-- Recent Activity -->
        <div class="bg-white rounded-lg p-4 shadow-sm mb-6">
          <h3 class="font-semibold text-gray-800 mb-3">Recent Activity</h3>
          <div v-if="recentPlantings.length === 0" class="text-center py-4">
            <div class="text-2xl mb-2">🌱</div>
            <p class="text-gray-500 text-sm">No recent plantings</p>
          </div>
          <div v-else class="space-y-2">
            <div v-for="planting in recentPlantings.slice(0, 3)" :key="planting.id" class="flex justify-between items-center">
              <div class="text-sm">
                <span class="font-medium">{{ planting.seedName }}</span>
                <span class="text-gray-500"> - {{ planting.quantity }} planted</span>
              </div>
              <div class="text-xs text-gray-500">{{ formatDate(planting.date) }}</div>
            </div>
          </div>
        </div>

        <!-- Quick Actions -->
        <div class="grid grid-cols-2 gap-4">
          <button 
            @click="showAddForm = true"
            class="bg-garden-green text-white py-4 px-6 rounded-lg font-semibold shadow-md hover:bg-green-600 transition-colors flex items-center justify-center gap-2"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"/>
            </svg>
            Add Seed
          </button>
          <button 
            @click="currentView = 'inventory'"
            class="bg-blue-500 text-white py-4 px-6 rounded-lg font-semibold shadow-md hover:bg-blue-600 transition-colors flex items-center justify-center gap-2"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"/>
            </svg>
            Inventory
          </button>
        </div>
      </div>

      <!-- Inventory View -->
      <div v-if="currentView === 'inventory'">
        <div class="mb-6">
          <button 
            @click="showAddForm = true"
            class="w-full bg-garden-green text-white py-4 px-6 rounded-lg font-semibold text-lg shadow-md hover:bg-green-600 transition-colors flex items-center justify-center gap-2"
          >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"/>
            </svg>
            Add New Seed
          </button>
        </div>

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
            <div class="mb-3">
              <div class="text-sm text-gray-600 mb-1">
                Qty: {{ seed.quantity }} • Exp: {{ formatDate(seed.expiry) }}
              </div>
              <div v-if="seed.plantings && seed.plantings.length > 0" class="text-xs text-gray-500">
                Last planted: {{ formatDate(seed.plantings[seed.plantings.length - 1].date) }}
              </div>
            </div>
            <div class="flex gap-2">
              <button 
                @click="openPlantingModal(seed)"
                class="flex-1 bg-garden-green text-white px-3 py-2 rounded text-sm hover:bg-green-600 transition-colors"
              >
                Plant
              </button>
              <button 
                @click="viewPlantingHistory(seed)"
                class="px-3 py-2 border border-gray-300 rounded text-sm hover:bg-gray-50 transition-colors"
              >
                History
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Planting History View -->
      <div v-if="currentView === 'plantings'">
        <div v-if="allPlantings.length === 0" class="text-center py-12">
          <div class="text-6xl mb-4">🌱</div>
          <h3 class="text-lg font-medium text-gray-700 mb-2">No plantings yet!</h3>
          <p class="text-gray-500">Plant some seeds to see your history</p>
        </div>
        
        <div v-else class="space-y-4">
          <div v-for="planting in allPlantings" :key="planting.id" class="bg-white rounded-lg p-4 shadow-sm">
            <div class="flex justify-between items-start mb-2">
              <h3 class="font-semibold text-gray-800">{{ planting.seedName }}</h3>
              <span class="text-sm text-gray-500">{{ formatDate(planting.date) }}</span>
            </div>
            <div class="text-sm text-gray-600 mb-1">
              Quantity: {{ planting.quantity }}
            </div>
            <div v-if="planting.location" class="text-sm text-gray-600 mb-1">
              📍 {{ planting.location }}
            </div>
            <div v-if="planting.notes" class="text-sm text-gray-700">
              {{ planting.notes }}
            </div>
          </div>
        </div>
      </div>

      <!-- Analytics View -->
      <div v-if="currentView === 'analytics'">
        <div class="space-y-6">
          <!-- Planting Stats -->
          <div class="bg-white rounded-lg p-4 shadow-sm">
            <h3 class="font-semibold text-gray-800 mb-3">Planting Statistics</h3>
            <div class="grid grid-cols-2 gap-4">
              <div class="text-center">
                <div class="text-2xl font-bold text-garden-green">{{ totalPlantings }}</div>
                <div class="text-sm text-gray-600">Total Plantings</div>
              </div>
              <div class="text-center">
                <div class="text-2xl font-bold text-blue-500">{{ uniqueLocations }}</div>
                <div class="text-sm text-gray-600">Locations Used</div>
              </div>
            </div>
          </div>

          <!-- Most Planted Seeds -->
          <div class="bg-white rounded-lg p-4 shadow-sm">
            <h3 class="font-semibold text-gray-800 mb-3">Most Planted Seeds</h3>
            <div v-if="mostPlantedSeeds.length === 0" class="text-center py-4">
              <p class="text-gray-500 text-sm">No planting data yet</p>
            </div>
            <div v-else class="space-y-2">
              <div v-for="seed in mostPlantedSeeds.slice(0, 5)" :key="seed.name" class="flex justify-between items-center">
                <span class="text-sm font-medium">{{ seed.name }}</span>
                <span class="text-sm text-gray-500">{{ seed.count }} times</span>
              </div>
            </div>
          </div>

          <!-- Planting Timeline -->
          <div class="bg-white rounded-lg p-4 shadow-sm">
            <h3 class="font-semibold text-gray-800 mb-3">Recent Planting Timeline</h3>
            <div v-if="recentPlantings.length === 0" class="text-center py-4">
              <p class="text-gray-500 text-sm">No recent plantings</p>
            </div>
            <div v-else class="space-y-3">
              <div v-for="planting in recentPlantings.slice(0, 8)" :key="planting.id" class="flex items-center gap-3">
                <div class="w-2 h-2 bg-garden-green rounded-full"></div>
                <div class="flex-1">
                  <div class="text-sm font-medium">{{ planting.seedName }}</div>
                  <div class="text-xs text-gray-500">{{ formatDate(planting.date) }} - {{ planting.quantity }} planted</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Bottom Navigation -->
    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 px-4 py-2">
      <div class="max-w-md mx-auto">
        <div class="flex justify-around">
          <button 
            @click="currentView = 'dashboard'"
            :class="[
              'flex flex-col items-center gap-1 py-2 px-3 rounded-lg transition-colors',
              currentView === 'dashboard' ? 'bg-green-50 text-garden-green' : 'text-gray-600 hover:text-gray-800'
            ]"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/>
            </svg>
            <span class="text-xs">Dashboard</span>
          </button>
          
          <button 
            @click="currentView = 'inventory'"
            :class="[
              'flex flex-col items-center gap-1 py-2 px-3 rounded-lg transition-colors',
              currentView === 'inventory' ? 'bg-green-50 text-garden-green' : 'text-gray-600 hover:text-gray-800'
            ]"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"/>
            </svg>
            <span class="text-xs">Inventory</span>
          </button>
          
          <button 
            @click="currentView = 'plantings'"
            :class="[
              'flex flex-col items-center gap-1 py-2 px-3 rounded-lg transition-colors',
              currentView === 'plantings' ? 'bg-green-50 text-garden-green' : 'text-gray-600 hover:text-gray-800'
            ]"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"/>
            </svg>
            <span class="text-xs">Plantings</span>
          </button>
          
          <button 
            @click="currentView = 'analytics'"
            :class="[
              'flex flex-col items-center gap-1 py-2 px-3 rounded-lg transition-colors',
              currentView === 'analytics' ? 'bg-green-50 text-garden-green' : 'text-gray-600 hover:text-gray-800'
            ]"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"/>
            </svg>
            <span class="text-xs">Analytics</span>
          </button>
        </div>
      </div>
    </nav>

    <!-- Plant Seed Modal -->
    <div v-if="showPlantingModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-md">
        <h2 class="text-xl font-bold mb-4">Plant {{ selectedSeed?.name }}</h2>
        <form @submit.prevent="recordPlanting">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">Planting Date</label>
            <input 
              v-model="plantingData.date" 
              type="date" 
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
          </div>
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">Quantity to Plant</label>
            <input 
              v-model="plantingData.quantity" 
              type="number" 
              min="1" 
              :max="selectedSeed?.quantity || 1"
              required
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
            <div class="text-xs text-gray-500 mt-1">Available: {{ selectedSeed?.quantity || 0 }}</div>
          </div>
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-1">Location (Optional)</label>
            <input 
              v-model="plantingData.location" 
              type="text" 
              placeholder="e.g., Garden bed A, Greenhouse, Window box"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            >
          </div>
          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-1">Notes (Optional)</label>
            <textarea 
              v-model="plantingData.notes" 
              rows="3" 
              placeholder="e.g., Germination rate, soil conditions, weather..."
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-garden-green"
            ></textarea>
          </div>
          <div class="flex gap-3">
            <button 
              type="button" 
              @click="closePlantingModal"
              class="flex-1 px-4 py-2 text-gray-700 border border-gray-300 rounded-md hover:bg-gray-50 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              class="flex-1 px-4 py-2 bg-garden-green text-white rounded-md hover:bg-green-600 transition-colors"
            >
              Record Planting
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Planting History Modal -->
    <div v-if="showHistoryModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-md max-h-96 overflow-y-auto">
        <h2 class="text-xl font-bold mb-4">{{ selectedSeed?.name }} - Planting History</h2>
        <div v-if="!selectedSeed?.plantings || selectedSeed.plantings.length === 0" class="text-center py-8">
          <div class="text-4xl mb-2">🌱</div>
          <p class="text-gray-500">No plantings recorded yet</p>
        </div>
        <div v-else class="space-y-4">
          <div v-for="planting in selectedSeed.plantings" :key="planting.id" class="border border-gray-200 rounded-lg p-3">
            <div class="flex justify-between items-start mb-2">
              <div class="font-medium">{{ formatDate(planting.date) }}</div>
              <div class="text-sm text-gray-500">Qty: {{ planting.quantity }}</div>
            </div>
            <div v-if="planting.location" class="text-sm text-gray-600 mb-1">
              📍 {{ planting.location }}
            </div>
            <div v-if="planting.notes" class="text-sm text-gray-700">
              {{ planting.notes }}
            </div>
          </div>
        </div>
        <div class="mt-6">
          <button 
            @click="closeHistoryModal"
            class="w-full px-4 py-2 bg-gray-100 text-gray-700 rounded-md hover:bg-gray-200 transition-colors"
          >
            Close
          </button>
        </div>
      </div>
    </div>

    <!-- Add Seed Modal -->
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
    const showPlantingModal = ref(false)
    const showHistoryModal = ref(false)
    const selectedSeed = ref(null)
    const currentView = ref('dashboard')
    const newSeed = ref({
      name: '',
      variety: '',
      quantity: '',
      expiry: ''
    })
    const plantingData = ref({
      date: new Date().toISOString().split('T')[0],
      quantity: 1,
      location: '',
      notes: ''
    })

    const totalSeeds = computed(() => seeds.value.length)
    const lowStockCount = computed(() => seeds.value.filter(seed => seed.quantity < 10).length)
    
    const allPlantings = computed(() => {
      const plantings = []
      seeds.value.forEach(seed => {
        if (seed.plantings) {
          seed.plantings.forEach(planting => {
            plantings.push({
              ...planting,
              seedName: seed.name
            })
          })
        }
      })
      return plantings.sort((a, b) => new Date(b.date) - new Date(a.date))
    })
    
    const recentPlantings = computed(() => allPlantings.value.slice(0, 10))
    
    const totalPlantings = computed(() => allPlantings.value.length)
    
    const uniqueLocations = computed(() => {
      const locations = new Set()
      allPlantings.value.forEach(planting => {
        if (planting.location) {
          locations.add(planting.location)
        }
      })
      return locations.size
    })
    
    const mostPlantedSeeds = computed(() => {
      const counts = {}
      allPlantings.value.forEach(planting => {
        counts[planting.seedName] = (counts[planting.seedName] || 0) + 1
      })
      
      return Object.entries(counts)
        .map(([name, count]) => ({ name, count }))
        .sort((a, b) => b.count - a.count)
    })

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
        createdAt: new Date().toISOString(),
        plantings: []
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

    const openPlantingModal = (seed) => {
      selectedSeed.value = seed
      plantingData.value = {
        date: new Date().toISOString().split('T')[0],
        quantity: 1,
        location: '',
        notes: ''
      }
      showPlantingModal.value = true
    }

    const closePlantingModal = () => {
      showPlantingModal.value = false
      selectedSeed.value = null
    }

    const recordPlanting = () => {
      if (!selectedSeed.value) return
      
      const planting = {
        id: Date.now(),
        date: plantingData.value.date,
        quantity: parseInt(plantingData.value.quantity),
        location: plantingData.value.location,
        notes: plantingData.value.notes,
        createdAt: new Date().toISOString()
      }
      
      // Initialize plantings array if it doesn't exist
      if (!selectedSeed.value.plantings) {
        selectedSeed.value.plantings = []
      }
      
      selectedSeed.value.plantings.push(planting)
      selectedSeed.value.quantity = Math.max(0, selectedSeed.value.quantity - planting.quantity)
      
      localStorage.setItem('seeds', JSON.stringify(seeds.value))
      closePlantingModal()
    }

    const viewPlantingHistory = (seed) => {
      selectedSeed.value = seed
      showHistoryModal.value = true
    }

    const closeHistoryModal = () => {
      showHistoryModal.value = false
      selectedSeed.value = null
    }
    
    const getPageSubtitle = () => {
      switch (currentView.value) {
        case 'dashboard':
          return 'Track your seeds, grow your garden'
        case 'inventory':
          return 'Manage your seed collection'
        case 'plantings':
          return 'View your planting history'
        case 'analytics':
          return 'Track your gardening progress'
        default:
          return 'Track your seeds, grow your garden'
      }
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
      showPlantingModal,
      showHistoryModal,
      selectedSeed,
      newSeed,
      plantingData,
      currentView,
      totalSeeds,
      lowStockCount,
      allPlantings,
      recentPlantings,
      totalPlantings,
      uniqueLocations,
      mostPlantedSeeds,
      formatDate,
      addSeed,
      openPlantingModal,
      closePlantingModal,
      recordPlanting,
      viewPlantingHistory,
      closeHistoryModal,
      getPageSubtitle
    }
  }
}
</script>