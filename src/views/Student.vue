<template>
  <div class="app-container">
    <!-- 顶部导航栏 -->
    <header class="top-nav">
      <div class="logo-section">
        <span class="logo-icon">
          <el-icon :size="28"><Reading /></el-icon>
        </span>
        <h1 class="app-title">综测服务智能体</h1>
      </div>
      <div class="nav-right">
        <span class="user-info-nav">
          <el-icon :size="18"><UserFilled /></el-icon>
          <span>{{ currentUser.name }} · {{ currentUser.grade }}</span>
        </span>
        <el-button text class="logout-btn" @click="handleLogout">
          <el-icon><SwitchButton /></el-icon>
          退出
        </el-button>
      </div>
    </header>

    <!-- 统计卡片区域 -->
    <section class="stats-cards">
      <el-row :gutter="16" class="card-row">
        <el-col :xs="24" :sm="12" :md="12" :lg="6" class="card-col">
          <div class="stat-card">
            <div class="card-header">
              <span class="card-icon icon-new">
                <el-icon :size="18"><Notification /></el-icon>
              </span>
              <span class="card-title">近3日新增活动</span>
            </div>
            <div class="card-content">
              <div class="card-big-text">{{ stats.newCount }} <span class="unit">场</span></div>
              <div class="card-list" v-if="stats.newList.length > 0">
                <div class="card-list-item" v-for="item in stats.newList" :key="item.id">
                  <span class="item-name">{{ item.title }}</span>
                  <span class="item-score">{{ item.category }}类 +{{ item.score }}</span>
                </div>
              </div>
              <div class="card-empty" v-else>暂无新活动</div>
            </div>
          </div>
        </el-col>
        <el-col :xs="24" :sm="12" :md="12" :lg="6" class="card-col">
          <div class="stat-card">
            <div class="card-header">
              <span class="card-icon icon-deadline">
                <el-icon :size="18"><Clock /></el-icon>
              </span>
              <span class="card-title">即将截止活动</span>
            </div>
            <div class="card-content">
              <div class="card-big-text">{{ stats.deadlineCount }} <span class="unit">场</span></div>
              <div class="card-list" v-if="stats.deadlineList.length > 0">
                <div class="card-list-item" v-for="item in stats.deadlineList" :key="item.id">
                  <span class="item-name">{{ item.title }}</span>
                  <span class="item-score deadline-text" :data-urgent="item.hoursLeft < 12 ? 'true' : item.hoursLeft < 24 ? 'warn' : 'normal'">
                    {{ item.category }}类 +{{ item.score }} | 剩余{{ item.hoursLeft }}h
                  </span>
                </div>
              </div>
              <div class="card-empty" v-else>暂无即将截止的活动</div>
            </div>
          </div>
        </el-col>
        <el-col :xs="24" :sm="12" :md="12" :lg="6" class="card-col">
          <div class="stat-card">
            <div class="card-header">
              <span class="card-icon icon-recommend">
                <el-icon :size="18"><StarFilled /></el-icon>
              </span>
              <span class="card-title">综测活动建议</span>
            </div>
            <div class="card-content">
              <div class="recommend-priority">{{ stats.recommendPriority }}类优先</div>
              <div class="recommend-reason">{{ stats.recommendReason }}</div>
              <div class="card-list" v-if="stats.recommendList.length > 0">
                <div class="card-list-item clickable" v-for="item in stats.recommendList" :key="item.id" @click="sendQuickQuestion(`帮我看看${item.title}`)">
                  <span class="item-name">{{ item.title }}</span>
                  <span class="item-score">{{ item.category }}类 +{{ item.score }}</span>
                </div>
              </div>
              <div class="card-empty" v-else>你的综测分数已达标，继续保持！</div>
            </div>
          </div>
        </el-col>
        <el-col :xs="24" :sm="12" :md="12" :lg="6" class="card-col">
          <div class="stat-card">
            <div class="card-header">
              <span class="card-icon icon-reminder">
                <el-icon :size="18"><Bell /></el-icon>
              </span>
              <span class="card-title">活动提醒</span>
            </div>
            <div class="card-content">
              <div class="card-big-text">{{ stats.reminderCount }} <span class="unit">条</span></div>
              <div class="card-list" v-if="stats.reminderList.length > 0">
                <div class="card-list-item" v-for="item in stats.reminderList" :key="item.id">
                  <span class="item-name">
                    {{ item.name }}
                    <el-tag v-if="item.isOral" size="small" type="info" class="oral-tag">口头</el-tag>
                  </span>
                  <span class="item-time">{{ item.eventTime }}</span>
                </div>
              </div>
              <div class="card-empty" v-else>暂无活动提醒，在对话中说帮我设置提醒即可</div>
            </div>
          </div>
        </el-col>
      </el-row>
    </section>

    <!-- 主体区域：左侧仪表盘 + 右侧对话 -->
    <main class="main-content">
      <!-- 左侧仪表盘 -->
      <aside class="dashboard-panel">
        <!-- 用户信息 -->
        <div class="user-info">
          <div class="user-avatar">
            <el-avatar :size="64" class="avatar-custom">
              {{ currentUser.name?.charAt(0) }}
            </el-avatar>
          </div>
          <div class="user-detail">
            <h3 class="user-name">{{ currentUser.name }}</h3>
            <p class="user-major">{{ currentUser.grade }} · {{ currentUser.major }}</p>
          </div>
        </div>

        <!-- 分数进度 -->
        <div class="score-section">
          <h4 class="section-title">
            <el-icon class="section-title-icon"><TrendCharts /></el-icon>
            综测分数
          </h4>
          <div class="score-item" v-for="score in scoreItems" :key="score.name">
            <div class="score-label">
              <span class="score-name">{{ score.name }}</span>
              <span class="score-value">{{ score.current }}<span class="score-max">/100</span></span>
            </div>
            <el-progress
              :percentage="score.current"
              :color="score.color"
              :stroke-width="10"
              :show-text="false"
              class="score-progress"
            />
          </div>

          <!-- 总分卡片 -->
          <div class="total-score-card">
            <div class="total-row">
              <span class="total-label">总分</span>
              <span class="total-value">{{ currentUser.totalScore }}</span>
            </div>
            <div class="total-row">
              <span class="total-label">目标</span>
              <span class="total-value target">{{ currentUser.targetScore }}</span>
            </div>
            <div class="total-row gap-row">
              <span class="total-label">缺口</span>
              <span class="total-value gap">{{ currentUser.gap }}</span>
            </div>
          </div>
        </div>

        <!-- 兴趣标签 -->
        <div class="interest-section">
          <h4 class="section-title">
            <el-icon class="section-title-icon"><Collection /></el-icon>
            兴趣标签
          </h4>
          <div class="tag-list">
            <el-tag v-for="tag in currentUser.interests" :key="tag" class="interest-tag" effect="plain">
              {{ tag }}
            </el-tag>
          </div>
        </div>

        <!-- 快捷提问 -->
        <div class="quick-questions">
          <h4 class="section-title">
            <el-icon class="section-title-icon"><ChatDotSquare /></el-icon>
            快捷提问
          </h4>
          <div class="question-buttons">
            <el-button
              v-for="q in quickQuestions"
              :key="q.text"
              class="question-btn"
              @click="sendQuickQuestion(q.text)"
            >
              {{ q.text }}
            </el-button>
          </div>
        </div>
      </aside>

      <!-- 右侧AI对话区 -->
      <section class="chat-panel">
        <div class="chat-messages" ref="chatMessagesRef">
          <div v-for="(msg, index) in chatMessages" :key="index" :class="['message', msg.role]">
            <div class="message-avatar">
              <el-icon v-if="msg.role === 'ai'" :size="20"><Service /></el-icon>
              <el-icon v-else :size="20"><UserFilled /></el-icon>
            </div>
            <div class="message-content">
              <div class="message-text" v-html="formatMessage(msg.text)"></div>
            </div>
          </div>
        </div>
        <div class="chat-input">
          <el-input
            v-model="inputMessage"
            placeholder="输入你的问题..."
            @keyup.enter="sendMessage"
            class="input-field"
          >
            <template #append>
              <el-button @click="sendMessage" type="primary" class="send-btn">发送</el-button>
            </template>
          </el-input>
        </div>
      </section>
    </main>

    <!-- 底部提醒列表 -->
    <section class="reminder-list-section">
      <h3 class="section-title">
        <el-icon class="section-title-icon"><Bell /></el-icon>
        我的活动提醒
      </h3>
      <el-table :data="reminderList" stripe style="width: 100%" class="reminder-table">
        <el-table-column prop="activityName" label="活动名称" width="200">
          <template #default="{ row }">
            {{ row.activityName }}
            <el-tag v-if="!row.isDbActivity" size="small" type="info" class="oral-tag">口头</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="eventTime" label="活动时间" width="140" />
        <el-table-column prop="remindAt" label="提醒时间" width="140" />
        <el-table-column prop="status" label="状态" width="100">
          <template #default="{ row }">
            <el-tag :type="row.isReminded ? 'info' : 'success'" size="small">
              {{ row.isReminded ? '已提醒' : '待提醒' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="100">
          <template #default="{ row }">
            <el-button type="danger" size="small" @click="cancelReminder(row.id)">取消</el-button>
          </template>
        </el-table-column>
      </el-table>
    </section>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import {
  Reading,
  Notification,
  Clock,
  Bell,
  TrendCharts,
  Collection,
  Service,
  UserFilled,
  ChatDotSquare,
  StarFilled,
  SwitchButton
} from '@element-plus/icons-vue'

const router = useRouter()

// 退出登录
const handleLogout = () => {
  localStorage.removeItem('studentInfo')
  router.push('/')
  ElMessage.info('已退出登录')
}

// 从 localStorage 加载用户数据（由登录页保存）
const loadUserInfo = () => {
  const info = localStorage.getItem('studentInfo')
  return info ? JSON.parse(info) : null
}

const userInfo = loadUserInfo() || {}

const currentUser = computed(() => {
  return {
    id: userInfo.studentId || '2024001',
    name: userInfo.name || '学生',
    grade: userInfo.grade || '大二',
    major: '计科',
    scoreA: userInfo.scoreA ?? 65,
    scoreB: userInfo.scoreB ?? 50,
    scoreC: userInfo.scoreC ?? 12,
    scoreD: userInfo.scoreD ?? 20,
    totalScore: userInfo.totalScore || 43.15,
    targetScore: 100,
    interests: userInfo.interests || ['D类实践', 'B类竞赛'],
    role: userInfo.role || 1
  }
})

// 分数进度条
const scoreItems = computed(() => [
  { name: 'A 思想品德', current: currentUser.value.scoreA, color: '#F39C12' },
  { name: 'B 科学文化', current: currentUser.value.scoreB, color: '#27AE60' },
  { name: 'C 身体心理', current: currentUser.value.scoreC, color: '#3498DB' },
  { name: 'D 劳动实践', current: currentUser.value.scoreD, color: '#E67E22' }
])

const API_BASE = 'http://localhost:8000/api'

// 统计卡片数据
const stats = ref({
  newCount: 0,
  newList: [],
  deadlineCount: 0,
  deadlineList: [],
  recommendPriority: 'B',
  recommendReason: '',
  recommendList: [],
  reminderCount: 0,
  reminderList: []
})

// 提醒列表
const reminderList = ref([])

// 从后端加载统计数据
const loadStats = async () => {
  try {
    const studentId = currentUser.value.id
    // 加载统计概览
    const overviewRes = await fetch(`${API_BASE}/stats/overview?student_id=${studentId}`)
    if (overviewRes.ok) {
      const overview = await overviewRes.json()
      stats.value = {
        newCount: overview.cards.recent_count,
        newList: overview.cards.recent_items.map((item, i) => ({ id: i, title: item.title, category: item.category, score: item.score_value })),
        deadlineCount: overview.cards.upcoming_count,
        deadlineList: overview.cards.upcoming_items.map((item, i) => ({
          id: i,
          title: item.title,
          category: overview.cards.suggestion_category,
          score: item.score_value || 5,
          hoursLeft: item.hours_left
        })),
        recommendPriority: overview.cards.suggestion_category,
        recommendReason: `${overview.cards.suggestion_category}类当前分数较低，建议优先提升`,
        recommendList: overview.cards.suggestion_items.map((item, i) => ({ id: i, title: item.title, category: overview.cards.suggestion_category, score: item.score_value })),
        reminderCount: overview.cards.reminder_count,
        reminderList: overview.cards.reminder_items.map((item, i) => ({ id: i, name: item.activity_name, eventTime: formatEventTime(item.event_time), isOral: false }))
      }
    }

    // 加载提醒列表
    const reminderRes = await fetch(`${API_BASE}/tools/reminders?student_id=${studentId}`)
    if (reminderRes.ok) {
      const reminders = await reminderRes.json()
      reminderList.value = reminders.map(r => ({
        id: r.id,
        activityName: r.activity_name,
        eventTime: formatEventTime(r.event_time),
        remindAt: formatEventTime(r.remind_at),
        isDbActivity: true,
        isReminded: r.is_reminded
      }))
    }
  } catch (err) {
    console.error('加载统计数据失败:', err)
  }
}

// 快捷提问
const quickQuestions = [
  { text: '帮我规划' },
  { text: '查D类活动' },
  { text: '我的分数' },
  { text: '综测规则' },
  { text: '设置提醒' }
]

// 聊天消息
const chatMessages = ref([
  { role: 'ai', text: `你好${currentUser.value.name}！你目前总分${currentUser.value.totalScore}，${currentUser.value.scoreB}类还是基础50分，建议优先提升B类。有什么需要帮忙的吗？` }
])

const inputMessage = ref('')
const chatMessagesRef = ref(null)

// 对话ID（用于保持会话上下文）
const conversationId = ref('')

// 发送消息
const sendMessage = async () => {
  if (!inputMessage.value.trim()) return

  const userMsg = inputMessage.value
  chatMessages.value.push({ role: 'user', text: userMsg })
  inputMessage.value = ''
  scrollToBottom()

  // 调用真实 Dify 接口
  try {
    const studentId = currentUser.value.id
    const res = await fetch(`${API_BASE}/dify/chat`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        query: userMsg,
        student_id: studentId,
        student_name: currentUser.value.name,
        grade: currentUser.value.grade,
        score_a: currentUser.value.scoreA,
        score_b: currentUser.value.scoreB,
        score_c: currentUser.value.scoreC,
        score_d: currentUser.value.scoreD,
        total_score: currentUser.value.totalScore,
        target_score: currentUser.value.targetScore,
        intersets: currentUser.value.interests.join(','),
        conversation_id: conversationId.value || '',
      }),
    })

    if (!res.ok) {
      const errorData = await res.json().catch(() => ({}))
      throw new Error(errorData.detail || `请求失败: ${res.status}`)
    }

    const data = await res.json()
    conversationId.value = data.conversation_id
    chatMessages.value.push({ role: 'ai', text: data.answer })
  } catch (err) {
    console.error('Dify 调用失败:', err)
    chatMessages.value.push({
      role: 'ai',
      text: `抱歉，智能体暂时无法响应：${err.message}`,
    })
  }

  scrollToBottom()
}

