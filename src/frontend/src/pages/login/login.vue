<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { loginAPI, getUserInfoAPI } from '../../apis/auth'
import { useUserStore } from '../../store/user'

const router = useRouter()
const userStore = useUserStore()

const loginForm = reactive({
  username: '',
  password: ''
})

const loading = ref(false)

const handleLogin = async () => {
  if (!loginForm.username || !loginForm.password) {
    ElMessage.warning('请输入用户名和密码')
    return
  }

  try {
    loading.value = true
    const response = await loginAPI(loginForm)
    
    // response.data结构可能是{status_code: number, data: {...}}
    const responseData = response.data
    if (responseData.status_code === 200) {
      ElMessage.success('登录成功')
      
      // 使用store管理用户状态
      const userData = responseData.data || {}
      if (userData.access_token && userData.user_id) {
        // 先保存基础用户信息
        userStore.setUserInfo(userData.access_token, {
          id: userData.user_id,
          username: loginForm.username
        })
        
        // 立即获取完整的用户信息（包括头像等）
        try {
          const userInfoResponse = await getUserInfoAPI(userData.user_id)
          const userInfoData = userInfoResponse.data
          if (userInfoData.status_code === 200) {
            const completeUserData = userInfoData.data || {}
            // 更新用户信息，包含头像
            userStore.updateUserInfo({
              avatar: completeUserData.user_avatar || completeUserData.avatar,
              description: completeUserData.user_description || completeUserData.description
            })
          }
        } catch (error) {
          console.error('获取用户详细信息失败:', error)
        }
      }
      
      // 跳转到主页
      router.push('/')
    } else {
      ElMessage.error(responseData.status_message || '登录失败')
    }
  } catch (error: any) {
    console.error('登录错误:', error)
    if (error.response?.data?.message) {
      ElMessage.error(error.response.data.status_message)
    } else if (error.response?.data?.detail) {
      ElMessage.error(error.response.data.detail)
    } else {
      ElMessage.error('登录失败，请检查网络连接')
    }
  } finally {
    loading.value = false
  }
}

const goToRegister = () => {
  router.push('/register')
}
</script>

<template>
  <div class="login-container">
    <!-- 左侧3D图形区域 -->
    <div class="left-section">
      <div class="graphic-container">
        <div class="cube-3d">
          <div class="cube-face front"></div>
          <div class="cube-face back"></div>
          <div class="cube-face right"></div>
          <div class="cube-face left"></div>
          <div class="cube-face top"></div>
          <div class="cube-face bottom"></div>
        </div>
        <div class="cylinder-3d"></div>
        <div class="sphere-3d"></div>
      </div>
    </div>

    <!-- 右侧登录表单区域 -->
    <div class="right-section">
      <div class="login-form-container">
        <!-- Logo和标题 -->
        <div class="header">
          <div class="logo">
            <span class="logo-text">AgentChat</span>
          </div>
          <p class="subtitle">智能小助手 - 更智能、更多元的大模型应用开发平台</p>
        </div>

        <!-- 登录表单 -->
        <div class="login-form">
          <div class="form-group">
            <label class="form-label">账号</label>
            <el-input
              v-model="loginForm.username"
              placeholder="请输入账号"
              size="large"
              class="login-input"
              @keyup.enter="handleLogin"
            />
          </div>

          <div class="form-group">
            <label class="form-label">密码</label>
            <el-input
              v-model="loginForm.password"
              type="password"
              placeholder="请输入密码"
              size="large"
              class="login-input"
              show-password
              @keyup.enter="handleLogin"
            />
          </div>

          <div class="form-actions">
            <div class="register-link">
              <span>没有账号？</span>
              <a href="#" @click="goToRegister">注册</a>
            </div>
          </div>

          <el-button
            type="primary"
            size="large"
            class="login-button"
            :loading="loading"
            @click="handleLogin"
          >
            登录
          </el-button>
        </div>

        <!-- 底部版本信息 -->
        <div class="footer">
          <div class="version-badge" title="AgentChat 版本">v2.5.0</div>
          <div class="footer-icons">
            <a href="https://github.com/Shy2593666979/AgentChat" target="_blank" class="icon-link" title="GitHub">
              <img src="../../assets/github.png" alt="GitHub" class="icon-img" />
            </a>
            <a href="https://uawlh9wstr9.feishu.cn/wiki/QOaLwMDtBiiduWk4YtAcavEsnne" target="_blank" class="icon-link" title="帮助文档">
              <img src="../../assets/help.png" alt="帮助文档" class="icon-img" />
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.login-container {
  display: flex;
  height: 100vh;
  background: linear-gradient(135deg, #f0e6ff 0%, #ffe4f0 40%, #e0f0ff 100%);
  position: relative;
  overflow: hidden;
}

/* 背景网格动画 */
.login-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image:
    linear-gradient(rgba(124, 58, 237, 0.04) 1px, transparent 1px),
    linear-gradient(90deg, rgba(124, 58, 237, 0.04) 1px, transparent 1px);
  background-size: 60px 60px;
  pointer-events: none;
  z-index: 0;
}

