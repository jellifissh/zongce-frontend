<template>
  <div class="app-shell">
    <header class="top-nav">
      <div class="logo-section">
        <span class="logo-icon">
          <el-icon :size="26"><Reading /></el-icon>
        </span>
        <div>
          <p class="app-kicker">Comprehensive Evaluation Agent</p>
          <h1 class="app-title">综测服务智能体</h1>
        </div>
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

    <section class="hero-panel">
      <div>
        <span class="hero-chip">当前总分 {{ currentUser.totalScore }}</span>
        <h2>今天优先补 {{ stats.recommendPriority || 'B' }} 类活动</h2>
        <p>系统会根据你的四维分数、截止时间和提醒记录做推荐。终于不是把学生扔进通知海里自由漂流了。</p>
      </div>
      <div class="hero-metrics">
        <div>
          <strong>{{ stats.deadlineCount }}</strong>
          <span>即将截止</span>
        </div>
        <div>
          <strong>{{ stats.reminderCount }}</strong>
          <span>待提醒</span>
        </div>
        <div>
          <strong>{{ currentUser.gap }}</strong>
          <span>目标缺口</span>
        </div>
      </div>
    </section>

    <section class="stats-grid">
      <article class="stat-card accent-new">
        <div class="card-header">
          <span class="card-icon icon-new"><el-icon :size="18"><Notification /></el-icon></span>
          <span class="card-title">近3日新增活动</span>
        </div>
        <div class="card-content">
          <div class="card-big-text">{{ stats.newCount }} <span class="unit">场</span></div>
          <div class="card-list" v-if="stats.newList.length > 0">
            <button class="card-list-item" v-for="item in stats.newList" :key="item.id" @click="sendQuickQuestion(`帮我看看${item.title}`)">
              <span class="item-name">{{ item.title }}</span>
              <span class="item-score">{{ item.category }}类 +{{ item.score }}</span>
            </button>
          </div>
          <div class="card-empty" v-else>暂无新活动</div>
        </div>
      </article>

      <article class="stat-card accent-deadline">
        <div class="card-header">
          <span class="card-icon icon-deadline"><el-icon :size="18"><Clock /></el-icon></span>
          <span class="card-title">即将截止活动</span>
        </div>
        <div class="card-content">
          <div class="card-big-text">{{ stats.deadlineCount }} <span class="unit">场</span></div>
          <div class="card-list" v-if="stats.deadlineList.length > 0">
            <button class="card-list-item" v-for="item in stats.deadlineList" :key="item.id" @click="sendQuickQuestion(`提醒我参加${item.title}`)">
              <span class="item-name">{{ item.title }}</span>
              <span class="item-score deadline-text" :style="{ color: getDeadlineColor(item.hoursLeft) }">
                {{ item.category }}类 +{{ item.score }} | {{ item.hoursLeft }}h
              </span>
            </button>
          </div>
          <div class="card-empty" v-else>暂无即将截止的活动</div>
        </div>
      </article>

      <article class="stat-card accent-recommend">
        <div class="card-header">
          <span class="card-icon icon-recommend"><el-icon :size="18"><StarFilled /></el-icon></span>
          <span class="card-title">综测活动建议</span>
        </div>
        <div class="card-content">
          <div class="recommend-priority">{{ stats.recommendPriority }}类优先</div>
          <div class="recommend-reason">{{ stats.recommendReason || '根据当前分数缺口，建议优先补低分高权重类别' }}</div>
          <div class="card-list" v-if="stats.recommendList.length > 0">
            <button class="card-list-item" v-for="item in stats.recommendList" :key="item.id" @click="sendQuickQuestion(`帮我看看${item.title}`)">
              <span class="item-name">{{ item.title }}</span>
              <span class="item-score">{{ item.category }}类 +{{ item.score }}</span>
            </button>
          </div>
          <div class="card-empty" v-else>你的综测分数已达标，继续保持！</div>
        </div>
      </article>

      <article class="stat-card accent-reminder">
        <div class="card-header">
          <span class="card-icon icon-reminder"><el-icon :size="18"><Bell /></el-icon></span>
          <span class="card-title">活动提醒</span>
        </div>
        <div class="card-content">
          <div class="card-big-text">{{ stats.reminderCount }} <span class="unit">条</span></div>
          <div class="card-list" v-if="stats.reminderList.length > 0">
            <button class="card-list-item" v-for="item in stats.reminderList" :key="item.id" @click="sendQuickQuestion(`查看${item.name}提醒`)">
              <span class="item-name">
                {{ item.name }}
                <el-tag v-if="item.isOral" size="small" type="info" class="oral-tag">口头</el-tag>
              </span>
              <span class="item-time">{{ item.eventTime }}</span>
            </button>
          </div>
          <div class="card-empty" v-else>暂无活动提醒，在对话中说“提醒我”即可</div>
        </div>
      </article>
    </section>

    <main class="main-content">
      <aside class="dashboard-panel">
        <div class="user-info">
          <el-avatar :size="62" class="avatar-custom">{{ currentUser.name?.charAt(0) }}</el-avatar>
          <div class="user-detail">
            <h3 class="user-name">{{ currentUser.name }}</h3>
            <p class="user-major">{{ currentUser.grade }} · {{ currentUser.major }}</p>
          </div>
        </div>

        <div class="score-section glass-block">
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

        <div class="interest-section glass-block">
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

        <div class="quick-questions glass-block">
          <h4 class="section-title">
            <el-icon class="section-title-icon"><ChatDotSquare /></el-icon>
            快捷提问
          </h4>
          <div class="question-buttons">
            <button
              v-for="q in quickQuestions"
              :key="q.text"
              class="question-btn"
              @click="sendQuickQuestion(q.text)"
            >
              {{ q.text }}
            </button>
          </div>
        </div>
      </aside>

      <section class="chat-panel">
        <div class="chat-header">
          <div>
            <h3>AI 综测顾问</h3>
            <p>可以直接问活动、规则、规划和提醒</p>
          </div>
          <span class="online-pill">在线</span>
        </div>

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

          <div class="chat-hints" v-if="chatMessages.length <= 1">
            <button v-for="q in quickQuestions.slice(0, 4)" :key="q.text" @click="sendQuickQuestion(q.text)">
              {{ q.text }}
            </button>
          </div>
        </div>

        <div class="chat-input">
          <el-input
            v-model="inputMessage"
            placeholder="输入你的问题，比如：这周末有没有 D 类活动？"
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

    <section class="reminder-list-section">
      <div class="table-title-row">
        <h3 class="section-title">
          <el-icon class="section-title-icon"><Bell /></el-icon>
          我的活动提醒
        </h3>
        <span>{{ reminderList.length }} 条记录</span>
      </div>
      <el-table :data="reminderList" stripe style="width: 100%" class="reminder-table">
        <el-table-column prop="activityName" label="活动名称" min-width="200">
          <template #default="{ row }">
            {{ row.activityName }}
            <el-tag v-if="!row.isDbActivity" size="small" type="info" class="oral-tag">口头</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="eventTime" label="活动时间" min-width="140" />
        <el-table-column prop="remindAt" label="提醒时间" min-width="140" />
        <el-table-column prop="status" label="状态" min-width="100">
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
const API_BASE = 'http://localhost:8000/api'