// 快捷提问
const sendQuickQuestion = (text) => {
  inputMessage.value = text
  sendMessage()
}

// 滚动到底部
const scrollToBottom = () => {
  nextTick(() => {
    if (chatMessagesRef.value) {
      chatMessagesRef.value.scrollTop = chatMessagesRef.value.scrollHeight
    }
  })
}

// 截止着色
const getDeadlineColor = (hours) => {
  if (hours < 12) return '#E74C3C'
  if (hours < 24) return '#E67E22'
  return '#333'
}

// 格式化消息（简单换行处理）
const formatMessage = (text) => {
  return text.replace(/\n/g, '<br>')
}
const cancelReminder = async (id) => {
  try {
    const res = await fetch(`${API_BASE}/tools/reminders/${id}`, { method: 'DELETE' })
    if (res.ok) {
      reminderList.value = reminderList.value.filter(r => r.id !== id)
      stats.value.reminderCount = reminderList.value.length
      stats.value.reminderList = reminderList.value.slice(0, 2).map(r => ({
        id: r.id,
        name: r.activityName,
        eventTime: r.eventTime,
        isOral: !r.isDbActivity
      }))
      ElMessage.success('已取消提醒')
    }
  } catch (err) {
    console.error('取消提醒失败:', err)
    ElMessage.error('取消提醒失败')
  }
}

