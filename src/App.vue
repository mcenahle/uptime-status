<template>
  <div class="min-h-screen flex flex-col bg-gradient-to-br from-gray-50 via-gray-50 to-gray-100 
    dark:from-gray-900 dark:via-gray-900 dark:to-gray-800 transition-colors duration-300">
    <div class="flex-1 p-3 sm:p-8">
      <main class="max-w-7xl mx-auto space-y-8">
        <Header 
          ref="headerRef"
          :title="title"
          :is-refreshing="isRefreshing"
          :is-dark="isDark"
          @refresh="refreshData"
          @toggle-theme="toggleTheme"
        />
        <p><b>此处的延迟、响应时间数据可能不准确。</b>因为本站使用的是 <a href="https://www.uptimerobot.com" target="_blank"><u>uptimerobot</u></a> 提供的服务（位于境外），所以数值偏大。</p>
        <Stats :monitors="monitors" />
        <Card 
          :monitors="monitors"
          :error="errorMessage"
        />
      </main>
    </div>
    <Footer />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { fetchMonitorData } from './utils/api'
import Header from './components/Header.vue'
import Stats from './components/Stats.vue'
import Card from './components/Card.vue'
import Footer from './components/Footer.vue'

const title = ref(import.meta.env.VITE_APP_TITLE || '状态监控')
const monitors = ref([])
const isRefreshing = ref(false)
const isDark = ref(false)
const errorMessage = ref('')

// 简化主题相关逻辑
const initTheme = () => {
  isDark.value = localStorage.getItem('theme') === 'dark' || 
    (!localStorage.getItem('theme') && window.matchMedia('(prefers-color-scheme: dark)').matches)
  updateTheme()
}

const updateTheme = () => document.documentElement.classList.toggle('dark', isDark.value)

const toggleTheme = () => {
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
  updateTheme()
}

// 简化刷新逻辑
const refreshData = async () => {
  if (isRefreshing.value) return
  
  isRefreshing.value = true
  monitors.value = []
  errorMessage.value = ''
  
  const timeoutPromise = new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 30000)
  )

  try {
    monitors.value = await Promise.race([
      fetchMonitorData(),
      timeoutPromise
    ])
  } catch (error) {
    console.error('获取监控数据失败:', error)
    errorMessage.value = error.message === 'Timeout' 
      ? '请求超时，请检查网络连接后重试'
      : '获取监控数据失败，请稍后重试'
  } finally {
    isRefreshing.value = false
  }
}

// 生命周期钩子
onMounted(() => {
  initTheme()
  refreshData()
})
</script>
