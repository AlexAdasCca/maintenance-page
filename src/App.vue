<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

// 维护状态和配置
const status = ref<'即将' | '进行中' | '已完成'>('进行中')
const startTime = ref(new Date('2025-05-30T10:00:00'))
const endTime = ref(new Date('2025-05-30T18:00:00'))
const currentTime = ref(new Date())
const remainingTime = ref('')

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
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>

<template>
  <div class="maintenance-container">
    <h1>网站维护通知</h1>
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

.notice {
  color: #7f8c8d;
  font-style: italic;
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