// 格式化时间
const formatEventTime = (isoStr) => {
  if (!isoStr) return ''
  const d = new Date(isoStr)
  return `${String(d.getMonth() + 1).padStart(2, '0')}/${String(d.getDate()).padStart(2, '0')} ${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`
}

onMounted(() => {
  loadStats()
})
</script>

<style scoped>
.app-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
}

/* ===== 顶部导航 ===== */
.top-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 28px;
  background: #1e3a5f;
  border-radius: 10px;
  margin-bottom: 20px;
  box-shadow: 0 2px 12px rgba(30, 58, 95, 0.15);
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.12);
  border-radius: 8px;
  color: #ffffff;
}

.app-title {
  color: #ffffff;
  font-size: 22px;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.nav-right {
  display: flex;
  align-items: center;
  gap: 16px;
}

.user-info-nav {
  display: flex;
  align-items: center;
  gap: 8px;
  color: rgba(255, 255, 255, 0.85);
  font-size: 14px;
}

.logout-btn {
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
}

.logout-btn:hover {
  color: #ffffff;
}

.student-select {
  width: 180px;
}

.student-select :deep(.el-input__wrapper) {
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 8px;
  box-shadow: none;
}

.student-select :deep(.el-input__wrapper:hover) {
  border-color: rgba(255, 255, 255, 0.3);
}

.student-select :deep(.el-input__inner) {
  color: #ffffff;
}

.student-select :deep(.el-input__inner::placeholder) {
  color: rgba(255, 255, 255, 0.5);
}

/* ===== 统计卡片 ===== */
.stats-cards {
  margin-bottom: 20px;
}

.card-row {
  margin: 0 -8px;
}

.card-col {
  padding: 0 8px;
  margin-bottom: 16px;
}

.stat-card {
  background: #ffffff;
  border-radius: 10px;
  padding: 22px 20px;
  min-height: 160px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.03);
  border: 1px solid #f0ece6;
  transition: box-shadow 0.25s ease, border-color 0.25s ease;
}

