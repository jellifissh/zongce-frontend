<template>
  <div class="login-page">
    <div class="orb orb-one"></div>
    <div class="orb orb-two"></div>

    <div class="login-shell">
      <section class="brand-panel">
        <div class="brand-badge">AI · 综测助手</div>
        <div class="brand-main">
          <div class="brand-icon">🎓</div>
          <h1>综测服务智能体</h1>
          <p>查活动、算综测、做规划、设提醒，一个入口搞定，终于不用在表格和通知群里考古了。</p>
        </div>

        <div class="feature-grid">
          <div class="feature-card">
            <strong>4 类分数</strong>
            <span>A/B/C/D 权重实时计算</span>
          </div>
          <div class="feature-card">
            <strong>智能推荐</strong>
            <span>按缺口匹配活动</span>
          </div>
          <div class="feature-card">
            <strong>活动提醒</strong>
            <span>截止前自动提醒</span>
          </div>
        </div>
      </section>

      <section class="form-panel">
        <div class="form-title-row">
          <div>
            <p class="eyebrow">登录入口</p>
            <h2>{{ isAdmin ? '进入管理端' : isStudentAdmin ? '学生管理员入口' : '学生信息确认' }}</h2>
          </div>
          <span class="mode-pill">{{ isAdmin ? '管理员' : isStudentAdmin ? '学生管理员' : '学生' }}</span>
        </div>

        <el-form
          :model="form"
          :rules="dynamicRules"
          ref="loginFormRef"
          class="login-form"
          label-position="top"
        >
          <div class="top-fields">
            <el-form-item prop="studentId" class="clean-form-item">
              <template #label>学号</template>
              <el-input
                v-model="form.studentId"
                placeholder="学生输入学号，管理员输入 admin"
                size="large"
                class="soft-input"
              />
            </el-form-item>

            <el-form-item prop="qq" class="clean-form-item">
              <template #label>QQ号</template>
              <el-input
                v-model="form.qq"
                placeholder="请输入QQ号"
                size="large"
                class="soft-input"
              />
            </el-form-item>
          </div>

          <el-alert
            v-if="isAdmin"
            title="已识别为管理员账号"
            description="点击进入后跳转至活动管理端，可维护活动信息。"
            type="info"
            :closable="false"
            show-icon
            class="mode-hint"
          />

          <el-alert
            v-if="isStudentAdmin"
            title="已识别为学生管理员账号"
            description="具有学生身份和管理权限，可用于演示双端能力。"
            type="warning"
            :closable="false"
            show-icon
            class="mode-hint"
          />

          <template v-if="!isAdmin">
            <div class="section-head">
              <span>首次登录信息</span>
              <small>全部内容压缩在一屏里，终于不用滚动考验鼠标寿命。</small>
            </div>

            <div class="score-block">
              <div class="score-grid">
                <div class="score-card score-a">
                  <span>A类·思想品德</span>
                  <el-input-number
                    v-model="form.scoreA"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
                <div class="score-card score-b">
                  <span>B类·科学文化</span>
                  <el-input-number
                    v-model="form.scoreB"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
                <div class="score-card score-c">
                  <span>C类·身体心理</span>
                  <el-input-number
                    v-model="form.scoreC"
                    :min="0"
                    :max="100"
                    controls-position="right"
                    class="score-number"
                  />
                </div>
                <div class="score-card score-d">
                  <span>D类·劳动实践</span>
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
                <span>当前总分</span>
                <strong>{{ totalScore }}</strong>
                <small>基础分：A/B 默认 50，C/D 默认 0</small>
              </div>
            </div>

            <div class="preference-grid">
              <el-form-item label="年级" prop="grade" class="clean-form-item">
                <el-radio-group v-model="form.grade" class="grade-group">
                  <el-radio-button value="大一">大一</el-radio-button>
                  <el-radio-button value="大二">大二</el-radio-button>
                  <el-radio-button value="大三">大三</el-radio-button>
                  <el-radio-button value="大四">大四</el-radio-button>
                </el-radio-group>
              </el-form-item>

              <el-form-item label="感兴趣的活动类型" prop="interests" class="clean-form-item">
                <el-checkbox-group v-model="form.interests" class="interest-grid">
                  <el-checkbox
                    v-for="item in interestOptions"
                    :key="item.value"
                    :label="item.value"
                    class="interest-card"
                  >
                    <span class="interest-name">{{ item.label }}</span>
                    <span class="interest-desc">{{ item.desc }}</span>
                  </el-checkbox>
                </el-checkbox-group>
              </el-form-item>
            </div>
          </template>

          <div class="submit-row">
            <el-button
              type="primary"
              size="large"
              class="submit-btn"
              :loading="loading"
              @click="handleLogin"
            >
              {{ isAdmin ? '进入管理端' : '进入学生端' }}
            </el-button>
            <p class="footer-hint">
              管理员：<code>admin</code>　学生管理员：<code>stuadmin1</code>
            </p>
          </div>
        </el-form>
      </section>
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

  if (!isAdmin.value) {
    base.grade = [
      { required: true, message: '请选择年级', trigger: 'change' }
    ]
    base.interests = [
      { type: 'array', required: true, min: 1, message: '请至少选择一个感兴趣的活动类型', trigger: 'change' }
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
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background:
    radial-gradient(circle at 16% 12%, rgba(109, 158, 235, 0.28), transparent 28%),
    radial-gradient(circle at 88% 18%, rgba(90, 187, 150, 0.22), transparent 24%),
    linear-gradient(135deg, #0f2744 0%, #1e4975 52%, #102844 100%);
  padding: 24px;
  position: relative;
  overflow: hidden;
}

.orb {
  position: absolute;
  border-radius: 999px;
  filter: blur(2px);
  opacity: 0.45;
  pointer-events: none;
}

.orb-one {
  width: 220px;
  height: 220px;
  left: -60px;
  bottom: 8%;
  background: rgba(255, 255, 255, 0.15);
}

.orb-two {
  width: 320px;
  height: 320px;
  right: -120px;
  top: -80px;
  background: rgba(255, 255, 255, 0.12);
}

.login-shell {
  width: min(1060px, 100%);
  max-height: calc(100vh - 48px);
  display: grid;
  grid-template-columns: 0.9fr 1.1fr;
  border-radius: 30px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.92);
  border: 1px solid rgba(255, 255, 255, 0.35);
  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.28);
  backdrop-filter: blur(18px);
  position: relative;
  z-index: 1;
}

