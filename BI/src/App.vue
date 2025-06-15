<template>
  <div id="app">
    <div class="main-container">
      <div class="header">
        <h1>📰 新闻实时动态分析系统</h1>
        <p>基于PENS数据集的智能新闻分析平台</p>
      </div>
    </div>

    <div class="dashboard">
      <!-- 单个新闻生命周期 -->
      <div class="card">
        <h3>📈 单个新闻生命周期分析</h3>
        <div class="input-group">
          <input type="text" v-model="newsLifecycle.newsId" placeholder="输入新闻ID (如: N54321)">
          <button class="btn" @click="queryNewsLifecycle" :disabled="newsLifecycle.loading">
            {{ newsLifecycle.loading ? '查询中...' : '查询生命周期' }}
          </button>
        </div>

        <div v-if="newsLifecycle.loading" class="loading">
          <div class="loading-spinner"></div>
          正在分析新闻生命周期...
        </div>

        <transition name="fade">
          <div v-if="newsLifecycle.error" class="error-message">
            {{ newsLifecycle.error }}
          </div>
        </transition>

        <div class="chart-container">
          <canvas :id="'newsLifecycleChart'" ref="newsLifecycleChart"></canvas>
        </div>
      </div>

      <!-- 新闻类别变化 -->
      <div class="card">
        <h3>📊 新闻类别趋势分析</h3>
        <div class="input-group">
          <select v-model="categoryTrend.selectedCategory">
            <option value="adexperience">adexperience</option>
            <option value="autos">autos</option>
            <option value="entertainment">entertainment</option>
            <option value="europe">europe</option>
            <option value="finance">finance</option>
            <option value="foodanddrink">foodanddrink</option>
            <option value="health">health</option>
            <option value="kids">kids</option>
            <option value="lifestyle">lifestyle</option>
            <option value="movies">movies</option>
            <option value="music">music</option>
            <option value="news">news</option>
            <option value="sports">sports</option>
            <option value="travel">travel</option>
            <option value="tv">tv</option>
            <option value="video">video</option>
            <option value="weather">weather</option>
          </select>
          <button class="btn" @click="queryCategoryTrend" :disabled="categoryTrend.loading">
            {{ categoryTrend.loading ? '查询中...' : '查询类别趋势' }}
          </button>
        </div>

        <div v-if="categoryTrend.loading" class="loading">
          <div class="loading-spinner"></div>
          正在分析类别趋势...
        </div>

        <transition name="fade">
          <div v-if="categoryTrend.error" class="error-message">
            {{ categoryTrend.error }}
          </div>
        </transition>

        <div class="chart-container">
          <canvas :id="'categoryTrendChart'" ref="categoryTrendChart"></canvas>
        </div>
      </div>

      <!-- 用户兴趣变化 -->
      <div class="card">
        <h3>👤 用户兴趣分析</h3>
        <div class="input-group">
          <input type="text" v-model="userInterest.userId" placeholder="输入用户ID (如: U99745)">
          <button class="btn" @click="queryUserInterest" :disabled="userInterest.loading || !userInterest.userId.trim()">
            {{ userInterest.loading ? '分析中...' : '分析用户兴趣' }}
          </button>
        </div>

        <div v-if="userInterest.loading" class="loading">
          <div class="loading-spinner"></div>
          正在分析用户兴趣变化...
        </div>

        <transition name="fade">
          <div v-if="userInterest.error" class="error-message">
            {{ userInterest.error }}
          </div>
        </transition>

        <div class="chart-container">
          <canvas :id="'userInterestChart'" ref="userInterestChart"></canvas>
        </div>
      </div>

      <!-- 多条件查询 -->
      <div class="card">
        <h3>🔍 多条件统计查询</h3>
        <div class="multi-condition-form">
          <div class="form-row">
            <label>时间范围:</label>
            <input type="datetime-local" v-model="multiCondition.startTime" placeholder="开始时间">
            <input type="datetime-local" v-model="multiCondition.endTime" placeholder="结束时间">
          </div>

          <div class="form-row">
            <label>新闻主题:</label>
            <select v-model="multiCondition.topic">
              <option value="">所有主题</option>
              <option value="adexperience">adexperience</option>
              <option value="autos">autos</option>
              <option value="entertainment">entertainment</option>
              <option value="europe">europe</option>
              <option value="finance">finance</option>
              <option value="foodanddrink">foodanddrink</option>
              <option value="health">health</option>
              <option value="kids">kids</option>
              <option value="lifestyle">lifestyle</option>
              <option value="movies">movies</option>
              <option value="music">music</option>
              <option value="news">news</option>
              <option value="sports">sports</option>
              <option value="travel">travel</option>
              <option value="tv">tv</option>
              <option value="video">video</option>
              <option value="weather">weather</option>
            </select>
          </div>

          <div class="form-row">
            <label>标题长度:</label>
            <input type="number" v-model="multiCondition.minTitleLength" placeholder="最小长度" min="0">
            <input type="number" v-model="multiCondition.maxTitleLength" placeholder="最大长度" min="0">
          </div>

          <div class="form-row">
            <label>内容长度:</label>
            <input type="number" v-model="multiCondition.minContentLength" placeholder="最小字数" min="0">
            <input type="number" v-model="multiCondition.maxContentLength" placeholder="最大字数" min="0">
          </div>

          <div class="form-row">
            <label>用户ID:</label>
            <textarea v-model="multiCondition.userIds" placeholder="输入用户ID，用逗号分隔 (如: U54737,U101144)" rows="2"></textarea>
          </div>

          <button class="btn btn-primary" @click="queryMultiCondition" :disabled="multiCondition.loading">
            {{ multiCondition.loading ? '查询中...' : '多条件查询' }}
          </button>
        </div>

        <div v-if="multiCondition.loading" class="loading">
          <div class="loading-spinner"></div>
          正在执行多条件查询...
        </div>

        <transition name="fade">
          <div v-if="multiCondition.error" class="error-message">
            {{ multiCondition.error }}
          </div>
        </transition>

        <!-- 查询结果显示 -->
        <transition name="fade">
          <div v-if="multiCondition.results" class="query-results">
            <h4>📊 查询结果</h4>
            <div class="result-cards">
              <div class="result-card">
                <div class="result-number">{{ multiCondition.results.impressions?.toLocaleString() || 0 }}</div>
                <div class="result-label">总曝光次数</div>
              </div>
              <div class="result-card">
                <div class="result-number">{{ formatDwellTime(multiCondition.results.total_dwell_time) }}</div>
                <div class="result-label">总停留时间</div>
              </div>
              <div class="result-card">
                <div class="result-number">{{ multiCondition.results.unique_news?.toLocaleString() || 0 }}</div>
                <div class="result-label">不同新闻数</div>
              </div>
              <div class="result-card">
                <div class="result-number">{{ multiCondition.results.unique_users?.toLocaleString() || 0 }}</div>
                <div class="result-label">不同用户数</div>
              </div>
            </div>
          </div>
        </transition>

        <div class="chart-container">
          <canvas :id="'multiConditionChart'" ref="multiConditionChart"></canvas>
        </div>
      </div>

      <!-- AI爆款分析 -->
      <div class="card">
        <h3>🤖 AI爆款新闻预测</h3>
        <div class="input-group">
          <textarea v-model="viralNews.query" placeholder="输入自然语言查询 (如: 什么样的科技新闻容易成为爆款?)"></textarea>
          <button class="btn" @click="queryViralNews" :disabled="viralNews.loading || !viralNews.query.trim()">
            {{ viralNews.loading ? 'AI分析中...' : 'AI分析' }}
          </button>
        </div>

        <div v-if="viralNews.loading" class="loading">
          <div class="loading-spinner"></div>
          AI正在分析爆款新闻特征...
        </div>

        <transition name="fade">
          <div v-if="viralNews.error" class="error-message">
            {{ viralNews.error }}
          </div>
        </transition>

        <transition name="slide">
          <div v-if="viralNews.response" class="ai-response" v-html="viralNews.response"></div>
        </transition>
      </div>

      <!-- 个性化推荐 -->
      <div class="card">
        <h3>🎯 个性化新闻推荐</h3>
        <div class="input-group">
          <input type="text" v-model="recommendations.userId" placeholder="输入用户ID (如: U99745)">
          <button class="btn" @click="getRecommendations" :disabled="recommendations.loading || !recommendations.userId.trim()">
            {{ recommendations.loading ? '生成中...' : '获取推荐' }}
          </button>
        </div>

        <div v-if="recommendations.loading" class="loading">
          <div class="loading-spinner"></div>
          正在生成个性化推荐...
        </div>

        <transition name="fade">
          <div v-if="recommendations.error" class="error-message">
            {{ recommendations.error }}
          </div>
        </transition>

        <div class="news-list">
          <transition-group name="slide">
            <div v-for="news in recommendations.list" :key="news.id" class="news-item">
              <div class="news-title">{{ news.title }}</div>
              <div class="news-meta">
                <span class="meta-item">📅 {{ news.date }}</span>
                <span class="meta-item">🏷️ {{ news.topic }}</span>
                <span class="meta-item">📁 {{ news.category }}</span>
                <span class="meta-item">📝 标题长度: {{ news.titleLength }}</span>
                <span class="meta-item">📄 内容长度: {{ news.contentLength }}字</span>
              </div>
            </div>
          </transition-group>
        </div>
      </div>

      <!-- SQL 执行日志查询 -->
      <div class="card">
        <h3>📋 SQL 执行日志查询</h3>
        <div class="input-group">
          <button class="btn" @click="querySQLLogs" :disabled="sqlLogs.loading">
            {{ sqlLogs.loading ? '查询中...' : '获取 SQL 执行日志' }}
          </button>
          <button class="btn btn-secondary" @click="refreshSQLLogs" :disabled="sqlLogs.loading">
            {{ sqlLogs.loading ? '刷新中...' : '刷新日志' }}
          </button>
        </div>

        <div v-if="sqlLogs.loading" class="loading">
          <div class="loading-spinner"></div>
          正在获取 SQL 执行日志...
        </div>

        <transition name="fade">
          <div v-if="sqlLogs.error" class="error-message">
            {{ sqlLogs.error }}
          </div>
        </transition>

        <!-- SQL 日志统计信息 -->
        <transition name="fade">
          <div v-if="sqlLogs.logs && sqlLogs.logs.length > 0" class="sql-logs-summary">
            <h4>📊 执行统计</h4>
            <div class="sql-stats-cards">
              <div class="sql-stat-card">
                <div class="stat-number">{{ sqlLogs.logs.length }}</div>
                <div class="stat-label">总查询数</div>
              </div>
              <div class="sql-stat-card">
                <div class="stat-number">{{ getQueryTypeStats().type1 }}</div>
                <div class="stat-label">类型1查询</div>
              </div>
              <div class="sql-stat-card">
                <div class="stat-number">{{ getQueryTypeStats().type2 }}</div>
                <div class="stat-label">类型2查询</div>
              </div>
              <div class="sql-stat-card">
                <div class="stat-number">{{ getQueryTypeStats().type3 }}</div>
                <div class="stat-label">类型3查询</div>
              </div>
              <div class="sql-stat-card">
                <div class="stat-number">{{ getQueryTypeStats().type4 }}</div>
                <div class="stat-label">类型4查询</div>
              </div>
              <div class="sql-stat-card">
                <div class="stat-number">{{ getAverageExecutionTime() }}ms</div>
                <div class="stat-label">平均执行时间</div>
              </div>
            </div>
          </div>
        </transition>

        <!-- SQL 日志列表 -->
        <div class="sql-logs-container">
          <transition-group name="slide">
            <div v-for="log in sqlLogs.logs" :key="log.id || Math.random()" class="sql-log-item">
              <div class="sql-log-header">
                <div class="sql-log-type">
                  <span class="type-badge" :class="'type-' + log.query_type">
                    查询类型 {{ log.query_type }}
                  </span>
                </div>
                <div class="sql-log-stats">
                  <span class="stat-item">
                    <i class="icon">⏱️</i>
                    平均: {{ log.avg_duration_ms }}ms
                  </span>
                  <span class="stat-item">
                    <i class="icon">🔢</i>
                    执行次数: {{ log.count }}
                  </span>
                  <span class="stat-item">
                    <i class="icon">⚡</i>
                    最快: {{ log.min_duration_ms }}ms
                  </span>
                  <span class="stat-item">
                    <i class="icon">🐌</i>
                    最慢: {{ log.max_duration_ms }}ms
                  </span>
                </div>
              </div>

              <div class="sql-log-performance">
                <div class="performance-bar">
                  <div class="performance-fill" :style="{ width: getPerformancePercentage(log.avg_duration_ms) + '%' }"></div>
                </div>
                <div class="performance-label">
                  性能评级: {{ getPerformanceGrade(log.avg_duration_ms) }}
                </div>
              </div>
            </div>
          </transition-group>
        </div>

        <!-- SQL 性能分析图表 -->
        <div class="chart-container" v-if="sqlLogs.logs && sqlLogs.logs.length > 0">
          <canvas :id="'sqlLogsChart'" ref="sqlLogsChart"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { marked } from 'marked'