.stat-card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.07);
  border-color: #e2dcd2;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 14px;
}

.card-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 8px;
  flex-shrink: 0;
}

.card-icon.icon-new {
  background: #fdf2e4;
  color: #c8851a;
}

.card-icon.icon-deadline {
  background: #fde8e4;
  color: #c8513a;
}

.card-icon.icon-recommend {
  background: #e8f4ee;
  color: #3d8b63;
}

.card-icon.icon-reminder {
  background: #e4edf8;
  color: #4a7dba;
}

.card-title {
  font-size: 14px;
  color: #5a6d7e;
  font-weight: 500;
}

.card-big-text {
  font-size: 34px;
  font-weight: 700;
  color: #2c3e50;
  margin-bottom: 10px;
  letter-spacing: -0.5px;
  font-variant-numeric: tabular-nums;
}

.card-big-text .unit {
  font-size: 15px;
  font-weight: 400;
  color: #8b9aab;
  margin-left: 4px;
}

.card-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.card-list-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 10px;
  background: #faf9f7;
  border-radius: 6px;
  font-size: 13px;
  border-left: 3px solid transparent;
  transition: background 0.2s, border-color 0.2s;
}

.card-list-item:hover {
  background: #f5f2ed;
  border-left-color: #c8851a;
}

.card-list-item.clickable {
  cursor: pointer;
}

