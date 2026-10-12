<script setup>
import { ref } from 'vue'

// State untuk toggle dropdown notifikasi
const showNotifications = ref(false)

// Data dummy notifikasi masuk
const notifications = ref([
  { id: 1, title: 'User Baru Terdaftar', message: 'Andi Wijaya baru saja mendaftar.', time: '2 menit yang lalu', unread: true },
  { id: 2, title: 'Laporan Masuk', message: 'Laporan mingguan telah diperbarui.', time: '1 jam yang lalu', unread: true },
  { id: 3, title: 'Sistem Update', message: 'Pembaruan sistem versi 1.2 sukses.', time: '1 hari yang lalu', unread: false }
])

// Toggle buka/tutup dropdown
const toggleNotifications = () => {
  showNotifications.value = !showNotifications.value
}

// Tandai semua notifikasi sudah dibaca
const markAllAsRead = () => {
  notifications.value.forEach(n => n.unread = false)
}
</script>

<template>
  <header class="bg-white border-b border-gray-200 px-6 py-4 flex justify-between items-center relative">
    <!-- Search / Title Ringkasan -->
    <div class="text-sm font-medium text-gray-500">
      Dashboard Admin
    </div>

    <!-- Area Kanan (Notifikasi & Profil) -->
    <div class="flex items-center gap-4 relative">
      <!-- Tombol Notifikasi -->
      <div class="relative">
        <button 
          @click="toggleNotifications"
          class="relative p-2 text-gray-600 hover:text-gray-900 hover:bg-gray-100 rounded-full transition-colors cursor-pointer"
        >
          <!-- Icon Lonceng Notifikasi -->
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
          </svg>

          <!-- Badge Indikator Belum Dibaca -->
          <span 
            v-if="notifications.some(n => n.unread)" 
            class="absolute top-1 right-1 w-2.5 h-2.5 bg-red-500 rounded-full border-2 border-white"
          ></span>
        </button>

        <!-- Dropdown Daftar Notifikasi -->
        <div 
          v-if="showNotifications"
          class="absolute right-0 mt-2 w-80 bg-white rounded-xl shadow-xl border border-gray-200 z-50 overflow-hidden"
        >
          <div class="p-4 border-b border-gray-100 flex justify-between items-center">
            <h3 class="font-bold text-gray-800 text-sm">Notifikasi</h3>
            <button 
              @click="markAllAsRead" 
              class="text-xs text-blue-600 hover:underline cursor-pointer"
            >
              Tandai dibaca
            </button>
          </div>

          <!-- Daftar Pesan Notifikasi -->
          <div class="max-h-64 overflow-y-auto divide-y divide-gray-100">
            <div 
              v-for="notif in notifications" 
              :key="notif.id"
              :class="['p-3 text-sm hover:bg-slate-50 transition-colors cursor-pointer', notif.unread ? 'bg-blue-50/50' : '']"
            >
              <div class="flex justify-between items-start mb-1">
                <span class="font-semibold text-gray-800 text-xs">{{ notif.title }}</span>
                <span class="text-[10px] text-gray-400">{{ notif.time }}</span>
              </div>
              <p class="text-xs text-gray-600 line-clamp-2">{{ notif.message }}</p>
            </div>

            <div v-if="notifications.length === 0" class="p-4 text-center text-xs text-gray-400">
              Tidak ada notifikasi.
            </div>
          </div>

          <div class="p-2 border-t border-gray-100 text-center bg-slate-50">
            <a href="#" class="text-xs text-blue-600 font-medium hover:underline">Lihat Semua Notifikasi</a>
          </div>
        </div>
      </div>

      <!-- Profil Avatar -->
      <div class="flex items-center gap-2 border-l border-gray-200 pl-4">
        <div class="w-8 h-8 rounded-full bg-blue-600 text-white flex items-center justify-center font-bold text-sm">
          A
        </div>
        <span class="text-sm font-medium text-gray-700 hidden sm:inline">Admin</span>
      </div>
    </div>
  </header>
</template>