const handleLogout = () => {
  localStorage.removeItem('studentInfo')
  router.push('/')
  ElMessage.info('已退出登录')
}

const loadUserInfo = () => {
  const info = localStorage.getItem('studentInfo')
  return info ? JSON.parse(info) : null
}

const userInfo = loadUserInfo() || {}

const calculateTotal = (a, b, c, d) => {
  return (Number(a) * 0.35 + Number(b) * 0.30 + Number(c) * 0.20 + Number(d) * 0.15).toFixed(2)
}

const currentUser = computed(() => {
  const scoreA = Number(userInfo.scoreA ?? 65)
  const scoreB = Number(userInfo.scoreB ?? 50)
  const scoreC = Number(userInfo.scoreC ?? 12)
  const scoreD = Number(userInfo.scoreD ?? 20)
  const totalScore = Number(userInfo.totalScore ?? calculateTotal(scoreA, scoreB, scoreC, scoreD))
  const targetScore = Number(userInfo.targetScore ?? 100)

  return {
    id: userInfo.studentId || '2024001',
    name: userInfo.name || '学生',
    grade: userInfo.grade || '大二',
    major: '计科',
    scoreA,
    scoreB,
    scoreC,
    scoreD,
    totalScore: totalScore.toFixed(2),
    targetScore,
    gap: Math.max(targetScore - totalScore, 0).toFixed(2),
    interests: userInfo.interests?.length ? userInfo.interests : ['D类实践', 'B类竞赛'],
    role: userInfo.role || 1
  }
})