.brand-panel {
  padding: 38px;
  color: #ffffff;
  background:
    linear-gradient(145deg, rgba(18, 50, 84, 0.96), rgba(32, 85, 134, 0.92)),
    radial-gradient(circle at top right, rgba(255, 255, 255, 0.24), transparent 38%);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 28px;
}

.brand-badge {
  width: fit-content;
  padding: 8px 14px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.14);
  border: 1px solid rgba(255, 255, 255, 0.18);
  font-size: 13px;
  letter-spacing: 0.5px;
}

.brand-icon {
  width: 64px;
  height: 64px;
  display: grid;
  place-items: center;
  border-radius: 22px;
  background: rgba(255, 255, 255, 0.14);
  font-size: 34px;
  margin-bottom: 18px;
}

.brand-main h1 {
  font-size: 32px;
  line-height: 1.2;
  letter-spacing: 1px;
  margin-bottom: 14px;
}

.brand-main p {
  max-width: 360px;
  color: rgba(255, 255, 255, 0.78);
  line-height: 1.8;
  font-size: 15px;
}

.feature-grid {
  display: grid;
  gap: 12px;
}

.feature-card {
  padding: 14px 16px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.14);
}

.feature-card strong {
  display: block;
  font-size: 15px;
  margin-bottom: 4px;
}

.feature-card span {
  color: rgba(255, 255, 255, 0.72);
  font-size: 13px;
}

.form-panel {
  padding: 30px 34px;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.96), rgba(248, 251, 255, 0.98));
  overflow: auto;
}

.form-title-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 18px;
}

.eyebrow {
  color: #6d86a5;
  font-size: 13px;
  margin-bottom: 4px;
}

.form-title-row h2 {
  color: #17375f;
  font-size: 25px;
  line-height: 1.25;
}

.mode-pill {
  padding: 8px 14px;
  border-radius: 999px;
  background: #eaf3ff;
  color: #1e5a92;
  font-weight: 700;
  font-size: 13px;
  white-space: nowrap;
}

.login-form :deep(.el-form-item) {
  margin-bottom: 14px;
}

.clean-form-item :deep(.el-form-item__label) {
  color: #526b86;
  font-weight: 700;
  font-size: 13px;
  line-height: 1;
  margin-bottom: 8px;
}

.top-fields {
  display: grid;
  grid-template-columns: 1.1fr 0.9fr;
  gap: 14px;
}

.soft-input :deep(.el-input__wrapper) {
  min-height: 44px;
  border-radius: 14px;
  box-shadow: none;
  border: 1px solid #dce6f2;
  background: #ffffff;
  transition: all 0.2s ease;
}

.soft-input :deep(.el-input__wrapper:hover),
.soft-input :deep(.el-input__wrapper.is-focus) {
  border-color: #2f6fa5;
  box-shadow: 0 0 0 4px rgba(47, 111, 165, 0.1);
}

.mode-hint {
  border-radius: 14px;
  margin: 6px 0 14px;
}

.section-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin: 4px 0 12px;
  padding-top: 12px;
  border-top: 1px solid #edf2f7;
}

.section-head span {
  font-weight: 800;
  color: #17375f;
  font-size: 15px;
}

.section-head small {
  color: #8aa0b6;
  font-size: 12px;
}