.item-name {
  color: #2c3e50;
  font-weight: 500;
}

.item-score {
  color: #5a6d7e;
  font-size: 12px;
}

.deadline-text[data-urgent="true"] {
  color: #c8513a;
  font-weight: 600;
}

.deadline-text[data-urgent="warn"] {
  color: #c8851a;
  font-weight: 500;
}

.card-empty {
  color: #8b9aab;
  font-size: 13px;
  text-align: center;
  padding: 12px 0;
}

.recommend-priority {
  font-size: 18px;
  font-weight: 700;
  color: #3d8b63;
  margin-bottom: 4px;
}

.recommend-reason {
  font-size: 12px;
  color: #5a6d7e;
  margin-bottom: 14px;
  line-height: 1.6;
}

.oral-tag {
  margin-left: 6px;
}

.item-time {
  font-size: 12px;
  color: #8b9aab;
}

/* ===== 主体区域 ===== */
.main-content {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 20px;
  margin-bottom: 20px;
  height: 600px;
}

@media (max-width: 1024px) {
  .main-content {
    grid-template-columns: 1fr;
  }
}

/* ===== 左侧仪表盘 ===== */
.dashboard-panel {
  background: #ffffff;
  border-radius: 10px;
  padding: 24px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.03);
  border: 1px solid #f0ece6;
  height: 100%;
  overflow-y: auto;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 16px;
  padding-bottom: 20px;
  border-bottom: 1px solid #f0ece6;
  margin-bottom: 20px;
}