import { ref, reactive, onMounted, nextTick } from 'vue'
import Chart from 'chart.js/auto'

// API基础URL - 根据你的后端地址修改
const API_BASE_URL = 'http://113.44.75.241:5000'

// refs for canvas elements
const newsLifecycleChart = ref(null)
const categoryTrendChart = ref(null)
const userInterestChart = ref(null)
const multiConditionChart = ref(null)
const sqlLogsChart = ref(null)

// 各功能模块的状态
const newsLifecycle = reactive({
  newsId: 'N90607',
  loading: false,
  error: null,
  chart: null
})

const categoryTrend = reactive({
  selectedCategory: 'sports',
  loading: false,
  error: null,
  chart: null
})

const userInterest = reactive({
  userId: '',
  loading: false,
  error: null,
  chart: null
})

const multiCondition = reactive({
  startTime: '2019-06-13T20:00',
  endTime: '2019-06-13T21:00',
  topic: '',
  minTitleLength: '',
  maxTitleLength: '',
  minContentLength: '',
  maxContentLength: '',
  userIds: '',
  loading: false,
  error: null,
  results: null,
  chart: null
})

const viralNews = reactive({
  query: '',
  loading: false,
  error: null,
  response: null
})

const recommendations = reactive({
  userId: '',
  loading: false,
  error: null,
  list: []
})