.score-block {
  display: grid;
  grid-template-columns: 1fr 170px;
  gap: 12px;
  align-items: stretch;
  margin-bottom: 12px;
}

.score-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 10px;
}

.score-card {
  padding: 12px;
  border-radius: 16px;
  background: #ffffff;
  border: 1px solid #e5edf6;
  box-shadow: 0 8px 22px rgba(20, 61, 103, 0.05);
}

.score-card span {
  display: block;
  margin-bottom: 8px;
  font-size: 13px;
  font-weight: 800;
}

.score-a span { color: #d98910; }
.score-b span { color: #24965b; }
.score-c span { color: #2b82c9; }
.score-d span { color: #df7723; }

.score-number {
  width: 100%;
}

.score-number :deep(.el-input__wrapper) {
  border-radius: 12px;
  box-shadow: none;
  border: 1px solid #e2eaf4;
}

.score-summary {
  border-radius: 18px;
  background: linear-gradient(145deg, #eaf4ff, #f5fbff);
  border: 1px solid #d7e8fa;
  padding: 14px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 4px;
}

.score-summary span {
  color: #55708f;
  font-size: 13px;
}

.score-summary strong {
  color: #17375f;
  font-size: 28px;
  line-height: 1.1;
}

.score-summary small {
  color: #7f94aa;
  line-height: 1.5;
}

.preference-grid {
  display: grid;
  grid-template-columns: 0.8fr 1.2fr;
  gap: 16px;
  align-items: start;
}

.grade-group {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px;
}

.grade-group :deep(.el-radio-button__inner) {
  width: 100%;
  border: 1px solid #dce6f2;
  border-radius: 12px !important;
  background: #ffffff;
  color: #526b86;
  box-shadow: none;
}

.grade-group :deep(.el-radio-button:first-child .el-radio-button__inner) {
  border-left: 1px solid #dce6f2;
}

.grade-group :deep(.el-radio-button.is-checked .el-radio-button__inner) {
  background: #17375f;
  border-color: #17375f;
  color: #ffffff;
  box-shadow: 0 8px 20px rgba(23, 55, 95, 0.22);
}

.interest-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px;
  width: 100%;
}

.interest-card {
  margin: 0 !important;
  height: auto;
}

.interest-card :deep(.el-checkbox__input) {
  align-self: flex-start;
  margin-top: 3px;
}

.interest-card :deep(.el-checkbox__label) {
  width: 100%;
  padding: 10px 12px;
  border-radius: 14px;
  background: #ffffff;
  border: 1px solid #dce6f2;
  display: flex;
  flex-direction: column;
  gap: 2px;
  transition: all 0.2s ease;
}

.interest-card :deep(.el-checkbox__label:hover) {
  border-color: #2f6fa5;
  background: #f7fbff;
}

.interest-card :deep(.el-checkbox.is-checked .el-checkbox__label) {
  background: #eaf4ff;
  border-color: #2f6fa5;
}

.interest-name {
  color: #243a52;
  font-weight: 800;
  font-size: 13px;
}

.interest-desc {
  color: #7e91a5;
  font-size: 12px;
}

.submit-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin-top: 8px;
  padding-top: 14px;
  border-top: 1px solid #edf2f7;
}

.submit-btn {
  min-width: 170px;
  height: 46px;
  border: none;
  border-radius: 14px;
  background: linear-gradient(135deg, #17375f, #2d6c9f);
  box-shadow: 0 12px 26px rgba(23, 55, 95, 0.24);
  font-size: 15px;
  font-weight: 800;
  letter-spacing: 1px;
}

.submit-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 16px 30px rgba(23, 55, 95, 0.3);
}

.footer-hint {
  color: #7f94aa;
  font-size: 12px;
  margin: 0;
  text-align: right;
}

.footer-hint code {
  padding: 2px 7px;
  border-radius: 8px;
  background: #eef4fb;
  color: #17375f;
  font-weight: 700;
}

.form-panel::-webkit-scrollbar {
  width: 6px;
}

.form-panel::-webkit-scrollbar-thumb {
  background: rgba(23, 55, 95, 0.18);
  border-radius: 999px;
}

@media (max-width: 900px) {
  .login-page {
    height: auto;
    min-height: 100vh;
    overflow: auto;
  }

  .login-shell {
    grid-template-columns: 1fr;
    max-height: none;
  }

  .brand-panel {
    padding: 28px;
  }

  .feature-grid {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}

@media (max-width: 680px) {
  .login-page {
    padding: 14px;
  }

  .form-panel {
    padding: 24px 18px;
  }

  .top-fields,
  .score-block,
  .preference-grid,
  .interest-grid {
    grid-template-columns: 1fr;
  }

  .feature-grid {
    grid-template-columns: 1fr;
  }

  .submit-row {
    flex-direction: column;
    align-items: stretch;
  }

  .footer-hint {
    text-align: center;
  }
}
</style>