.user-avatar {
  flex-shrink: 0;
}

.avatar-custom {
  background: #1e3a5f !important;
  font-weight: 600;
  font-size: 24px;
}

.user-name {
  font-size: 20px;
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 4px;
}

.user-major {
  font-size: 14px;
  color: #8b9aab;
}

.score-section {
  margin-bottom: 20px;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 15px;
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 16px;
}

.section-title-icon {
  color: #5a6d7e;
}

.score-item {
  margin-bottom: 14px;
}

.score-label {
  display: flex;
  justify-content: space-between;
  margin-bottom: 6px;
}

.score-name {
  font-size: 13px;
  color: #5a6d7e;
  font-weight: 500;
}

.score-value {
  font-size: 14px;
  font-weight: 600;
  color: #2c3e50;
  font-variant-numeric: tabular-nums;
}

.score-max {
  font-weight: 400;
  color: #8b9aab;
  font-size: 12px;
}

.score-progress {
  height: 10px;
}

.score-progress :deep(.el-progress-bar__outer) {
  background: #f2efe9;
  border-radius: 5px;
  overflow: hidden;
}

.score-progress :deep(.el-progress-bar__inner) {
  border-radius: 5px;
}

.total-score-card {
  background: #faf9f7;
  border-radius: 8px;
  padding: 16px 18px;
  margin-top: 18px;
  border: 1px solid #f0ece6;
}

.total-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px 0;
}

.total-label {
  font-size: 14px;
  color: #5a6d7e;
}

.total-value {
  font-size: 20px;
  font-weight: 700;
  color: #2c3e50;
  font-variant-numeric: tabular-nums;
}

