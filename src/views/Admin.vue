<template>
  <div class="admin-container">
    <!-- 顶部导航栏 -->
    <header class="admin-nav">
      <div class="nav-left">
        <span class="nav-icon">‍💼</span>
        <h1 class="nav-title">综测服务智能体 · 管理端</h1>
      </div>
      <div class="nav-right">
        <el-tag type="info" effect="plain" class="admin-tag">管理员</el-tag>
        <el-button text class="logout-btn" @click="handleLogout">
          <el-icon><SwitchButton /></el-icon>
          退出登录
        </el-button>
      </div>
    </header>

    <!-- Tabs 区域 -->
    <section class="tabs-section">
      <el-tabs v-model="activeTab" class="admin-tabs" @tab-change="onTabChange">
        <!-- ===== 待审核 Tab ===== -->
        <el-tab-pane label="待审核活动" name="pending">
          <el-table :data="pendingActivities" stripe class="admin-table" v-loading="loading">
            <el-table-column prop="id" label="ID" width="70" />
            <el-table-column prop="title" label="活动名称" min-width="160" show-overflow-tooltip />
            <el-table-column prop="category" label="类别" width="80">
              <template #default="{ row }">
                <el-tag :type="getCategoryType(row.category)" size="small">
                  {{ row.category }}类
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="scoreValue" label="分值" width="70" align="center">
              <template #default="{ row }">
                <span class="score-badge">+{{ row.scoreValue }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="organizer" label="发起单位" width="120" show-overflow-tooltip />
            <el-table-column prop="targetGrade" label="面向年级" width="110" show-overflow-tooltip />
            <el-table-column prop="capacity" label="人数" width="70" align="center" />
            <el-table-column prop="location" label="活动地点" width="120" show-overflow-tooltip />
            <el-table-column prop="description" label="活动介绍" min-width="140" show-overflow-tooltip />
            <el-table-column prop="notes" label="注意事项" min-width="120" show-overflow-tooltip />
            <el-table-column prop="submitter" label="发布人" width="100" />
            <el-table-column prop="submitTime" label="提交时间" width="150" />
            <el-table-column label="操作" width="150" fixed="right">
              <template #default="{ row }">
                <el-popconfirm title="确定通过该活动吗？" @confirm="handleApprove(row)">
                  <template #reference>
                    <el-button type="success" link size="small">通过</el-button>
                  </template>
                </el-popconfirm>
                <el-button type="danger" link size="small" @click="openRejectDialog(row)">驳回</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- ===== 全部活动 Tab ===== -->
        <el-tab-pane label="全部活动" name="all">
          <!-- 筛选栏 -->
          <div class="filter-bar">
            <el-input
              v-model="searchForm.keyword"
              placeholder="搜索活动名称..."
              clearable
              class="search-input"
              @clear="handleSearch"
              @keyup.enter="handleSearch"
            >
              <template #prefix>
                <el-icon><Search /></el-icon>
              </template>
            </el-input>

            <el-select v-model="searchForm.category" placeholder="活动类别" clearable @change="handleSearch">
              <el-option label="A类·思想品德" value="A" />
              <el-option label="B类·科学文化" value="B" />
              <el-option label="C类·身体心理" value="C" />
              <el-option label="D类·劳动实践" value="D" />
            </el-select>

            <el-select v-model="searchForm.status" placeholder="活动状态" clearable @change="handleSearch">
              <el-option label="待审核" value="待审核" />
              <el-option label="已通过" value="已通过" />
              <el-option label="已驳回" value="已驳回" />
              <el-option label="报名中" value="报名中" />
              <el-option label="进行中" value="进行中" />
              <el-option label="已结束" value="已结束" />
            </el-select>

            <el-button type="primary" @click="handleSearch">查询</el-button>
            <el-button @click="handleReset">重置</el-button>
          </div>

          <!-- 操作按钮 -->
          <div class="action-buttons">
            <el-button type="primary" @click="openAddDialog">
              <el-icon><Plus /></el-icon>
              新增活动
            </el-button>
            <el-button type="danger" :disabled="selectedIds.length === 0" @click="handleBatchDelete">
              <el-icon><Delete /></el-icon>
              批量删除 ({{ selectedIds.length }})
            </el-button>
            <span class="selected-info">已选 {{ selectedIds.length }} 项 | 共 {{ totalActivities }} 条</span>
          </div>

          <!-- 数据表格 -->
          <el-table
            :data="allActivities"
            stripe
            @selection-change="handleSelectionChange"
            class="admin-table"
            v-loading="loading"
          >
            <el-table-column type="selection" width="55" />
            <el-table-column prop="id" label="ID" width="80" />
            <el-table-column prop="title" label="活动名称" min-width="180" show-overflow-tooltip />
            <el-table-column prop="category" label="类别" width="100">
              <template #default="{ row }">
                <el-tag :type="getCategoryType(row.category)" size="small">
                  {{ row.category }}类
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="scoreValue" label="分值" width="80" align="center">
              <template #default="{ row }">
                <span class="score-badge">+{{ row.scoreValue }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="creditHours" label="学时" width="80" align="center" />
            <el-table-column prop="organizer" label="发起单位" width="140" show-overflow-tooltip />
            <el-table-column prop="current_status" label="状态" width="100">
              <template #default="{ row }">
                <el-tag :type="getFullStatusType(row.current_status)" size="small">
                  {{ row.current_status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="apply_end" label="申请截止" width="140" />
            <el-table-column label="操作" width="160" fixed="right">
              <template #default="{ row }">
                <el-button type="primary" link size="small" @click="openEditDialog(row)">编辑</el-button>
                <el-button type="danger" link size="small" @click="handleDelete(row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>

          <!-- 分页 -->
          <div class="pagination-wrapper">
            <el-pagination
              v-model:current-page="currentPage"
              v-model:page-size="pageSize"
              :page-sizes="[10, 20, 50]"
              :total="totalActivities"
              layout="total, sizes, prev, pager, next"
              background
              @current-change="loadActivities"
              @size-change="loadActivities"
            />
          </div>
        </el-tab-pane>
      </el-tabs>
    </section>

    <!-- 新增/编辑对话框 -->
    <el-dialog
      v-model="dialogVisible"
      :title="isEdit ? '编辑活动' : '新增活动'"
      width="680px"
      :close-on-click-modal="false"
    >
      <el-form
        ref="formRef"
        :model="activityForm"
        :rules="activityRules"
        label-width="100px"
        class="activity-form"
      >
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="活动名称" prop="title">
              <el-input v-model="activityForm.title" placeholder="请输入活动名称" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="活动类别" prop="category">
              <el-select v-model="activityForm.category" placeholder="请选择类别" style="width: 100%">
                <el-option label="A类·思想品德" value="A" />
                <el-option label="B类·科学文化" value="B" />
                <el-option label="C类·身体心理" value="C" />
                <el-option label="D类·劳动实践" value="D" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="发起单位" prop="organizer">
              <el-input v-model="activityForm.organizer" placeholder="请输入发起单位" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="分值" prop="scoreValue">
              <el-input-number v-model="activityForm.scoreValue" :min="1" :max="20" style="width: 100%" />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="面向年级" prop="targetGrade">
              <el-select v-model="activityForm.targetGrade" placeholder="请选择（可多选）" multiple style="width: 100%">
                <el-option label="大一" value="大一" />
                <el-option label="大二" value="大二" />
                <el-option label="大三" value="大三" />
                <el-option label="大四" value="大四" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="学分" prop="creditHours">
              <el-input-number v-model="activityForm.creditHours" :min="0" :max="10" style="width: 100%" />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="报名开始" prop="applyStart">
              <el-date-picker
                v-model="activityForm.applyStart"
                type="datetime"
                placeholder="选择报名开始时间"
                style="width: 100%"
                value-format="YYYY-MM-DDTHH:mm:ss"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="报名截止" prop="applyEnd">
              <el-date-picker
                v-model="activityForm.applyEnd"
                type="datetime"
                placeholder="选择报名截止时间"
                style="width: 100%"
                value-format="YYYY-MM-DDTHH:mm:ss"
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="活动开始" prop="eventStart">
              <el-date-picker
                v-model="activityForm.eventStart"
                type="datetime"
                placeholder="选择活动开始时间"
                style="width: 100%"
                value-format="YYYY-MM-DDTHH:mm:ss"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="活动结束" prop="eventEnd">
              <el-date-picker
                v-model="activityForm.eventEnd"
                type="datetime"
                placeholder="选择活动结束时间"
                style="width: 100%"
                value-format="YYYY-MM-DDTHH:mm:ss"
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="活动人数" prop="capacity">
              <el-input-number v-model="activityForm.capacity" :min="1" style="width: 100%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="活动地点" prop="location">
              <el-input v-model="activityForm.location" placeholder="请输入活动地点" />
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="活动介绍" prop="description">
          <el-input
            v-model="activityForm.description"
            type="textarea"
            :rows="3"
            placeholder="请输入活动介绍"
          />
        </el-form-item>

        <el-form-item label="注意事项" prop="notes">
          <el-input
            v-model="activityForm.notes"
            type="textarea"
            :rows="2"
            placeholder="请输入注意事项"
          />
        </el-form-item>
      </el-form>

      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleSubmit" :loading="submitting">
          {{ isEdit ? '保存修改' : '确认新增' }}
        </el-button>
      </template>
    </el-dialog>

    <!-- 驳回原因对话框 -->
    <el-dialog v-model="rejectDialogVisible" title="驳回原因" width="480px">
      <el-input
        v-model="rejectReason"
        type="textarea"
        :rows="3"
        placeholder="请输入驳回原因"
      />
      <template #footer>
        <el-button @click="rejectDialogVisible = false">取消</el-button>
        <el-button type="danger" @click="handleReject">确认驳回</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Search,
  Plus,
  Delete,
  SwitchButton
} from '@element-plus/icons-vue'

const router = useRouter()

const API_BASE = 'http://localhost:8000/api'
const currentUser = JSON.parse(localStorage.getItem('studentInfo') || '{}')

// Tab 切换
const activeTab = ref('pending')

// 搜索表单
const searchForm = reactive({
  keyword: '',
  category: '',
  status: ''
})

// 数据
const pendingActivities = ref([])
const allActivities = ref([])
const totalActivities = ref(0)

// 分页
const currentPage = ref(1)
const pageSize = ref(20)

// 加载状态
const loading = ref(false)
const submitting = ref(false)

// 选中项
const selectedIds = ref([])

// 对话框
const dialogVisible = ref(false)
const isEdit = ref(false)
const formRef = ref(null)

// 驳回对话框
const rejectDialogVisible = ref(false)
const rejectReason = ref('')
const rejectTarget = ref(null)

// 表单数据
const activityForm = reactive({
  id: null,
  title: '',
  category: '',
  organizer: '',
  targetGrade: [],
  applyStart: '',
  applyEnd: '',
  eventStart: '',
  eventEnd: '',
  creditHours: 2,
  scoreValue: 5,
  location: '',
  description: '',
  notes: '',
  capacity: 50
})

// 表单验证规则
const activityRules = {
  title: [{ required: true, message: '请输入活动名称', trigger: 'blur' }],
  category: [{ required: true, message: '请选择活动类别', trigger: 'change' }],
  organizer: [{ required: true, message: '请输入发起单位', trigger: 'blur' }],
  scoreValue: [{ required: true, message: '请输入分值', trigger: 'blur' }]
}

// ===== API 调用 =====

// 加载待审核活动
const loadPending = async () => {
  try {
    loading.value = true
    const res = await fetch(`${API_BASE}/admin/activities/pending`)
    if (res.ok) {
      const data = await res.json()
      pendingActivities.value = data.map(item => ({
        id: item.id,
        title: item.title,
        category: item.category,
        scoreValue: item.score_value,
        organizer: item.organizer || '',
        targetGrade: item.target_grade || '',
        capacity: item.capacity || '',
        location: item.location || '',
        description: item.description || '',
        notes: item.notes || '',
        submitter: item.submitted_by || '',
        submitTime: item.created_at ? new Date(item.created_at).toLocaleString('zh-CN', { hour12: false }).replace(/\//g, '-') : ''
      }))
    }
    loading.value = false
  } catch (err) {
    loading.value = false
    console.error('加载待审核活动失败:', err)
  }
}

// 加载全部活动
const loadActivities = async () => {
  try {
    loading.value = true
    const params = new URLSearchParams({
      page: currentPage.value,
      page_size: pageSize.value
    })
    if (searchForm.keyword) params.append('keyword', searchForm.keyword)
    if (searchForm.category) params.append('category', searchForm.category)
    if (searchForm.status) params.append('status_filter', searchForm.status)

    const res = await fetch(`${API_BASE}/admin/activities?${params}`)
    if (res.ok) {
      const data = await res.json()
      allActivities.value = data.items.map(item => ({
        id: item.id,
        title: item.title,
        category: item.category,
        organizer: item.organizer || '',
        scoreValue: item.score_value,
        creditHours: item.credit_hours || 0,
        target_grade: item.target_grade || '',
        current_status: item.current_status,
        apply_end: item.apply_end ? new Date(item.apply_end).toLocaleString('zh-CN') : '',
        apply_start: item.apply_start,
        event_start: item.event_start,
        event_end: item.event_end,
        location: item.location || '',
        description: item.description || '',
        notes: item.notes || '',
        capacity: item.capacity || '',
        created_at: item.created_at
      }))
      totalActivities.value = data.total
    }
    loading.value = false
  } catch (err) {
    loading.value = false
    console.error('加载活动列表失败:', err)
  }
}

// Tab 切换
const onTabChange = (tab) => {
  if (tab === 'pending') loadPending()
  else loadActivities()
}

// 类别标签类型
const getCategoryType = (category) => {
  const map = { A: 'warning', B: 'success', C: 'primary', D: 'danger' }
  return map[category] || 'info'
}

// 状态标签类型
const getFullStatusType = (status) => {
  const map = {
    '待审核': 'info',
    '已通过': 'success',
    '已驳回': 'danger',
    '报名中': 'success',
    '进行中': 'warning',
    '已结束': 'info',
    '未开始': 'warning'
  }
  return map[status] || 'info'
}

// 搜索
const handleSearch = () => {
  currentPage.value = 1
  loadActivities()
}

// 重置
const handleReset = () => {
  searchForm.keyword = ''
  searchForm.category = ''
  searchForm.status = ''
  currentPage.value = 1
  loadActivities()
}

// 选中变化
const handleSelectionChange = (rows) => {
  selectedIds.value = rows.map(r => r.id)
}

// 通过活动
const handleApprove = async (row) => {
  try {
    const res = await fetch(`${API_BASE}/admin/activities/${row.id}/approve`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ reviewed_by: 'admin' })
    })
    if (res.ok) {
      ElMessage.success(`活动「${row.title}」已通过审核`)
      loadPending()
      loadActivities()
    } else {
      const errData = await res.json().catch(() => ({}))
      console.error('审批失败:', errData)
      ElMessage.error(errData.detail || '审批失败')
    }
  } catch (err) {
    console.error('审批失败:', err)
    ElMessage.error('审批失败')
  }
}

// 打开驳回对话框
const openRejectDialog = (row) => {
  rejectTarget.value = row
  rejectReason.value = ''
  rejectDialogVisible.value = true
}

// 驳回活动
const handleReject = async () => {
  if (!rejectReason.value.trim()) {
    ElMessage.warning('请输入驳回原因')
    return
  }
  try {
    const res = await fetch(`${API_BASE}/admin/activities/${rejectTarget.value.id}/reject`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ reviewed_by: 'admin', comment: rejectReason.value })
    })
    if (res.ok) {
      ElMessage.success(`活动「${rejectTarget.value.title}」已驳回`)
      rejectDialogVisible.value = false
      loadPending()
      loadActivities()
    }
  } catch (err) {
    console.error('驳回失败:', err)
    ElMessage.error('驳回失败')
  }
}