const scoreItems = computed(() => [
  { name: 'A 思想品德', current: currentUser.value.scoreA, color: '#F39C12' },
  { name: 'B 科学文化', current: currentUser.value.scoreB, color: '#27AE60' },
  { name: 'C 身体心理', current: currentUser.value.scoreC, color: '#3498DB' },
  { name: 'D 劳动实践', current: currentUser.value.scoreD, color: '#E67E22' }
])

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

const reminderList = ref([])

const normalizeItems = (items = [], fallbackCategory = 'B') => {
  return items.map((item, i) => ({
    id: item.id ?? i,
    title: item.title || item.activity_name || '未命名活动',
    category: item.category || fallbackCategory,
    score: item.score_value ?? item.score ?? 0,
    hoursLeft: item.hours_left ?? 0
  }))
}

const loadStats = async () => {
  try {
    const studentId = currentUser.value.id
    const overviewRes = await fetch(`${API_BASE}/stats/overview?student_id=${studentId}`)

    if (overviewRes.ok) {
      const overview = await overviewRes.json()
      const cards = overview.cards || overview
      const suggestionCategory = cards.suggestion_category || cards.recommendation?.priority_category || 'B'

      stats.value = {
        newCount: cards.recent_count ?? cards.new_activities_count ?? 0,
        newList: normalizeItems(cards.recent_items || cards.new_activities_detail || [], suggestionCategory),
        deadlineCount: cards.upcoming_count ?? cards.deadline_soon_count ?? 0,
        deadlineList: normalizeItems(cards.upcoming_items || cards.deadline_soon_detail || [], suggestionCategory),
        recommendPriority: suggestionCategory,
        recommendReason: cards.recommendation?.reason || `${suggestionCategory}类当前分数较低，建议优先提升`,
        recommendList: normalizeItems(cards.suggestion_items || cards.recommendation?.suggested_activities || [], suggestionCategory),
        reminderCount: cards.reminder_count ?? cards.pending_reminders_count ?? 0,
        reminderList: (cards.reminder_items || cards.pending_reminders_detail || []).map((item, i) => ({
          id: item.id ?? i,
          name: item.activity_name || item.title || '活动提醒',
          eventTime: formatEventTime(item.event_time),
          isOral: item.activity_id == null || item.is_db_activity === false
        }))
      }
    }

    const reminderRes = await fetch(`${API_BASE}/tools/reminders?student_id=${studentId}`)
    if (reminderRes.ok) {
      const reminders = await reminderRes.json()
      reminderList.value = reminders.map(r => ({
        id: r.id,
        activityName: r.activity_name,
        eventTime: formatEventTime(r.event_time),
        remindAt: formatEventTime(r.remind_at),
        isDbActivity: r.activity_id != null,
        isReminded: r.is_reminded
      }))
    }
  } catch (err) {
    console.error('加载统计数据失败:', err)
  }
}