.total-value.target {
  color: #3d8b63;
}

.total-value.gap {
  color: #c8851a;
}

.gap-row {
  border-top: 1px solid #f0ece6;
  margin-top: 7px;
  padding-top: 7px;
}

.interest-section {
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid #f0ece6;
}

.tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.interest-tag {
  font-size: 13px;
  border-radius: 6px;
  padding: 4px 12px;
  background: #faf9f7;
  border-color: #e8e4db;
  color: #5a6d7e;
}

.quick-questions .section-title {
  margin-bottom: 12px;
}

.question-buttons {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.question-btn {
  width: 100%;
  text-align: left;
  justify-content: flex-start;
  padding: 10px 16px;
  border: 1px solid #e8e4db;
  border-radius: 8px;
  background: #fdfcfb;
  font-size: 13px;
  color: #5a6d7e;
  transition: all 0.2s;
}

.question-btn:hover {
  border-color: #c8851a;
  color: #c8851a;
  background: #fdf9f2;
}

/* ===== 右侧对话区 ===== */
.chat-panel {
  background: #ffffff;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.03);
  border: 1px solid #f0ece6;
  height: 100%;
  overflow: hidden;
}

.chat-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.chat-messages::-webkit-scrollbar {
  width: 6px;
}

.chat-messages::-webkit-scrollbar-thumb {
  background: #e8e4db;
  border-radius: 3px;
}

.chat-messages::-webkit-scrollbar-thumb:hover {
  background: #ccc6b8;
}

.message {
  display: flex;
  gap: 12px;
  max-width: 80%;
}

.message.ai {
  align-self: flex-start;
}

.message.user {
  align-self: flex-end;
  flex-direction: row-reverse;
}

.message-avatar {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.message.ai .message-avatar {
  background: #e8f0f8;
  color: #4a7dba;
}

.message.user .message-avatar {
  background: #f2efe9;
  color: #5a6d7e;
}

.message-content {
  padding: 12px 16px;
  border-radius: 10px;
  line-height: 1.65;
  font-size: 14px;
}

.message.ai .message-content {
  background: #f8f7f5;
  color: #2c3e50;
  border-bottom-left-radius: 4px;
}

.message.user .message-content {
  background: #e8f0f8;
  color: #1e3a5f;
  border-bottom-right-radius: 4px;
}

.chat-input {
  padding: 16px 20px;
  border-top: 1px solid #f0ece6;
}

.chat-input :deep(.el-input__wrapper) {
  border-radius: 8px;
  padding: 8px 12px;
  box-shadow: none;
  border-color: #e8e4db;
}

.chat-input :deep(.el-input__wrapper:hover) {
  border-color: #ccc6b8;
}

.chat-input :deep(.el-input__wrapper.is-focus) {
  border-color: #1e3a5f;
  box-shadow: 0 0 0 1px rgba(30, 58, 95, 0.1);
}

.send-btn {
  background: #1e3a5f;
  border-color: #1e3a5f;
  border-radius: 0 8px 8px 0;
}

.send-btn:hover {
  background: #2a4f7a;
  border-color: #2a4f7a;
}

/* ===== 底部提醒列表 ===== */
.reminder-list-section {
  background: #ffffff;
  border-radius: 10px;
  padding: 24px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.03);
  border: 1px solid #f0ece6;
}

.reminder-list-section .section-title {
  margin-bottom: 16px;
}

.reminder-table {
  border-radius: 8px;
  overflow: hidden;
}

.reminder-table :deep(.el-table__header-wrapper th) {
  background: #faf9f7;
  font-weight: 600;
  color: #5a6d7e;
  border-bottom: 1px solid #f0ece6;
}

.reminder-table :deep(.el-table__body-wrapper td) {
  color: #2c3e50;
}

.reminder-table :deep(.el-table--striped .el-table__body tr.el-table__row--striped td) {
  background: #fdfcfb;
}
</style>