// 删除活动
const handleDelete = (row) => {
  ElMessageBox.confirm(`确定删除活动「${row.title}」吗？`, '确认删除', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async () => {
    try {
      const res = await fetch(`${API_BASE}/admin/activities/${row.id}`, { method: 'DELETE' })
      if (res.ok) {
        ElMessage.success('删除成功')
        loadActivities()
      }
    } catch (err) {
      console.error('删除失败:', err)
      ElMessage.error('删除失败')
    }
  }).catch(() => {})
}

// 批量删除
const handleBatchDelete = () => {
  ElMessageBox.confirm(`确定删除选中的 ${selectedIds.value.length} 个活动吗？`, '确认批量删除', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(async () => {
    try {
      const res = await fetch(`${API_BASE}/admin/activities/batch-delete`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(selectedIds.value)
      })
      if (res.ok) {
        selectedIds.value = []
        ElMessage.success('批量删除成功')
        loadActivities()
      }
    } catch (err) {
      console.error('批量删除失败:', err)
      ElMessage.error('批量删除失败')
    }
  }).catch(() => {})
}

// 打开新增对话框
const openAddDialog = () => {
  isEdit.value = false
  resetForm()
  dialogVisible.value = true
}

// 打开编辑对话框
const openEditDialog = (row) => {
  isEdit.value = true
  Object.assign(activityForm, {
    id: row.id,
    title: row.title,
    category: row.category,
    organizer: row.organizer,
    targetGrade: row.target_grade ? (typeof row.target_grade === 'string' ? row.target_grade.split(',').filter(Boolean) : row.target_grade) : [],
    applyStart: row.apply_start,
    applyEnd: row.apply_end,
    eventStart: row.event_start,
    eventEnd: row.event_end,
    creditHours: row.credit_hours,
    scoreValue: row.score_value,
    location: row.location,
    description: row.description,
    notes: row.notes,
    capacity: row.capacity
  })
  dialogVisible.value = true
}

