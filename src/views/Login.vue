<template>
  <div class="login-page">
    <div class="login-card">
      <div class="login-header">
        <div class="header-icon">🎓</div>
        <h1 class="header-title">综测服务智能体</h1>
      </div>

      <el-form
        :model="form"
        :rules="dynamicRules"
        ref="loginFormRef"
        class="login-form"
      >
        <!-- 学号 -->
        <el-form-item prop="studentId">
          <div class="form-row">
            <span class="form-label">学号</span>
            <el-input
              v-model="form.studentId"
              placeholder="学生请输入学号，管理员输入 admin"
              size="large"
              class="form-input"
            />
          </div>
        </el-form-item>

        <!-- QQ号 -->
        <el-form-item prop="qq">
          <div class="form-row">
            <span class="form-label">QQ号</span>
            <el-input
              v-model="form.qq"
              placeholder="请输入QQ号"
              size="large"
              class="form-input"
            />
          </div>
        </el-form-item>

        <!-- 管理员提示 -->
        <el-alert
          v-if="isAdmin"
          title="管理员模式"
          description="已识别为管理员账号，点击进入将跳转至管理端"
          type="info"
          :closable="false"
          show-icon
          class="admin-hint"
        />

        <!-- 学生管理员提示 -->
        <el-alert
          v-if="isStudentAdmin"
          title="学生管理员模式"
          description="已识别为学生管理员账号，具有学生身份和管理权限"
          type="warning"
          :closable="false"
          show-icon
          class="admin-hint"
        />

        <!-- 分割线 -->
        <div v-if="!isAdmin" class="divider">
          <span class="divider-text">以下首次登录填写</span>
        </div>

        <!-- ABCD四类素质分 -->
        <template v-if="!isAdmin">
          <el-form-item label="素质分" prop="scoreA">
            <div class="score-inputs">
              <div class="score-row">
                <div class="score-item">
                  <span class="score-label-a">A类·思想品德</span>
                  <el-input-number
                    v-model="form.scoreA"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
                <div class="score-item">
                  <span class="score-label-b">B类·科学文化</span>
                  <el-input-number
                    v-model="form.scoreB"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
              </div>
              <div class="score-row">
                <div class="score-item">
                  <span class="score-label-c">C类·身体心理</span>
                  <el-input-number
                    v-model="form.scoreC"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
                <div class="score-item">
                  <span class="score-label-d">D类·劳动实践</span>
                  <el-input-number
                    v-model="form.scoreD"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
              </div>
              <div class="score-summary">
                <span class="summary-text">
                  总分：<strong>{{ totalScore }}</strong> 分
                  （基础分：A类默认50，B类默认50，C/D类默认0）
                </span>
              </div>
            </div>
          </el-form-item>
        </template>

        <!-- 年级选择 -->
        <el-form-item v-if="!isAdmin" label="年级" prop="grade">
          <el-radio-group v-model="form.grade" class="grade-group">
            <el-radio-button value="大一">大一</el-radio-button>
            <el-radio-button value="大二">大二</el-radio-button>
            <el-radio-button value="大三">大三</el-radio-button>
            <el-radio-button value="大四">大四</el-radio-button>
          </el-radio-group>
        </el-form-item>

        <!-- 感兴趣的活动类型 -->
        <el-form-item v-if="!isAdmin" label="感兴趣的活动类型" prop="interests" class="interest-item">
          <div class="interest-label-sub">可多选</div>
          <div class="interest-list">
            <el-checkbox
              v-for="item in interestOptions"
              :key="item.value"
              v-model="form.interests"
              :value="item.value"
              class="interest-checkbox"
            >
              <div class="checkbox-content">
                <span class="checkbox-name">{{ item.label }}</span>
                <span class="checkbox-desc">{{ item.desc }}</span>
              </div>
            </el-checkbox>
          </div>
        </el-form-item>

        <!-- 进入按钮 -->
        <el-form-item class="submit-item">
          <el-button
            type="primary"
            size="large"
            class="submit-btn"
            :loading="loading"
            @click="handleLogin"
          >
            {{ isAdmin ? '进入管理端' : '进入' }}
          </el-button>
        </el-form-item>

        <!-- 底部提示 -->
        <div class="footer-hint">
          <p>首次登录后，ABCD分数可在学生端"我的分数"中随时调整</p>
          <p>管理员账号输入学号：<code>admin</code> | 学生管理员输入学号：<code>stuadmin1</code></p>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'

const router = useRouter()
const loginFormRef = ref(null)
const loading = ref(false)

const form = reactive({
  studentId: '',
  qq: '',
  grade: '',
  scoreA: 50,
  scoreB: 50,
  scoreC: 0,
  scoreD: 0,
  interests: []
})

const isAdmin = computed(() => {
  return form.studentId.toLowerCase() === 'admin'
})

const isStudentAdmin = computed(() => {
  return form.studentId.toLowerCase().includes('stuadmin')
})