// 新增：SQL 执行日志状态
const sqlLogs = reactive({
  loading: false,
  error: null,
  logs: []
})

// Chart.js 实例存储
const charts = {}

// 创建或更新图表
const createOrUpdateChart = (canvasRef, config, chartKey) => {
  if (!canvasRef) {
    console.error('Canvas reference is null')
    return
  }

  const ctx = canvasRef.getContext('2d')

  // 销毁旧图表
  if (charts[chartKey]) {
    charts[chartKey].destroy()
  }

  // 创建新图表
  charts[chartKey] = new Chart(ctx, config)
  return charts[chartKey]
}

// 新闻生命周期查询
const queryNewsLifecycle = async () => {
  if (!newsLifecycle.newsId.trim()) {
    newsLifecycle.error = '请输入新闻ID'
    return
  }

  newsLifecycle.loading = true
  newsLifecycle.error = null

  try {
    await nextTick()

    const response = await axios.get('http://113.44.75.241:5000/api/news/trend', {
      params: {
        news_id: newsLifecycle.newsId
      }
    })

    const result = response.data

    const labels = result.trend.map(item => {
      const date = new Date(item.ts)
      return date.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
    })
    const views = result.trend.map(item => item.impressions)

    if (!newsLifecycleChart.value) {
      console.error('newsLifecycleChart ref is null')
      return
    }

    const chartConfig = {
      type: 'line',
      data: {
        labels,
        datasets: [{
          label: '浏览量',
          data: views,
          borderColor: '#667eea',
          backgroundColor: 'rgba(102, 126, 234, 0.1)',
          borderWidth: 3,
          fill: true,
          tension: 0.4,
          pointBackgroundColor: '#667eea',
          pointBorderColor: '#fff',
          pointBorderWidth: 2,
          pointRadius: 5
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: `新闻 ${newsLifecycle.newsId} 生命周期分析`,
            font: { size: 16, weight: 'bold' }
          },
          legend: {
            display: true,
            position: 'top'
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: '浏览量',
              font: { weight: 'bold' }
            },
            grid: {
              color: 'rgba(0,0,0,0.1)'
            }
          },
          x: {
            title: {
              display: true,
              text: '时间',
              font: { weight: 'bold' }
            },
            grid: {
              color: 'rgba(0,0,0,0.1)'
            }
          }
        }
      }
    }

    createOrUpdateChart(newsLifecycleChart.value, chartConfig, 'newsLifecycle')

  } catch (err) {
    newsLifecycle.error = `查询失败：${err.message}`
    console.error('Error in queryNewsLifecycle:', err)
  } finally {
    newsLifecycle.loading = false
  }
}

