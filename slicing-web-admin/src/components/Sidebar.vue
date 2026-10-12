<template>
  <aside class="w-64 bg-slate-900 text-white min-h-screen p-4 flex flex-col justify-between">
    <div>
      <!-- Logo / Title -->
      <div class="text-xl font-bold p-2 mb-6 tracking-wide border-b border-slate-700">
        Web Admin
      </div>

      <!-- Menu Items -->
      <nav class="space-y-2">
        <router-link 
          to="/" 
          class="flex items-center gap-3 p-3 rounded-lg hover:bg-slate-800 transition-colors text-slate-300 hover:text-white"
        >
          <span>Dashboard</span>
        </router-link>
        
        <router-link 
          to="/users" 
          class="flex items-center gap-3 p-3 rounded-lg hover:bg-slate-800 transition-colors text-slate-300 hover:text-white"
        >
          <span>Manajemen User</span>
        </router-link>
      </nav>
    </div>

    <!-- User Info / Footer Sidebar -->
    <div class="p-3 border-t border-slate-800 text-sm text-slate-400">
      <p class="font-medium text-white">Admin System</p>
      <p class="text-xs">admin@example.com</p>
    </div>
  </aside>
</template>