const isNormalStudent = computed(() => {
  return !isAdmin.value && !isStudentAdmin.value && form.studentId.length > 0
})

const totalScore = computed(() => {
  return (form.scoreA * 0.35 + form.scoreB * 0.30 + form.scoreC * 0.20 + form.scoreD * 0.15).toFixed(2)
})

const dynamicRules = computed(() => {
  const base = {
    studentId: [
      { required: true, message: '请输入学号', trigger: 'blur' },
      { min: 1, max: 20, message: '学号长度 1-20 位', trigger: 'blur' }
    ],
    qq: [
      { required: true, message: '请输入QQ号', trigger: 'blur' }
    ]
  }
  // 管理员不需要填写年级和兴趣，学生和学生管理员需要
  if (!isAdmin.value) {
    base.grade = [
      { required: true, message: '请选择年级', trigger: 'change' }
    ]
    base.interests = [
      { required: true, message: '请至少选择一个感兴趣的活动类型', trigger: 'change' }
    ]
  }
  return base
})

const interestOptions = [
  { value: 'D类实践', label: 'D类·劳动实践', desc: '志愿服务、社会实践' },
  { value: 'B类竞赛', label: 'B类·科学文化', desc: '学科竞赛、科研项目' },
  { value: 'A类活动', label: 'A类·思想品德', desc: '思政讲座、评优评先' },
  { value: 'C类运动', label: 'C类·身体心理', desc: '体育运动、心理健康' }
]

const handleLogin = async () => {
  if (!loginFormRef.value) return

  await loginFormRef.value.validate(async (valid) => {
    if (valid) {
      loading.value = true
      try {
        const payload = {
          student_id: form.studentId,
          qq: form.qq,
          grade: form.grade || null,
          score_a: form.scoreA,
          score_b: form.scoreB,
          score_c: form.scoreC,
          score_d: form.scoreD,
          interests: form.interests || []
        }

        const res = await fetch('http://localhost:8000/api/entry', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        })

        if (!res.ok) {
          const err = await res.json()
          ElMessage.error(err.detail || '登录失败')
          loading.value = false
          return
        }

        const data = await res.json()
        // 保存用户信息到 localStorage
        localStorage.setItem('studentInfo', JSON.stringify({
          studentId: data.student_id,
          name: data.name,
          qq: form.qq,
          grade: form.grade || '',
          interests: form.interests || [],
          scoreA: form.scoreA,
          scoreB: form.scoreB,
          scoreC: form.scoreC,
          scoreD: form.scoreD,
          totalScore: totalScore.value,
          role: data.role,
          isNewUser: data.is_new_user
        }))

        ElMessage.success('登录成功！')
        // 根据后端返回的 redirect 跳转
        router.push(data.redirect)
        loading.value = false
      } catch (err) {
        console.error('登录请求失败:', err)
        ElMessage.error('后端服务未启动，请检查连接')
        loading.value = false
      }
    }
  })
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #1e3a5f 0%, #2d5a8e 50%, #1e3a5f 100%);
  padding: 20px;
  position: relative;
  overflow: hidden;
}

.login-page::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.03) 0%, transparent 70%);
  animation: rotate 30s linear infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.login-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 48px 44px;
  width: 100%;
  max-width: 520px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  position: relative;
  z-index: 1;
}

.login-header {
  text-align: center;
  margin-bottom: 40px;
}

.header-icon {
  font-size: 56px;
  margin-bottom: 12px;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

.header-title {
  font-size: 26px;
  font-weight: 600;
  color: #1e3a5f;
  letter-spacing: 2px;
}

.login-form {
  width: 100%;
}

.login-form :deep(.el-form-item) {
  margin-bottom: 24px;
}

.login-form :deep(.el-form-item__label) {
  font-size: 15px;
  font-weight: 500;
  color: #2c3e50;
  padding-bottom: 8px;
}

.admin-hint {
  border-radius: 10px;
  margin-bottom: 24px;
}

.form-row {
  display: flex;
  align-items: center;
  gap: 16px;
  width: 100%;
}

.form-label {
  width: 60px;
  text-align: right;
  font-size: 15px;
  font-weight: 500;
  color: #5a6d7e;
  flex-shrink: 0;
}

.form-input {
  flex: 1;
}

.form-input :deep(.el-input__wrapper) {
  border-radius: 10px;
  padding: 6px 14px;
  box-shadow: none;
  border: 1px solid #e8e4db;
  transition: all 0.3s;
}

.form-input :deep(.el-input__wrapper:hover) {
  border-color: #1e3a5f;
}

.form-input :deep(.el-input__wrapper.is-focus) {
  border-color: #1e3a5f;
  box-shadow: 0 0 0 3px rgba(30, 58, 95, 0.1);
}

/* 分割线 */
.divider {
  display: flex;
  align-items: center;
  margin: 32px 0;
}

.divider::before,
.divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: linear-gradient(to right, transparent, #e8e4db, transparent);
}

.divider-text {
  padding: 0 16px;
  font-size: 13px;
  color: #8b9aab;
  white-space: nowrap;
}

/* ABCD素质分 */
.score-inputs {
  width: 100%;
}

.score-row {
  display: flex;
  gap: 16px;
  margin-bottom: 12px;
}

.score-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 12px;
  background: #faf9f7;
  border-radius: 10px;
  border: 1px solid #f0ece6;
  transition: all 0.2s;
}

