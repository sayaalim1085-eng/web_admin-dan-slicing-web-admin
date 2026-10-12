<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

// State Data Users dari API
const users = ref([])

// Helper Header Token
const getAuthHeader = () => {
  const token = localStorage.getItem('token')
  return { headers: { Authorization: `Bearer ${token}` } }
}

// 1. GET DATA (Fetch Users)
const fetchUsers = async () => {
  try {
    const response = await axios.get('http://127.0.0.1:8000/api/users', getAuthHeader())
    users.value = response.data.data
  } catch (error) {
    console.error('Gagal mengambil data users:', error)
  }
}

onMounted(() => {
  fetchUsers()
})

// State Pencarian & Filter
const searchQuery = ref('')
const filterRole = ref('')

// State Paginasi
const currentPage = ref(1)
const itemsPerPage = 4

// State Modal
const showModal = ref(false)
const isEditMode = ref(false)
const selectedUserId = ref(null)

const form = ref({
  name: '',
  email: '',
  password: 'password123',
  role: 'user',
  status: 'aktif'
})

// Filter Data berdasarkan Search & Dropdown Role
const filteredUsers = computed(() => {
  return users.value.filter(user => {
    const matchSearch = (user.name || '').toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                        (user.email || '').toLowerCase().includes(searchQuery.value.toLowerCase())
    const userRole = (user.role || '').toLowerCase()
    const matchRole = filterRole.value ? userRole === filterRole.value.toLowerCase() : true
    
    return matchSearch && matchRole
  })
})

// Paginasi Data
const totalPages = computed(() => Math.ceil(filteredUsers.value.length / itemsPerPage) || 1)
const paginatedUsers = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredUsers.value.slice(start, start + itemsPerPage)
})

// Modal Actions
const openAddModal = () => {
  isEditMode.value = false
  selectedUserId.value = null
  form.value = { name: '', email: '', password: 'password123', role: 'user', status: 'aktif' }
  showModal.value = true
}

const openEditModal = (user) => {
  isEditMode.value = true
  selectedUserId.value = user.id
  form.value = { 
    name: user.name,
    email: user.email,
    role: user.role || 'user',
    status: user.status || 'aktif'
  }
  showModal.value = true
}

// 2. POST / PUT DATA (Tambah & Edit User via API)
const simpanUser = async () => {
  if (!form.value.name || !form.value.email) {
    alert('Nama dan Email wajib diisi!')
    return
  }

  try {
    if (isEditMode.value) {
      // PUT request ke Laravel
      await axios.put(`http://127.0.0.1:8000/api/users/${selectedUserId.value}`, form.value, getAuthHeader())
    } else {
      const payload = {
        ...form.value,
        password: 'password123',
        password_confirmation: 'password123'
      }
      // PERBAIKAN: Mengirimkan payload yang berisi password
      await axios.post('http://127.0.0.1:8000/api/users', payload, getAuthHeader())
    }
    
    // Refresh data dari server
    await fetchUsers()
    showModal.value = false
  } catch (error) {
    console.error('Gagal menyimpan data user:', error.response?.data || error)
    alert('Terjadi kesalahan saat menyimpan data.')
  }
}

// 3. DELETE DATA (Hapus User via API)
const hapusUser = async (id) => {
  if (confirm('Apakah Anda yakin ingin menghapus user ini?')) {
    try {
      await axios.delete(`http://127.0.0.1:8000/api/users/${id}`, getAuthHeader())
      await fetchUsers()
    } catch (error) {
      console.error('Gagal menghapus user:', error)
      alert('Terjadi kesalahan saat menghapus data.')
    }
  }
}
</script>