// 新闻类别趋势查询
const queryCategoryTrend = async () => {
  categoryTrend.loading = true
  categoryTrend.error = null

  try {
    await nextTick()

    const response = await axios.get('http://113.44.75.241:5000/api/news/category-trend', {
      params: {
        category: categoryTrend.selectedCategory
      }
    })

    const result = response.data

    const data = {
      labels: result.trend.map(item => {
        const date = new Date(item.day)
        return `${date.getMonth() + 1}-${date.getDate()}`
      }),
      views: result.trend.map(item => item.impressions)
    }

    if (!categoryTrendChart.value) {
      console.error('categoryTrendChart ref is null')
      return
    }

    const chartConfig = {
      type: 'bar',
      data: {
        labels: data.labels,
        datasets: [{
          label: `${categoryTrend.selectedCategory} 类别浏览量`,
          data: data.views,
          backgroundColor: 'rgba(118, 75, 162, 0.8)',
          borderColor: '#764ba2',
          borderWidth: 1,
          borderRadius: 8
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: `${categoryTrend.selectedCategory} 类别趋势分析`,
            font: { size: 16, weight: 'bold' }
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: '浏览量',
              font: { weight: 'bold' }
            }
          }
        }
      }
    }

    createOrUpdateChart(categoryTrendChart.value, chartConfig, 'categoryTrend')

  } catch (error) {
    categoryTrend.error = '查询失败: ' + error.message
  } finally {
    categoryTrend.loading = false
  }
}

// 用户兴趣查询
const queryUserInterest = async () => {
  if (!userInterest.userId.trim()) {
    userInterest.error = '请输入用户ID'
    return
  }

  userInterest.loading = true
  userInterest.error = null

  try {
    await nextTick()
    const response = await axios.get('http://113.44.75.241:5000/api/user/interest-trend', {
      params: {
        user_id: userInterest.userId
      }
    })

    const result = response.data

    const data = {
      labels: result.trend.map(item => item.category),
      time: result.trend.map(item => Math.round(item.total_dwell_time / 60))
    }

    if (!userInterestChart.value) {
      console.error('userInterestChart ref is null')
      return
    }

    const chartConfig = {
      type: 'doughnut',
      data: {
        labels: data.labels,
        datasets: [{
          data: data.time,
          backgroundColor: [
            '#667eea',
            '#764ba2',
            '#f093fb',
            '#f5576c',
            '#4facfe',
            '#43e97b'
          ],
          borderWidth: 3,
          borderColor: '#fff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: `User ${userInterest.userId} Interest Distribution (minutes)`,
            font: { size: 16, weight: 'bold' }
          },
          legend: {
            position: 'bottom',
            labels: {
              padding: 20,
              usePointStyle: true
            }
          }
        }
      }
    }

    createOrUpdateChart(userInterestChart.value, chartConfig, 'userInterest')

  } catch (error) {
    userInterest.error = '查询失败: ' + error.message
  } finally {
    userInterest.loading = false
  }
}