.left-section {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  z-index: 1;

  .graphic-container {
    position: relative;
    width: 400px;
    height: 400px;

    .cube-3d {
      position: absolute;
      width: 120px;
      height: 120px;
      top: 50px;
      left: 100px;
      transform-style: preserve-3d;
      animation: rotateCube 10s infinite linear;

      .cube-face {
        position: absolute;
        width: 120px;
        height: 120px;
        background: linear-gradient(135deg, rgba(124, 58, 237, 0.4), rgba(0, 102, 255, 0.2));
        border: 1px solid rgba(124, 58, 237, 0.3);
        box-shadow: 0 0 20px rgba(124, 58, 237, 0.15);
        backdrop-filter: blur(2px);

        &.front { transform: rotateY(0deg) translateZ(60px); }
        &.back { transform: rotateY(180deg) translateZ(60px); }
        &.right { transform: rotateY(90deg) translateZ(60px); }
        &.left { transform: rotateY(-90deg) translateZ(60px); }
        &.top { transform: rotateX(90deg) translateZ(60px); }
        &.bottom { transform: rotateX(-90deg) translateZ(60px); }
      }
    }

    .cylinder-3d {
      position: absolute;
      width: 80px;
      height: 160px;
      top: 200px;
      left: 50px;
      background: linear-gradient(180deg, rgba(236, 72, 153, 0.6), rgba(168, 85, 247, 0.3));
      border-radius: 40px;
      box-shadow: 0 0 40px rgba(124, 58, 237, 0.25), 0 10px 30px rgba(168, 85, 247, 0.15);
      animation: floatUp 6s ease-in-out infinite;
    }

    .sphere-3d {
      position: absolute;
      width: 60px;
      height: 60px;
      top: 120px;
      right: 80px;
      background: radial-gradient(circle at 30% 30%, rgba(236, 72, 153, 0.7), rgba(0, 102, 255, 0.3));
      border-radius: 50%;
      box-shadow: 0 0 50px rgba(124, 58, 237, 0.3), 0 8px 25px rgba(168, 85, 247, 0.2);
      animation: floatDown 8s ease-in-out infinite;
    }
  }
}

