<template>
  <el-card class="overview-card">
    <template #header>
      <div class="card-header">
        <span>阅读概况</span>
      </div>
    </template>
    <div class="overview-grid">
      <div class="overview-item">
        <div class="label">总阅读时长</div>
        <div class="value">{{ totalMinutes }} 分钟</div>
      </div>
      <div class="overview-item">
        <div class="label">书籍数量</div>
        <div class="value">{{ totalBooks }} 本</div>
      </div>
      <div class="overview-item">
        <div class="label">平均单次阅读</div>
        <div class="value">{{ averageMinutes }} 分钟</div>
      </div>
    </div>
  </el-card>
</template>

<script setup>
import { computed } from 'vue'
import { useStore } from 'vuex'

const store = useStore()
const logs = computed(() => store.state.logs || [])

const totalMinutes = computed(() =>
  logs.value.reduce((sum, log) => sum + (log.readingMinutes || 0), 0)
)
const totalBooks = computed(() =>
  [...new Set(logs.value.map(log => log.bookTitle))].length
)
const averageMinutes = computed(() =>
  logs.value.length > 0 ? Math.round(totalMinutes.value / logs.value.length) : 0
)
</script>

<style scoped>
.overview-card {
  background: #1e2a38;
  color: #e0f0ff;
  border-radius: 12px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  padding: 20px;
}
.card-header {
  font-size: 18px;
  font-weight: bold;
  color: #fff;
}
.overview-grid {
  display: flex;
  justify-content: space-around;
  padding-top: 20px;
}
.overview-item {
  text-align: center;
}
.label {
  font-size: 14px;
  color: #aaa;
}
.value {
  font-size: 20px;
  font-weight: bold;
  margin-top: 8px;
  color: #6cf;
}
</style>
