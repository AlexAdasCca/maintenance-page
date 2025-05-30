<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

// 维护状态和配置
const status = ref<'即将' | '进行中' | '已完成'>('进行中')
const startTime = ref(new Date())
const endTime = ref(new Date())
const currentTime = ref(new Date())
const remainingTime = ref('')
const isLoading = ref(true)
const error = ref('')

// 从TextDB获取维护数据
const fetchMaintenanceData = async () => {
  try {
    if (!import.meta.env.TEXTDB_ID || !import.meta.env.MAINTENANCE_ID) {
      throw new Error('获取维护信息失败，请检查环境变量配置')
    }

    const textdbUrl = `https://textdb.online/${import.meta.env.TEXTDB_ID}`
    console.log('Fetching maintenance data from:', textdbUrl)
    const response = await fetch(textdbUrl, {
      headers: {
        'Cache-Control': 'no-cache',
        'Pragma': 'no-cache',
        'Expires': '0'
      }
    })
    if (!response.ok) throw new Error('获取维护信息失败，当前可能未设置维护信息')
    
    const text = await response.text()
    if (!text) throw new Error('获取到空响应')
    
    let allData
    try {
      allData = JSON.parse(text)
    } catch (e) {
      throw new Error('维护信息格式无效')
    }
    
    if (typeof allData !== 'object' || allData === null) {
      throw new Error('维护信息格式无效')
    }
    
    const maintenanceData = allData[import.meta.env.MAINTENANCE_ID]
    
    if (!maintenanceData) {
      throw new Error('获取维护信息失败，当前可能未设置维护信息')
    }

    status.value = maintenanceData.status
    startTime.value = new Date(maintenanceData.startTime)
    endTime.value = new Date(maintenanceData.endTime)
  } catch (err: unknown) {
    if (err instanceof Error) {
      error.value = err.message
      console.error(err)
      // 显示短暂提示
      const notification = document.createElement('div')
      notification.className = 'notification'
      notification.textContent = error.value
      document.body.appendChild(notification)
      setTimeout(() => {
        notification.remove()
      }, 3000)
    } else {
      error.value = '发生未知错误'
      console.error('Unexpected error', err)
    }
  } finally {
    isLoading.value = false
  }
}

// 更新当前时间和剩余时间
const updateTime = () => {
  currentTime.value = new Date()
  
  if (currentTime.value < startTime.value) {
    status.value = '即将'
    const diff = startTime.value.getTime() - currentTime.value.getTime()
    remainingTime.value = formatTime(diff)
  } else if (currentTime.value < endTime.value) {
    status.value = '进行中'
    const diff = endTime.value.getTime() - currentTime.value.getTime()
    remainingTime.value = formatTime(diff)
  } else {
    status.value = '已完成'
    remainingTime.value = '维护已完成'
  }
}

// 格式化时间为小时:分钟:秒
const formatTime = (ms: number) => {
  const seconds = Math.floor((ms / 1000) % 60)
  const minutes = Math.floor((ms / (1000 * 60)) % 60)
  const hours = Math.floor((ms / (1000 * 60 * 60)))
  
  return `${hours}小时 ${minutes}分钟 ${seconds}秒`
}

// 设置定时器
let timer: number
onMounted(() => {
  fetchMaintenanceData().then(() => {
    updateTime()
    timer = setInterval(updateTime, 1000)
  })
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>

<template>
  <div class="maintenance-container">
    <h1>网站维护通知</h1>
    
    <div v-if="isLoading" class="loading">
      正在加载维护信息...
    </div>
    
    <template v-if="error">
      <div class="default-maintenance">
        <h2>网站维护中</h2>
        <p>我们正在对网站进行维护升级，请稍后再访问。</p>
        <p>给您带来的不便，我们深表歉意。</p>
      </div>
    </template>
    
    <template v-else>
      <div class="status-card">
        <div class="status-badge" :class="status">
          {{ status }}维护
        </div>
        
        <div class="time-info">
          <div class="time-row">
            <span>开始时间:</span>
            <span>{{ startTime.toLocaleString() }}</span>
          </div>
          <div class="time-row">
            <span>结束时间:</span>
            <span>{{ endTime.toLocaleString() }}</span>
          </div>
          <div class="time-row">
            <span>当前时间:</span>
            <span>{{ currentTime.toLocaleString() }}</span>
          </div>
          <div class="time-row">
            <span>剩余时间:</span>
            <span>{{ remainingTime }}</span>
          </div>
        </div>
      </div>
      
      <p class="notice">给您带来的不便，我们深表歉意。维护完成后，我们将立即恢复服务。</p>
    </template>
  </div>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f7fa;
  color: #333;
}

.maintenance-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 2rem;
}

.status-card {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.status-badge {
  display: inline-block;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: bold;
  margin-bottom: 1.5rem;
}

.status-badge.即将 {
  background-color: #f39c12;
  color: white;
}

.status-badge.进行中 {
  background-color: #e74c3c;
  color: white;
}

.status-badge.已完成 {
  background-color: #2ecc71;
  color: white;
}

.time-info {
  text-align: left;
  max-width: 500px;
  margin: 0 auto;
}

.time-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.8rem;
  padding-bottom: 0.8rem;
  border-bottom: 1px solid #eee;
}

.time-row span:first-child {
  font-weight: 600;
  color: #555;
}

.loading {
  color: #3498db;
  font-size: 1.2rem;
  margin: 2rem 0;
}

.error {
  color: #e74c3c;
  font-size: 1.2rem;
  margin: 2rem 0;
  padding: 1rem;
  background-color: #fdecea;
  border-radius: 8px;
}

.notice {
  color: #7f8c8d;
  font-style: italic;
}

.default-maintenance {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.notification {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: #e74c3c;
  color: white;
  padding: 12px 24px;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  animation: fadeInOut 3s ease-in-out;
}

@keyframes fadeInOut {
  0% { opacity: 0; transform: translateX(-50%) translateY(-20px); }
  10% { opacity: 1; transform: translateX(-50%) translateY(0); }
  90% { opacity: 1; transform: translateX(-50%) translateY(0); }
  100% { opacity: 0; transform: translateX(-50%) translateY(-20px); }
}

@media (max-width: 600px) {
  .maintenance-container {
    padding: 1rem;
  }
  
  .status-card {
    padding: 1rem;
  }
  
  .time-row {
    flex-direction: column;
  }
}
</style>