// 重置表单
const resetForm = () => {
  Object.assign(activityForm, {
    id: null, title: '', category: '', organizer: '',
    targetGrade: [], applyStart: '', applyEnd: '', eventStart: '', eventEnd: '',
    creditHours: 2, scoreValue: 5, location: '', description: '', notes: '', capacity: 50
  })
  if (formRef.value) formRef.value.clearValidate()
}

// 提交表单
const handleSubmit = async () => {
  if (!formRef.value) return

  await formRef.value.validate(async (valid) => {
    if (valid) {
      submitting.value = true
      try {
        const payload = {
          title: activityForm.title,
          category: activityForm.category,
          organizer: activityForm.organizer,
          target_grade: Array.isArray(activityForm.targetGrade) ? activityForm.targetGrade.join(',') : activityForm.targetGrade,
          apply_start: activityForm.applyStart,
          apply_end: activityForm.applyEnd,
          event_start: activityForm.eventStart,
          event_end: activityForm.eventEnd,
          credit_hours: activityForm.creditHours,
          score_value: activityForm.scoreValue,
          location: activityForm.location,
          description: activityForm.description,
          notes: activityForm.notes,
          capacity: activityForm.capacity
        }

        if (isEdit.value) {
          const res = await fetch(`${API_BASE}/admin/activities/${activityForm.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
          })
          if (res.ok) ElMessage.success('修改成功')
        } else {
          const res = await fetch(`${API_BASE}/admin/activities?reviewed_by=admin`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
          })
          if (res.ok) ElMessage.success('新增成功')
        }

        submitting.value = false
        dialogVisible.value = false
        resetForm()
        loadActivities()
      } catch (err) {
        submitting.value = false
        console.error('提交失败:', err)
        ElMessage.error('提交失败')
      }
    }
  })
}

// 退出登录
const handleLogout = () => {
  localStorage.removeItem('studentInfo')
  router.push('/')
  ElMessage.info('已退出登录')
}

onMounted(() => {
  loadPending()
})
</script>

<style scoped>
.admin-container {
  min-height: 100vh;
  background: #f5f7fa;
}

/* 顶部导航 */
.admin-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 28px;
  background: #2c3e50;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.nav-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.nav-icon {
  font-size: 28px;
}

.nav-title {
  color: #ffffff;
  font-size: 20px;
  font-weight: 600;
  letter-spacing: 0.5px;
}

.nav-right {
  display: flex;
  align-items: center;
  gap: 16px;
}

.admin-tag {
  font-size: 13px;
}

.logout-btn {
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
}

.logout-btn:hover {
  color: #ffffff;
}

/* Tabs 区域 */
.tabs-section {
  margin: 20px 28px;
}

.admin-tabs {
  background: #ffffff;
  border-radius: 12px;
  padding: 20px 24px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.admin-tabs :deep(.el-tabs__header) {
  margin-bottom: 20px;
}

.admin-tabs :deep(.el-tabs__nav-wrap::after) {
  background-color: #f0ece6;
}

/* 筛选栏 */
.filter-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.search-input {
  width: 240px;
}

/* 操作按钮 */
.action-buttons {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 16px;
}

.selected-info {
  margin-left: auto;
  font-size: 13px;
  color: #8b9aab;
}

/* 表格 */
.admin-table {
  border-radius: 8px;
  overflow: hidden;
}

.admin-table :deep(.el-table__header-wrapper th) {
  background: #f8f9fa;
  font-weight: 600;
  color: #5a6d7e;
}

.score-badge {
  display: inline-block;
  padding: 2px 10px;
  background: #e8f4ee;
  color: #3d8b63;
  border-radius: 12px;
  font-weight: 600;
  font-size: 13px;
}

.pagination-wrapper {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

/* 对话框表单 */
.activity-form {
  padding: 10px 0;
}

.activity-form :deep(.el-form-item__label) {
  font-weight: 500;
}
</style>