.right-section {
  width: 450px;
  background: var(--bg-secondary);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: -2px 0 24px rgba(124, 58, 237, 0.08), -1px 0 0 var(--border-light);
  z-index: 1;

  .login-form-container {
    width: 320px;
    padding: 40px 0;

    .header {
      text-align: center;
      margin-bottom: 40px;

      .logo {
        margin-bottom: 16px;

        .logo-text {
          display: inline-block;
          background: var(--gradient-primary);
          background-size: 200% 200%;
          color: white;
          padding: 12px 24px;
          border-radius: var(--radius-md);
          font-size: 20px;
          font-weight: 700;
          letter-spacing: 2px;
          font-family: 'SF Pro Display', 'Helvetica Neue', 'Arial', sans-serif;
          box-shadow: var(--shadow-colorful);
          animation: gradient-shift 4s ease infinite;
        }
      }

      .subtitle {
        color: var(--text-secondary);
        font-size: 15px;
        margin: 0;
        line-height: 1.6;
        font-weight: 400;
        font-family: 'PingFang SC', 'Helvetica Neue', 'Arial', sans-serif;
      }
    }

    .login-form {
      .form-group {
        margin-bottom: 20px;

        .form-label {
          display: block;
          font-size: 15px;
          font-weight: 600;
          color: var(--text-primary);
          margin-bottom: 10px;
          font-family: 'PingFang SC', 'Helvetica Neue', 'Arial', sans-serif;
          letter-spacing: 0.5px;
        }

        .login-input {
          :deep(.el-input__wrapper) {
            background: var(--bg-tertiary);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 12px 16px;
            box-shadow: none;
            transition: all 0.2s ease;

            &:hover {
              border-color: var(--accent);
            }

            &.is-focus {
              border-color: var(--accent);
              box-shadow: 0 0 0 3px var(--accent-glow), 0 0 12px var(--accent-glow);
            }
          }

          :deep(.el-input__inner) {
            color: var(--text-primary);
            font-size: 16px;
            font-family: 'PingFang SC', 'Helvetica Neue', 'Arial', sans-serif;
            font-weight: 400;

            &::placeholder {
              color: var(--text-muted);
              font-size: 15px;
            }
          }
        }
      }

      .form-actions {
        display: flex;
        justify-content: flex-end;
        margin-bottom: 24px;

        .register-link {
          font-size: 15px;
          color: var(--text-secondary);
          font-family: 'PingFang SC', 'Helvetica Neue', 'Arial', sans-serif;

          a {
            color: var(--accent);
            text-decoration: none;
            margin-left: 6px;
            font-weight: 500;
            transition: all 0.2s ease;

            &:hover {
              text-decoration: underline;
              color: var(--accent-light);
              text-shadow: 0 0 8px var(--accent-glow);
            }
          }
        }
      }

      .login-button {
        width: 100%;
        height: 52px;
        background: var(--gradient-primary);
        background-size: 200% 200%;
        border: none;
        border-radius: var(--radius-md);
        font-size: 18px;
        font-weight: 600;
        color: #fff;
        font-family: 'PingFang SC', 'Helvetica Neue', 'Arial', sans-serif;
        letter-spacing: 1px;
        transition: all var(--transition-base);
        box-shadow: var(--shadow-colorful);
        animation: gradient-shift 4s ease infinite;

        &:hover {
          transform: translateY(-2px);
          box-shadow: var(--shadow-lg), 0 0 40px rgba(124, 58, 237, 0.25);
        }

        &:active {
          transform: translateY(0);
        }
      }
    }

    .footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: 36px;
      color: var(--text-muted);
      font-size: 13px;
      font-family: 'PingFang SC', 'Helvetica Neue', 'Arial', sans-serif;
      font-weight: 400;
      border-top: 1px solid var(--border);
      padding-top: 16px;

      .version-badge {
        display: inline-flex;
        align-items: center;
        padding: 4px 10px;
        border-radius: 999px;
        background: var(--accent-glow);
        color: var(--accent-light);
        border: 1px solid rgba(88, 166, 255, 0.25);
        font-weight: 600;
        letter-spacing: 0.3px;
      }

      .footer-icons {
        display: flex;
        gap: 10px;

        a {
          width: 28px;
          height: 28px;
          display: flex;
          align-items: center;
          justify-content: center;
          background: var(--bg-tertiary);
          border: 1px solid var(--border);
          border-radius: 8px;
          transition: all 0.2s ease;
          overflow: hidden;

          &:hover {
            transform: translateY(-1px);
            box-shadow: 0 0 16px rgba(88, 166, 255, 0.2);
            border-color: rgba(88, 166, 255, 0.4);
          }

          .icon-img {
            width: 18px;
            height: 18px;
            object-fit: contain;
            filter: brightness(0.9);
          }
        }
      }
    }
  }
}

@keyframes rotateCube {
  0% { transform: rotateX(0deg) rotateY(0deg); }
  100% { transform: rotateX(360deg) rotateY(360deg); }
}

@keyframes floatUp {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-15px); }
}

@keyframes floatDown {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(10px); }
}
</style>