.score-item:hover {
  border-color: #e2dcd2;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.score-label-a,
.score-label-b,
.score-label-c,
.score-label-d {
  font-size: 13px;
  font-weight: 600;
  white-space: nowrap;
}

.score-label-a { color: #F39C12; }
.score-label-b { color: #27AE60; }
.score-label-c { color: #3498DB; }
.score-label-d { color: #E67E22; }

.score-number {
  width: 100%;
}

.score-number :deep(.el-input__wrapper) {
  border-radius: 8px;
  box-shadow: none;
  border: 1px solid #e8e4db;
}

.score-number :deep(.el-input__wrapper.is-focus) {
  border-color: #1e3a5f;
  box-shadow: 0 0 0 2px rgba(30, 58, 95, 0.08);
}

.score-summary {
  text-align: center;
  padding: 12px;
  background: #e8f0f8;
  border-radius: 8px;
  margin-top: 8px;
}

.summary-text {
  font-size: 14px;
  color: #2c3e50;
}

.summary-text strong {
  font-size: 18px;
  color: #1e3a5f;
}

/* 年级选择 */
.grade-group {
  width: 100%;
  display: flex;
  gap: 0;
}

.grade-group :deep(.el-radio-button__inner) {
  border-radius: 0;
  padding: 12px 28px;
  font-size: 14px;
  border-color: #e8e4db;
  background: #faf9f7;
  color: #5a6d7e;
  transition: all 0.3s;
}

.grade-group :deep(.el-radio-button:first-child .el-radio-button__inner) {
  border-radius: 10px 0 0 10px;
  border-left: 1px solid #e8e4db;
}

.grade-group :deep(.el-radio-button:last-child .el-radio-button__inner) {
  border-radius: 0 10px 10px 0;
}

.grade-group :deep(.el-radio-button.is-checked .el-radio-button__inner) {
  background: #1e3a5f;
  border-color: #1e3a5f;
  color: #ffffff;
  box-shadow: none;
}

/* 感兴趣的活动类型 */
.interest-item {
  margin-bottom: 32px;
}

.interest-item :deep(.el-form-item__content) {
  flex-direction: column;
  align-items: flex-start;
}

.interest-label-sub {
  font-size: 13px;
  color: #8b9aab;
  margin-bottom: 12px;
}

.interest-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  width: 100%;
}

.interest-checkbox {
  width: 100%;
  margin: 0 !important;
}

.interest-checkbox :deep(.el-checkbox__label) {
  width: 100%;
  padding: 10px 14px;
  border-radius: 8px;
  transition: all 0.2s;
}

.interest-checkbox :deep(.el-checkbox__label:hover) {
  background: #faf9f7;
}

.interest-checkbox :deep(.el-checkbox.is-checked .el-checkbox__label) {
  background: #e8f0f8;
}

.checkbox-content {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.checkbox-name {
  font-size: 14px;
  font-weight: 500;
  color: #2c3e50;
}

.checkbox-desc {
  font-size: 12px;
  color: #8b9aab;
}

/* 提交按钮 */
.submit-item {
  margin-top: 36px;
  margin-bottom: 0;
}

.submit-item :deep(.el-form-item__content) {
  justify-content: center;
}

.submit-btn {
  width: 200px;
  height: 48px;
  font-size: 17px;
  font-weight: 600;
  letter-spacing: 8px;
  border-radius: 12px;
  background: #1e3a5f;
  border: none;
  transition: all 0.3s;
}

.submit-btn:hover {
  background: #2a4f7a;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(30, 58, 95, 0.35);
}

.submit-btn:active {
  transform: translateY(0);
}

/* 底部提示 */
.footer-hint {
  text-align: center;
  margin-top: 24px;
  padding-top: 16px;
  border-top: 1px solid #f0ece6;
}

.footer-hint p {
  font-size: 12px;
  color: #8b9aab;
  margin-bottom: 4px;
}

.footer-hint code {
  background: #f0ece6;
  padding: 2px 8px;
  border-radius: 4px;
  color: #1e3a5f;
  font-size: 12px;
}

/* 响应式 */
@media (max-width: 560px) {
  .login-card {
    padding: 36px 24px;
  }

  .form-row {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .form-label {
    width: auto;
    text-align: left;
  }

  .score-row {
    flex-direction: column;
    gap: 10px;
  }

  .grade-group :deep(.el-radio-button__inner) {
    padding: 10px 18px;
    font-size: 13px;
  }

  .submit-btn {
    width: 100%;
    letter-spacing: 4px;
  }
}
</style>
