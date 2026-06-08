<template>
  <aside
    :class="[
      'fixed left-0 top-16 h-[calc(100vh-64px)] bg-white border-r border-gray-200 transition-[width] duration-300 ease-in-out z-30 shadow-sm flex flex-col overflow-hidden',
      collapsed ? 'w-20' : 'w-64'
    ]"
  >
    <!-- 側邊欄頂部：收合開關 -->
    <div class="relative flex items-center h-16 border-b border-gray-50 flex-shrink-0">
      <button
        @click="$emit('update:collapsed', !collapsed)"
        class="absolute right-5 p-1.5 rounded-lg bg-gray-50 text-gray-500 hover:bg-indigo-50 hover:text-indigo-600 focus:outline-none transition-colors"
        title="切換選單狀態"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-6 w-6 transition-transform duration-300"
          :class="{ 'rotate-180': collapsed }"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7" />
        </svg>
      </button>
    </div>

    <!-- 功能選項區 -->
    <nav class="flex-1 mt-4 px-4 space-y-2">
      <router-link
        to="/ipqc"
        class="flex items-center h-12 px-3 text-gray-600 rounded-xl hover:bg-indigo-50 hover:text-indigo-600 transition-all group"
        active-class="bg-indigo-50 text-indigo-700 font-semibold"
      >
        <div class="w-6 flex-shrink-0 flex items-center justify-center">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
          </svg>
        </div>
        <span
          :class="['ml-4 whitespace-nowrap transition-opacity duration-300', collapsed ? 'opacity-0' : 'opacity-100']"
        >
          IPQC 檢驗系統
        </span>
      </router-link>
    </nav>

    <!-- 底部版本資訊 (僅展開時顯示) -->
    <div :class="['p-4 border-t border-gray-100 text-center transition-opacity duration-300 flex-shrink-0', collapsed ? 'opacity-0' : 'opacity-100']">
      <span class="text-xs text-gray-400 whitespace-nowrap">v1.0.0</span>
    </div>
  </aside>
</template>

<script setup>
defineProps({
  collapsed: Boolean
})
defineEmits(['update:collapsed'])
</script>