const quickQuestions = [
  { text: '帮我规划' },
  { text: '查D类活动' },
  { text: '我的分数' },
  { text: '综测规则' },
  { text: '设置提醒' }
]

const chatMessages = ref([
  {
    role: 'ai',
    text: `你好 ${currentUser.value.name}！你目前总分 ${currentUser.value.totalScore}，B 类当前 ${currentUser.value.scoreB} 分，建议优先提升 B 类。有什么需要帮忙的吗？`
  }
])

const inputMessage = ref('')
const chatMessagesRef = ref(null)
const conversationId = ref('')

const sendMessage = async () => {
  if (!inputMessage.value.trim()) return

  const userMsg = inputMessage.value
  chatMessages.value.push({ role: 'user', text: userMsg })
  inputMessage.value = ''
  scrollToBottom()

  try {
    const studentId = currentUser.value.id
    const interests = currentUser.value.interests.join(',')
    const res = await fetch(`${API_BASE}/dify/chat`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
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
        interests,
        intersets: interests,
        conversation_id: conversationId.value || ''
      })
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
      text: `抱歉，智能体暂时无法响应：${err.message}`
    })
  }

  scrollToBottom()
}

const sendQuickQuestion = (text) => {
  inputMessage.value = text
  sendMessage()
}

const scrollToBottom = () => {
  nextTick(() => {
    if (chatMessagesRef.value) {
      chatMessagesRef.value.scrollTop = chatMessagesRef.value.scrollHeight
    }
  })
}

const getDeadlineColor = (hours) => {
  if (hours < 12) return '#E74C3C'
  if (hours < 24) return '#E67E22'
  return '#51667f'
}