// 格式化停留时间显示
const formatDwellTime = (seconds) => {
  if (!seconds) return '0秒'

  const hours = Math.floor(seconds / 3600)
  const minutes = Math.floor((seconds % 3600) / 60)
  const secs = seconds % 60

  if (hours > 0) {
    return `${hours}小时${minutes}分钟`
  } else if (minutes > 0) {
    return `${minutes}分钟${secs}秒`
  } else {
    return `${secs}秒`
  }
}

// 多条件查询
const queryMultiCondition = async () => {
  multiCondition.loading = true
  multiCondition.error = null
  multiCondition.results = null

  try {
    await nextTick()

    const params = new URLSearchParams()

    if (multiCondition.startTime) {
      const startTime = multiCondition.startTime.replace('T', ' ') + ':00'
      params.append('start_time', startTime)
    }

    if (multiCondition.endTime) {
      const endTime = multiCondition.endTime.replace('T', ' ') + ':00'
      params.append('end_time', endTime)
    }

    if (multiCondition.topic) {
      params.append('topic', multiCondition.topic)
    }

    if (multiCondition.minTitleLength) {
      params.append('min_title_len', multiCondition.minTitleLength)
    }
    if (multiCondition.maxTitleLength) {
      params.append('max_title_len', multiCondition.maxTitleLength)
    }

    if (multiCondition.minContentLength) {
      params.append('min_content_len', multiCondition.minContentLength)
    }
    if (multiCondition.maxContentLength) {
      params.append('max_content_len', multiCondition.maxContentLength)
    }

    if (multiCondition.userIds.trim()) {
      const userIdList = multiCondition.userIds.split(',').map(id => id.trim()).filter(id => id)
      userIdList.forEach(userId => {
        params.append('user_id', userId)
      })
    }

    const response = await axios.get(`${API_BASE_URL}/api/news/advanced-query?${params.toString()}`)

    if (response.data && response.data.length > 0) {
      multiCondition.results = response.data[0]
      createResultsChart()
    } else {
      multiCondition.error = '未找到符合条件的数据'
    }

  } catch (error) {
    console.error('多条件查询失败:', error)
    multiCondition.error = '查询失败: ' + (error.response?.data?.message || error.message)
  } finally {
    multiCondition.loading = false
  }
}

