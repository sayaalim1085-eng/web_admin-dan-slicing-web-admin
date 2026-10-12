<script setup>
import Sidebar from './components/Sidebar.vue'
import Navbar from './components/Navbar.vue'
</script>

<template>
  <div class="flex min-h-screen bg-slate-50">
    <!-- Sidebar -->
    <Sidebar />

    <!-- Main Content Area -->
    <div class="flex-1 flex flex-col">
      <Navbar />
      
      <main class="p-6 flex-1">
        <router-view />
      </main>
    </div>
  </div>
</template>