<template>
  <div>
    <!-- Header Halaman -->
    <div class="flex justify-between items-center mb-6">
      <div>
        <h1 class="text-2xl font-bold text-gray-800">Manajemen User</h1>
        <p class="text-gray-500 text-sm">Kelola daftar pengguna sistem di sini.</p>
      </div>
      <button 
        @click="openAddModal"
        class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg text-sm font-medium transition-colors cursor-pointer"
      >
        + Tambah User
      </button>
    </div>

    <!-- Bar Pencarian & Filter -->
    <div class="bg-white p-4 rounded-xl border border-gray-200 mb-4 flex flex-col md:flex-row gap-4 justify-between">
      <input 
        v-model="searchQuery"
        type="text" 
        placeholder="Cari nama atau email..."
        class="border border-gray-300 rounded-lg px-3 py-2 text-sm w-full md:w-64 focus:ring-2 focus:ring-blue-500 focus:outline-none"
      />
      <select 
        v-model="filterRole"
        class="border border-gray-300 rounded-lg px-3 py-2 text-sm w-full md:w-48 focus:ring-2 focus:ring-blue-500 focus:outline-none"
      >
        <option value="">Semua Role</option>
        <option value="admin">Administrator</option>
        <option value="user">Staff / User</option>
      </select>
    </div>

    <!-- Tabel Data User -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
      <table class="w-full text-left border-collapse">
        <thead class="bg-slate-50 border-b border-gray-200 text-xs text-gray-500 uppercase tracking-wider">
          <tr>
            <th class="p-4">Nama</th>
            <th class="p-4">Email</th>
            <th class="p-4">Role</th>
            <th class="p-4">Status</th>
            <th class="p-4 text-center">Aksi</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100 text-sm text-gray-700">
          <tr v-for="user in paginatedUsers" :key="user.id" class="hover:bg-slate-50 transition-colors">
            <td class="p-4 font-medium text-gray-900">{{ user.name }}</td>
            <td class="p-4 text-gray-500">{{ user.email }}</td>
            <td class="p-4">
              <!-- PERBAIKAN: Pengecekan role dinamis & flexible (case insensitive) -->
              <span 
                :class="user.role?.toLowerCase() === 'admin' ? 'bg-blue-100 text-blue-700' : 'bg-gray-100 text-gray-700'"
                class="px-2.5 py-1 rounded-full text-xs font-semibold capitalize"
              >
                {{ user.role || 'User' }}
              </span>
            </td>
            <td class="p-4">
              <!-- PERBAIKAN: Tampilan status dinamis & penanganan nilai default -->
              <span 
                :class="(user.status?.toLowerCase() === 'aktif' || !user.status) ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'"
                class="px-2.5 py-1 rounded-full text-xs font-semibold capitalize"
              >
                {{ user.status || 'Aktif' }}
              </span>
            </td>
            <td class="p-4 text-center space-x-2">
              <button @click="openEditModal(user)" class="text-blue-600 hover:underline cursor-pointer">Edit</button>
              <button @click="hapusUser(user.id)" class="text-red-600 hover:underline cursor-pointer">Hapus</button>
            </td>
          </tr>
          <tr v-if="paginatedUsers.length === 0">
            <td colspan="5" class="p-4 text-center text-gray-400">Data tidak ditemukan.</td>
          </tr>
        </tbody>
      </table>

      <!-- Navigasi Paginasi -->
      <div class="p-4 border-t border-gray-200 flex justify-between items-center text-sm text-gray-600">
        <span>Halaman {{ currentPage }} dari {{ totalPages }}</span>
        <div class="flex gap-2">
          <button 
            :disabled="currentPage === 1"
            @click="currentPage--"
            class="px-3 py-1 border rounded hover:bg-gray-50 disabled:opacity-50 cursor-pointer"
          >
            Sebelumnya
          </button>
          <button 
            :disabled="currentPage === totalPages"
            @click="currentPage++"
            class="px-3 py-1 border rounded hover:bg-gray-50 disabled:opacity-50 cursor-pointer"
          >
            Selanjutnya
          </button>
        </div>
      </div>
    </div>

    <!-- Modal Form (Tambah / Edit User) -->
    <div v-if="showModal" class="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-xl max-w-md w-full p-6 shadow-2xl">
        <h2 class="text-xl font-bold text-gray-800 mb-4">
          {{ isEditMode ? 'Edit User' : 'Tambah User Baru' }}
        </h2>
        
        <form @submit.prevent="simpanUser" class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Nama Lengkap</label>
            <input 
              v-model="form.name" 
              type="text" 
              placeholder="Masukkan nama"
              class="w-full border border-gray-300 rounded-lg p-2.5 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Email</label>
            <input 
              v-model="form.email" 
              type="email" 
              placeholder="nama@example.com"
              class="w-full border border-gray-300 rounded-lg p-2.5 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
            />
          </div>

          <div class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">Role</label>
              <select 
                v-model="form.role" 
                class="w-full border border-gray-300 rounded-lg p-2.5 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
              >
                <!-- PERBAIKAN: Value option disesuaikan dengan nilai backend -->
                <option value="user">Staff / User</option>
                <option value="admin">Administrator</option>
              </select>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">Status</label>
              <select 
                v-model="form.status" 
                class="w-full border border-gray-300 rounded-lg p-2.5 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
              >
                <!-- PERBAIKAN: Value option disesuaikan dengan nilai backend -->
                <option value="aktif">Aktif</option>
                <option value="non-aktif">Non-Aktif</option>
              </select>
            </div>
          </div>

          <div class="flex justify-end gap-3 mt-6">
            <button 
              type="button" 
              @click="showModal = false"
              class="px-4 py-2 border border-gray-300 rounded-lg text-sm text-gray-700 hover:bg-gray-50 cursor-pointer"
            >
              Batal
            </button>
            <button 
              type="submit" 
              class="px-4 py-2 bg-blue-600 text-white rounded-lg text-sm font-medium hover:bg-blue-700 cursor-pointer"
            >
              {{ isEditMode ? 'Update' : 'Simpan' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>