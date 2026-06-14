<template>
  <router-view />
</template>

<script setup>
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'PingFang SC', 'Microsoft YaHei', 'Helvetica Neue', sans-serif;
  background: #f5f7fa;
  min-height: 100vh;
  color: #2c3e50;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* 学生端概览区补充说明文案 */
.hero-panel p {
  font-size: 0 !important;
  line-height: 0 !important;
}

.hero-panel p::after {
  content: '系统会根据你的四维分数、截止时间和提醒记录生成活动建议，帮助你更快找到优先补充方向。';
  display: block;
  color: #667f99;
  font-size: 16px;
  line-height: 1.7;
  max-width: 680px;
}
</style>