const formatMessage = (text = '') => {
  return String(text).replace(/\n/g, '<br>')
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
.app-shell {
  width: min(1480px, calc(100% - 48px));
  margin: 0 auto;
  padding: 22px 0 34px;
  min-height: 100vh;
  position: relative;
}

.app-shell::before {
  content: '';
  position: fixed;
  inset: 0;
  z-index: -1;
  background:
    radial-gradient(circle at 10% 8%, rgba(47, 111, 165, 0.14), transparent 26%),
    radial-gradient(circle at 88% 14%, rgba(46, 166, 125, 0.13), transparent 24%),
    linear-gradient(180deg, #f4f8fc 0%, #eef3f8 46%, #f8fafc 100%);
}

.top-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 28px;
  background: linear-gradient(135deg, #14345b, #235d91);
  border-radius: 22px;
  margin-bottom: 18px;
  box-shadow: 0 18px 40px rgba(20, 52, 91, 0.22);
  border: 1px solid rgba(255, 255, 255, 0.14);
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 14px;
}

.logo-icon {
  display: grid;
  place-items: center;
  width: 46px;
  height: 46px;
  background: rgba(255, 255, 255, 0.14);
  border-radius: 16px;
  color: #ffffff;
}

.app-kicker {
  color: rgba(255, 255, 255, 0.56);
  font-size: 12px;
  letter-spacing: 0.8px;
  margin-bottom: 2px;
}

.app-title {
  color: #ffffff;
  font-size: 23px;
  font-weight: 800;
  letter-spacing: 0.5px;
}

.nav-right {
  display: flex;
  align-items: center;
  gap: 14px;
}

.user-info-nav {
  display: flex;
  align-items: center;
  gap: 8px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
}

.logout-btn {
  color: rgba(255, 255, 255, 0.74);
  border-radius: 999px;
}

.logout-btn:hover {
  color: #ffffff;
  background: rgba(255, 255, 255, 0.1);
}

.hero-panel {
  display: flex;
  justify-content: space-between;
  gap: 22px;
  align-items: center;
  padding: 20px 24px;
  margin-bottom: 18px;
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.78);
  border: 1px solid rgba(210, 222, 236, 0.8);
  box-shadow: 0 14px 32px rgba(31, 65, 101, 0.08);
  backdrop-filter: blur(12px);
}

.hero-chip {
  display: inline-flex;
  width: fit-content;
  margin-bottom: 8px;
  padding: 6px 12px;
  border-radius: 999px;
  color: #1e5a92;
  background: #e8f3ff;
  font-size: 13px;
  font-weight: 800;
}

.hero-panel h2 {
  color: #17375f;
  font-size: 24px;
  margin-bottom: 6px;
}

.hero-panel p {
  color: #667f99;
  line-height: 1.7;
  max-width: 680px;
}

.hero-metrics {
  display: grid;
  grid-template-columns: repeat(3, minmax(96px, 1fr));
  gap: 10px;
}

.hero-metrics div {
  min-width: 98px;
  padding: 14px;
  border-radius: 18px;
  background: linear-gradient(180deg, #ffffff, #f4f8fc);
  border: 1px solid #e3edf7;
  text-align: center;
}

.hero-metrics strong {
  display: block;
  color: #17375f;
  font-size: 22px;
  line-height: 1;
  margin-bottom: 6px;
}

.hero-metrics span {
  color: #7d91a8;
  font-size: 12px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 16px;
  margin-bottom: 18px;
}

.stat-card {
  position: relative;
  overflow: hidden;
  min-height: 178px;
  padding: 20px;
  border-radius: 22px;
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(218, 229, 241, 0.9);
  box-shadow: 0 14px 32px rgba(31, 65, 101, 0.08);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.stat-card::after {
  content: '';
  position: absolute;
  width: 120px;
  height: 120px;
  right: -50px;
  top: -54px;
  border-radius: 999px;
  opacity: 0.16;
}

.accent-new::after { background: #f2a93b; }
.accent-deadline::after { background: #ef6b58; }
.accent-recommend::after { background: #34a66f; }
.accent-reminder::after { background: #4d8ed8; }

.stat-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 20px 42px rgba(31, 65, 101, 0.13);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 14px;
  position: relative;
  z-index: 1;
}

.card-icon {
  display: grid;
  place-items: center;
  width: 36px;
  height: 36px;
  border-radius: 13px;
  flex-shrink: 0;
}

.icon-new { background: #fff2df; color: #cf7e0f; }
.icon-deadline { background: #ffe8e4; color: #d35442; }
.icon-recommend { background: #e7f6ee; color: #2f9961; }
.icon-reminder { background: #e8f2ff; color: #367aca; }

.card-title {
  color: #51667f;
  font-size: 14px;
  font-weight: 800;
}

.card-big-text {
  color: #17375f;
  font-size: 34px;
  font-weight: 900;
  letter-spacing: -1px;
  margin-bottom: 10px;
  font-variant-numeric: tabular-nums;
}

.unit {
  font-size: 15px;
  color: #8598aa;
  font-weight: 600;
  margin-left: 3px;
}

.card-list {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.card-list-item {
  width: 100%;
  border: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  padding: 9px 11px;
  border-radius: 12px;
  background: #f6f9fc;
  color: inherit;
  text-align: left;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.2s ease;
}

.card-list-item:hover {
  background: #eef5fd;
  transform: translateX(2px);
}

.item-name {
  color: #263d55;
  font-size: 13px;
  font-weight: 700;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.item-score,
.item-time {
  color: #657b93;
  font-size: 12px;
  white-space: nowrap;
}

.card-empty {
  display: grid;
  place-items: center;
  min-height: 58px;
  color: #92a4b6;
  font-size: 13px;
  border-radius: 14px;
  background: #f7fafc;
}

.recommend-priority {
  color: #24885a;
  font-size: 20px;
  font-weight: 900;
  margin-bottom: 4px;
}

.recommend-reason {
  min-height: 34px;
  color: #667f99;
  font-size: 12px;
  line-height: 1.55;
  margin-bottom: 10px;
}

.oral-tag {
  margin-left: 6px;
}

.main-content {
  display: grid;
  grid-template-columns: 330px minmax(0, 1fr);
  gap: 18px;
  margin-bottom: 18px;
  min-height: 590px;
}

.dashboard-panel,
.chat-panel,
.reminder-list-section {
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(218, 229, 241, 0.9);
  box-shadow: 0 14px 32px rgba(31, 65, 101, 0.08);
}

.dashboard-panel {
  padding: 20px;
  overflow: auto;
}

.dashboard-panel::-webkit-scrollbar,
.chat-messages::-webkit-scrollbar {
  width: 6px;
}

.dashboard-panel::-webkit-scrollbar-thumb,
.chat-messages::-webkit-scrollbar-thumb {
  background: rgba(23, 55, 95, 0.16);
  border-radius: 999px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-bottom: 16px;
  padding: 16px;
  border-radius: 20px;
  background: linear-gradient(135deg, #17375f, #2d6c9f);
  color: #ffffff;
}

.avatar-custom {
  background: rgba(255, 255, 255, 0.16) !important;
  color: #ffffff !important;
  font-weight: 900;
  font-size: 22px;
}

.user-name {
  font-size: 20px;
  font-weight: 900;
  margin-bottom: 4px;
}

.user-major {
  color: rgba(255, 255, 255, 0.72);
  font-size: 13px;
}

.glass-block {
  padding: 16px;
  border-radius: 20px;
  background: #f8fbfe;
  border: 1px solid #e5eef8;
  margin-bottom: 14px;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #253c55;
  font-size: 15px;
  font-weight: 900;
  margin-bottom: 14px;
}

.section-title-icon {
  color: #416f9f;
}

.score-item {
  margin-bottom: 13px;
}

.score-label {
  display: flex;
  justify-content: space-between;
  margin-bottom: 6px;
}

.score-name {
  color: #536a84;
  font-size: 13px;
  font-weight: 700;
}

.score-value {
  color: #263d55;
  font-size: 14px;
  font-weight: 900;
  font-variant-numeric: tabular-nums;
}

.score-max {
  color: #8799aa;
  font-size: 12px;
  font-weight: 500;
}

.score-progress :deep(.el-progress-bar__outer) {
  background: #edf2f7;
  border-radius: 999px;
}

.score-progress :deep(.el-progress-bar__inner) {
  border-radius: 999px;
}

.total-score-card {
  padding: 14px;
  border-radius: 16px;
  background: #ffffff;
  border: 1px solid #e5eef8;
}

.total-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 0;
}

.total-label {
  color: #657b93;
  font-size: 14px;
}

.total-value {
  color: #17375f;
  font-size: 20px;
  font-weight: 900;
  font-variant-numeric: tabular-nums;
}

.total-value.target { color: #24885a; }
.total-value.gap { color: #d98910; }

.gap-row {
  border-top: 1px solid #edf2f7;
  margin-top: 6px;
  padding-top: 10px;
}

.tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.interest-tag {
  border-radius: 999px;
  padding: 5px 12px;
  background: #ffffff;
  border-color: #dbe8f5;
  color: #41627f;
  font-weight: 700;
}

.question-buttons {
  display: grid;
  grid-template-columns: 1fr;
  gap: 8px;
}

.question-btn,
.chat-hints button {
  border: 1px solid #dbe8f5;
  background: #ffffff;
  border-radius: 14px;
  padding: 10px 12px;
  color: #48637f;
  font-weight: 700;
  text-align: left;
  cursor: pointer;
  transition: all 0.2s ease;
}

.question-btn:hover,
.chat-hints button:hover {
  border-color: #2d6c9f;
  color: #1e5a92;
  background: #eef6ff;
  transform: translateX(2px);
}

.chat-panel {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 22px;
  border-bottom: 1px solid #e5eef8;
  background: linear-gradient(180deg, #ffffff, #f8fbfe);
}

.chat-header h3 {
  color: #17375f;
  font-size: 18px;
  font-weight: 900;
  margin-bottom: 4px;
}

.chat-header p {
  color: #7f94aa;
  font-size: 13px;
}

.online-pill {
  padding: 7px 12px;
  border-radius: 999px;
  color: #1f8d58;
  background: #e7f6ee;
  font-size: 12px;
  font-weight: 900;
}

.chat-messages {
  flex: 1;
  padding: 22px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
  background:
    radial-gradient(circle at 12% 10%, rgba(77, 142, 216, 0.08), transparent 24%),
    #ffffff;
}

.message {
  display: flex;
  gap: 12px;
  max-width: 78%;
}

.message.ai { align-self: flex-start; }
.message.user {
  align-self: flex-end;
  flex-direction: row-reverse;
}

.message-avatar {
  width: 38px;
  height: 38px;
  border-radius: 14px;
  display: grid;
  place-items: center;
  flex-shrink: 0;
}

.message.ai .message-avatar {
  background: #e8f3ff;
  color: #367aca;
}

.message.user .message-avatar {
  background: #17375f;
  color: #ffffff;
}

.message-content {
  padding: 12px 16px;
  border-radius: 16px;
  line-height: 1.7;
  font-size: 14px;
  box-shadow: 0 8px 22px rgba(31, 65, 101, 0.07);
}

.message.ai .message-content {
  color: #263d55;
  background: #f5f8fb;
  border-bottom-left-radius: 6px;
}

.message.user .message-content {
  color: #ffffff;
  background: linear-gradient(135deg, #17375f, #2d6c9f);
  border-bottom-right-radius: 6px;
}

.chat-hints {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-left: 50px;
  margin-top: 4px;
}

.chat-hints button {
  width: auto;
}

.chat-input {
  padding: 16px 18px;
  border-top: 1px solid #e5eef8;
  background: #ffffff;
}

.chat-input :deep(.el-input__wrapper) {
  min-height: 44px;
  border-radius: 14px 0 0 14px;
  box-shadow: none;
  border: 1px solid #dbe8f5;
}

.chat-input :deep(.el-input__wrapper:hover),
.chat-input :deep(.el-input__wrapper.is-focus) {
  border-color: #2d6c9f;
  box-shadow: 0 0 0 4px rgba(45, 108, 159, 0.1);
}

.send-btn {
  min-height: 44px;
  padding: 0 20px;
  border: none;
  border-radius: 0 14px 14px 0;
  background: linear-gradient(135deg, #17375f, #2d6c9f);
  font-weight: 800;
}

.reminder-list-section {
  padding: 20px;
}

.table-title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.table-title-row .section-title {
  margin-bottom: 0;
}

.table-title-row span {
  color: #7f94aa;
  font-size: 13px;
}

.reminder-table {
  border-radius: 16px;
  overflow: hidden;
}

.reminder-table :deep(.el-table__header-wrapper th) {
  background: #f4f8fc;
  color: #51667f;
  font-weight: 900;
  border-bottom: 1px solid #e5eef8;
}

.reminder-table :deep(.el-table__body-wrapper td) {
  color: #263d55;
}

.reminder-table :deep(.el-table--striped .el-table__body tr.el-table__row--striped td) {
  background: #f8fbfe;
}

@media (max-width: 1180px) {
  .stats-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .hero-panel {
    align-items: stretch;
    flex-direction: column;
  }
}

@media (max-width: 980px) {
  .app-shell {
    width: min(100% - 28px, 1480px);
  }

  .main-content {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 680px) {
  .top-nav,
  .nav-right,
  .hero-metrics,
  .stats-grid {
    grid-template-columns: 1fr;
  }

  .top-nav,
  .hero-panel {
    flex-direction: column;
    align-items: stretch;
  }

  .message {
    max-width: 94%;
  }
}
</style>
