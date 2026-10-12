<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// State untuk ringkasan dan daftar aktivitas
const users = ref([])
const totalUser = ref(0)
const userAktif = ref(0)
const administratorCount = ref(0)

// Helper Header Token
const getAuthHeader = () => {
  const token = localStorage.getItem('token')
  return { headers: { Authorization: `Bearer ${token}` } }
}

// Ambil data user dari API Laravel
const fetchDashboardData = async () => {
  try {
    const response = await axios.get('http://127.0.0.1:8000/api/users', getAuthHeader())
    const allUsers = response.data.data
    
    users.value = allUsers
    totalUser.value = allUsers.length
    
    // Hitung statistik otomatis
    userAktif.value = allUsers.filter(u => (u.status || 'aktif').toLowerCase() === 'aktif').length
    administratorCount.value = allUsers.filter(u => (u.role || '').toLowerCase() === 'admin').length
  } catch (error) {
    console.error('Gagal memuat data dashboard:', error)
  }
}

onMounted(() => {
  fetchDashboardData()
})
</script>

<template>
  <div>
    <!-- Kartu Statistik Atas -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
      <div class="bg-white p-4 rounded-xl border border-gray-200 shadow-sm flex justify-between items-center">
        <div>
          <p class="text-xs font-semibold text-gray-400 uppercase">Total User</p>
          <h3 class="text-2xl font-bold text-gray-800 mt-1">{{ totalUser }}</h3>
        </div>
        <div class="bg-blue-100 text-blue-600 p-3 rounded-lg text-lg">👥</div>
      </div>

      <div class="bg-white p-4 rounded-xl border border-gray-200 shadow-sm flex justify-between items-center">
        <div>
          <p class="text-xs font-semibold text-gray-400 uppercase">User Aktif</p>
          <h3 class="text-2xl font-bold text-gray-800 mt-1">{{ userAktif }}</h3>
        </div>
        <div class="bg-green-100 text-green-600 p-3 rounded-lg text-lg">✅</div>
      </div>

      <div class="bg-white p-4 rounded-xl border border-gray-200 shadow-sm flex justify-between items-center">
        <div>
          <p class="text-xs font-semibold text-gray-400 uppercase">Administrator</p>
          <h3 class="text-2xl font-bold text-gray-800 mt-1">{{ administratorCount }}</h3>
        </div>
        <div class="bg-purple-100 text-purple-600 p-3 rounded-lg text-lg">🔑</div>
      </div>

      <div class="bg-white p-4 rounded-xl border border-gray-200 shadow-sm flex justify-between items-center">
        <div>
          <p class="text-xs font-semibold text-gray-400 uppercase">Pending Support</p>
          <h3 class="text-2xl font-bold text-gray-800 mt-1">0</h3>
        </div>
        <div class="bg-orange-100 text-orange-600 p-3 rounded-lg text-lg">📥</div>
      </div>
    </div>

    <!-- Bagian Aktivitas Terbaru -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-lg font-bold text-gray-800">Aktivitas Terbaru</h2>
      </div>

      <div class="divide-y divide-gray-100">
        <!-- Loop data user terbaru dari database -->
        <div v-for="user in users.slice(0, 5)" :key="user.id" class="py-3 flex justify-between items-center">
          <div>
            <p class="text-sm font-semibold text-gray-900">{{ user.name }}</p>
            <p class="text-xs text-gray-500">Terdaftar dengan email {{ user.email }}</p>
          </div>
          <span class="text-xs bg-slate-100 text-slate-600 px-2.5 py-1 rounded-full font-medium capitalize">
            {{ user.role || 'User' }}
          </span>
        </div>

        <div v-if="users.length === 0" class="py-4 text-center text-gray-400 text-sm">
          Belum ada aktivitas terbaru.
        </div>
      </div>
    </div>
  </div>
</template>