// 创建结果可视化图表
const createResultsChart = () => {
  if (!multiConditionChart.value || !multiCondition.results) {
    return
  }

  const results = multiCondition.results

  const chartConfig = {
    type: 'bar',
    data: {
      labels: ['曝光次数', '不同新闻数', '不同用户数'],
      datasets: [{
        label: '统计指标',
        data: [
          results.impressions || 0,
          results.unique_news || 0,
          results.unique_users || 0
        ],
        backgroundColor: [
          'rgba(102, 126, 234, 0.8)',
          'rgba(118, 75, 162, 0.8)',
          'rgba(240, 147, 251, 0.8)'
        ],
        borderColor: [
          '#667eea',
          '#764ba2',
          '#f093fb'
        ],
        borderWidth: 2,
        borderRadius: 8
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        title: {
          display: true,
          text: '多条件查询统计结果',
          font: { size: 16, weight: 'bold' }
        },
        legend: {
          display: false
        }
      },
      scales: {
        y: {
          beginAtZero: true,
          title: {
            display: true,
            text: '数量',
            font: { weight: 'bold' }
          },
          ticks: {
            callback: function(value) {
              return value.toLocaleString()
            }
          }
        },
        x: {
          title: {
            display: true,
            text: '统计指标',
            font: { weight: 'bold' }
          }
        }
      }
    }
  }

  createOrUpdateChart(multiConditionChart.value, chartConfig, 'multiCondition')
}

// 爆款新闻分析
const queryViralNews = async () => {
  if (!viralNews.query.trim()) {
    viralNews.error = '请输入查询内容'
    return
  }

  viralNews.loading = true
  viralNews.error = null
  viralNews.response = null

  try {
    const response = await axios.post('http://113.44.75.241:5001/api/analyze', {
      query: viralNews.query
    })

    const resultMarkdown = response.data.result || '⚠️ 返回结果为空'
    viralNews.response = marked.parse(resultMarkdown)

  } catch (error) {
    viralNews.error = 'AI分析失败: ' + (error.response?.data?.message || error.message)
  } finally {
    viralNews.loading = false
  }
}

// 个性化推荐查询
const getRecommendations = async () => {
  if (!recommendations.userId.trim()) {
    recommendations.error = '请输入用户ID'
    return
  }

  recommendations.loading = true
  recommendations.error = null

  try {
    const response = await fetch(`http://113.44.75.241:5000/api/recommendations/${recommendations.userId}`)
    if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`)

    const data = await response.json()

    // 解析后端返回数据格式
    recommendations.list = data.recommendations.map((item, index) => ({
      id: index + 1,
      date: new Date().toISOString().slice(0, 10), // 默认填今天的日期
      title: `为用户 ${recommendations.userId} 推荐: ${item.title}`,
      topic: item.topic || '未知话题',
      category: item.category || 'unknown',
      titleLength: item.title.length,
      contentLength: 0 // 后端没有提供内容长度，可根据需要补充
    }))
  } catch (error) {
    recommendations.error = '获取推荐失败: ' + error.message
  } finally {
    recommendations.loading = false
  }
}


// 新增：查询 SQL 执行日志
const querySQLLogs = async () => {
  sqlLogs.loading = true
  sqlLogs.error = null

  try {
    await nextTick()

    const response = await axios.get(`${API_BASE_URL}/api/query/logs`)

    if (response.data && Array.isArray(response.data)) {
      sqlLogs.logs = response.data

      // 创建 SQL 性能分析图表
      if (sqlLogs.logs.length > 0) {
        createSQLLogsChart()
      }
    } else {
      sqlLogs.error = '未获取到日志数据'
    }

  } catch (error) {
    console.error('SQL日志查询失败:', error)
    sqlLogs.error = '查询失败: ' + (error.response?.data?.message || error.message)
  } finally {
    sqlLogs.loading = false
  }
}

// 新增：刷新 SQL 日志
const refreshSQLLogs = async () => {
  await querySQLLogs()
}

// 新增：获取查询类型统计
const getQueryTypeStats = () => {
  if (!sqlLogs.logs || sqlLogs.logs.length === 0) {
    return { type1: 0, type2: 0, type3: 0, type4: 0 }
  }

  const stats = { type1: 0, type2: 0, type3: 0, type4: 0 }

  sqlLogs.logs.forEach(log => {
    switch (log.query_type) {
      case 1: stats.type1 += log.count; break
      case 2: stats.type2 += log.count; break
      case 3: stats.type3 += log.count; break
      case 4: stats.type4 += log.count; break
    }
  })

  return stats
}

// 新增：计算平均执行时间
const getAverageExecutionTime = () => {
  if (!sqlLogs.logs || sqlLogs.logs.length === 0) return 0

  const totalTime = sqlLogs.logs.reduce((sum, log) => sum + (log.avg_duration_ms * log.count), 0)
  const totalCount = sqlLogs.logs.reduce((sum, log) => sum + log.count, 0)

  return totalCount > 0 ? Math.round(totalTime / totalCount) : 0
}

// 新增：获取性能百分比
const getPerformancePercentage = (avgTime) => {
  const maxTime = Math.max(...sqlLogs.logs.map(log => log.avg_duration_ms), 1000)
  return Math.min((avgTime / maxTime) * 100, 100)
}

// 新增：获取性能等级
const getPerformanceGrade = (avgTime) => {
  if (avgTime < 100) return '优秀'
  if (avgTime < 300) return '良好'
  if (avgTime < 500) return '一般'
  if (avgTime < 1000) return '较差'
  return '很差'
}

// 新增：创建 SQL 日志性能分析图表
const createSQLLogsChart = async () => {
  await nextTick()

  if (!sqlLogsChart.value || !sqlLogs.logs || sqlLogs.logs.length === 0) {
    return
  }

  const chartConfig = {
    type: 'bar',
    data: {
      labels: sqlLogs.logs.map(log => `类型 ${log.query_type}`),
      datasets: [
        {
          label: '平均执行时间 (ms)',
          data: sqlLogs.logs.map(log => log.avg_duration_ms),
          backgroundColor: 'rgba(102, 126, 234, 0.8)',
          borderColor: '#667eea',
          borderWidth: 2,
          borderRadius: 8,
          yAxisID: 'y'
        },
        {
          label: '执行次数',
          data: sqlLogs.logs.map(log => log.count),
          backgroundColor: 'rgba(118, 75, 162, 0.8)',
          borderColor: '#764ba2',
          borderWidth: 2,
          borderRadius: 8,
          yAxisID: 'y1'
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        title: {
          display: true,
          text: 'SQL 查询性能分析',
          font: { size: 16, weight: 'bold' }
        },
        legend: {
          display: true,
          position: 'top'
        }
      },
      scales: {
        y: {
          type: 'linear',
          display: true,
          position: 'left',
          title: {
            display: true,
            text: '执行时间 (ms)',
            font: { weight: 'bold' }
          },
          beginAtZero: true
        },
        y1: {
          type: 'linear',
          display: true,
          position: 'right',
          title: {
            display: true,
            text: '执行次数',
            font: { weight: 'bold' }
          },
          beginAtZero: true,
          grid: {
            drawOnChartArea: false,
          },
        },
        x: {
          title: {
            display: true,
            text: '查询类型',
            font: { weight: 'bold' }
          }
        }
      }
    }
  }

  createOrUpdateChart(sqlLogsChart.value, chartConfig, 'sqlLogs')
}

// 组件挂载后的初始化
onMounted(() => {
  console.log('新闻实时动态分析系统已加载')
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
  line-height: 1.6;
}

#app {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
}

.main-container {
  width: 100%;
  display: flex;
  flex-direction: column;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
  padding: 20px 0;
  width: 100%;
  display: block;
}

.header h1 {
  font-size: 3rem;
  margin-bottom: 15px;
  text-shadow: 0 4px 8px rgba(0,0,0,0.3);
  font-weight: 700;
  letter-spacing: -0.5px;
}

.header p {
  font-size: 1.2rem;
  opacity: 0.95;
  font-weight: 300;
}

.input-group {
  display: flex;
  gap: 20px;
  align-items: center;
  flex-wrap: wrap;
  width: 100%;
}

.multi-condition-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-row {
  display: flex;
  align-items: center;
  gap: 15px;
  flex-wrap: wrap;
}

.form-row label {
  min-width: 100px;
  font-weight: 600;
  color: #2d3748;
}

input[type="text"],
input[type="date"],
input[type="number"],
input[type="datetime-local"],
select,
textarea {
  padding: 14px 18px;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  font-size: 15px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  min-width: 220px;
  background: white;
  font-family: inherit;
}

input[type="number"] {
  min-width: 120px;
}

textarea {
  min-width: 350px;
  min-height: 80px;
  resize: vertical;
  font-family: inherit;
}

input:focus,
select:focus,
textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.1);
  transform: translateY(-1px);
}

.btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 14px 28px;
  border-radius: 12px;
  cursor: pointer;
  font-size: 15px;
  font-weight: 600;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.btn-primary {
  margin-top: 10px;
  padding: 16px 32px;
  font-size: 16px;
}

.btn-secondary {
  background: linear-gradient(135deg, #718096 0%, #4a5568 100%);
  box-shadow: 0 4px 12px rgba(113, 128, 150, 0.3);
}

.btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
}

.btn-secondary:hover:not(:disabled) {
  box-shadow: 0 8px 25px rgba(113, 128, 150, 0.4);
}

.btn:active:not(:disabled) {
  transform: translateY(-1px);
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.dashboard {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(650px, 1fr));
  gap: 30px;
  width: 100%;
}

.card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
  border: 1px solid rgba(255,255,255,0.2);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.card:hover {
  transform: translateY(-8px);
  box-shadow: 0 30px 50px rgba(0,0,0,0.15);
}

.card h3 {
  color: #2d3748;
  margin-bottom: 25px;
  font-size: 1.3rem;
  font-weight: 600;
  border-bottom: 3px solid #f7fafc;
  padding-bottom: 15px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.chart-container {
  position: relative;
  height: 350px;
  margin-top: 20px;
  background: white;
  border-radius: 12px;
  padding: 15px;
}

.chart-container canvas {
  width: 100% !important;
  height: 100% !important;
}

.loading {
  text-align: center;
  padding: 50px 20px;
  color: #718096;
}

.loading-spinner {
  display: inline-block;
  width: 24px;
  height: 24px;
  border: 3px solid #e2e8f0;
  border-top: 3px solid #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-right: 12px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.news-list {
  max-height: 450px;
  overflow-y: auto;
  padding-right: 10px;
}

.news-list::-webkit-scrollbar {
  width: 6px;
}

.news-list::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 3px;
}

.news-list::-webkit-scrollbar-thumb {
  background: #cbd5e0;
  border-radius: 3px;
}

.news-item {
  padding: 20px;
  border-left: 5px solid #667eea;
  margin-bottom: 20px;
  background: linear-gradient(135deg, #f8faff 0%, #f1f5ff 100%);
  border-radius: 0 15px 15px 0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.news-item:hover {
  transform: translateX(8px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.15);
  border-left-color: #764ba2;
}

.news-title {
  font-weight: 600;
  color: #2d3748;
  margin-bottom: 12px;
  font-size: 1.1rem;
  line-height: 1.4;
}

.news-meta {
  display: flex;
  gap: 12px;
  font-size: 13px;
  color: #718096;
  flex-wrap: wrap;
}

.meta-item {
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  padding: 6px 12px;
  border-radius: 20px;
  font-weight: 500;
}

.ai-response {
  background: linear-gradient(135deg, #f8faff 0%, #f1f5ff 100%);
  border-radius: 15px;
  padding: 25px;
  border-left: 5px solid #667eea;
  line-height: 1.7;
  color: #2d3748;
  margin-top: 20px;
}

.ai-response h4 {
  color: #667eea;
  margin-bottom: 15px;
  font-size: 1.2rem;
}

.ai-response ul {
  margin: 15px 0;
  padding-left: 25px;
}

.ai-response li {
  margin-bottom: 8px;
}

.error-message {
  background: linear-gradient(135deg, #fed7d7 0%, #feb2b2 100%);
  color: #c53030;
  padding: 15px 20px;
  border-radius: 12px;
  margin-top: 15px;
  border-left: 4px solid #f56565;
  font-weight: 500;
}

.success-message {
  background: linear-gradient(135deg, #c6f6d5 0%, #9ae6b4 100%);
  color: #2f855a;
  padding: 15px 20px;
  border-radius: 12px;
  margin-top: 15px;
  border-left: 4px solid #48bb78;
  font-weight: 500;
}

.query-results {
  margin-top: 25px;
  padding: 20px;
  background: linear-gradient(135deg, #f8faff 0%, #f1f5ff 100%);
  border-radius: 15px;
  border-left: 5px solid #667eea;
}

.query-results h4 {
  color: #667eea;
  margin-bottom: 20px;
  font-size: 1.2rem;
  text-align: center;
}

.result-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.result-card {
  background: white;
  padding: 20px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  transition: transform 0.3s ease;
}

.result-card:hover {
  transform: translateY(-3px);
}

.result-number {
  font-size: 2rem;
  font-weight: 700;
  color: #667eea;
  margin-bottom: 8px;
}

.result-label {
  font-size: 0.9rem;
  color: #718096;
  font-weight: 500;
}

/* SQL 日志相关样式 */
.sql-logs-summary {
  margin-top: 25px;
  padding: 20px;
  background: linear-gradient(135deg, #f8faff 0%, #f1f5ff 100%);
  border-radius: 15px;
  border-left: 5px solid #667eea;
}

.sql-logs-summary h4 {
  color: #667eea;
  margin-bottom: 20px;
  font-size: 1.2rem;
  text-align: center;
}

.sql-stats-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 12px;
}

.sql-stat-card {
  background: white;
  padding: 16px;
  border-radius: 10px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
  transition: transform 0.3s ease;
}

.sql-stat-card:hover {
  transform: translateY(-2px);
}

.stat-number {
  font-size: 1.5rem;
  font-weight: 700;
  color: #667eea;
  margin-bottom: 6px;
}

.stat-label {
  font-size: 0.8rem;
  color: #718096;
  font-weight: 500;
}

.sql-logs-container {
  max-height: 500px;
  overflow-y: auto;
  margin-top: 20px;
  padding-right: 10px;
}

.sql-logs-container::-webkit-scrollbar {
  width: 6px;
}

.sql-logs-container::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 3px;
}

.sql-logs-container::-webkit-scrollbar-thumb {
  background: #cbd5e0;
  border-radius: 3px;
}

.sql-log-item {
  background: linear-gradient(135deg, #f8faff 0%, #f1f5ff 100%);
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 15px;
  border-left: 5px solid #667eea;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.sql-log-item:hover {
  transform: translateX(5px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.15);
}

.sql-log-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 15px;
  flex-wrap: wrap;
  gap: 15px;
}

.sql-log-type {
  flex-shrink: 0;
}

.type-badge {
  display: inline-block;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 13px;
  font-weight: 600;
  color: white;
}

.type-badge.type-1 {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.type-badge.type-2 {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.type-badge.type-3 {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
}

.type-badge.type-4 {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
}

.sql-log-stats {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 13px;
  color: #718096;
  background: rgba(255, 255, 255, 0.8);
  padding: 6px 12px;
  border-radius: 20px;
  font-weight: 500;
}

.stat-item .icon {
  font-size: 14px;
}

.sql-log-performance {
  margin-top: 15px;
}

.performance-bar {
  height: 8px;
  background: #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 8px;
}

.performance-fill {
  height: 100%;
  background: linear-gradient(90deg, #48bb78 0%, #f093fb 50%, #f56565 100%);
  border-radius: 10px;
  transition: width 0.5s ease;
}

.performance-label {
  font-size: 12px;
  color: #718096;
  text-align: center;
  font-weight: 500;
}

/* Vue过渡动画 */
.fade-enter-active, .fade-leave-active {
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.slide-enter-active, .slide-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-enter-from {
  transform: translateY(-20px);
  opacity: 0;
}

.slide-leave-to {
  transform: translateY(20px);
  opacity: 0;
}

.slide-move {
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@media (max-width: 768px) {
  body {
    padding: 15px;
  }

  .header h1 {
    font-size: 2.2rem;
  }

  .dashboard {
    grid-template-columns: 1fr;
    gap: 20px;
  }

  .input-group, .form-row {
    flex-direction: column;
    align-items: stretch;
  }

  .form-row label {
    min-width: auto;
  }

  input[type="text"],
  input[type="date"],
  input[type="number"],
  input[type="datetime-local"],
  select,
  textarea {
    min-width: auto;
    width: 100%;
  }

  .card {
    padding: 20px;
  }

  .chart-container {
    height: 280px;
  }

  .result-cards {
    grid-template-columns: repeat(2, 1fr);
  }

  .sql-stats-cards {
    grid-template-columns: repeat(2, 1fr);
  }

  .sql-log-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .sql-log-stats {
    flex-direction: column;
    gap: 8px;
  }

  .result-number {
    font-size: 1.5rem;
  }
}

@media (max-width: 480px) {
  .header h1 {
    font-size: 1.8rem;
  }

  .header p {
    font-size: 1rem;
  }

  .card {
    padding: 20px;
  }

  .result-cards {
    grid-template-columns: 1fr;
  }

  .sql-stats-cards {
    grid-template-columns: 1fr;
  }
}

canvas {
  display: block;
  box-sizing: border-box;
}
</style>
