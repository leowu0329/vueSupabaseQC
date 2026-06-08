<template>
  <div class="min-h-screen bg-gray-50">
    <NavBar v-if="authStore.user" />
    <div class="flex">
      <!-- 側邊欄：僅在登入時顯示 -->
      <SideBar 
        v-if="authStore.user" 
        v-model:collapsed="isSidebarCollapsed"
      />
      
      <!-- 主內容區域：如果是登入狀態，則增加左邊距以避開固定定位的側邊欄 -->
      <main :class="['flex-1 transition-all duration-300', authStore.user ? (isSidebarCollapsed ? 'pl-20' : 'pl-64') : '']">
        <router-view />
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useAuthStore } from '@/stores/auth'
import NavBar from '@/components/NavBar.vue'
import SideBar from '@/components/SideBar.vue'

const authStore = useAuthStore()
const isSidebarCollapsed = ref(false)

onMounted(async () => {
  await authStore.fetchUser()
})
</script>
