<script setup lang="ts">
import { onMounted, ref, watch, computed } from "vue"
import { useRouter } from "vue-router"
import { useRoute } from "vue-router"
import { ElMessage, ElMessageBox } from 'element-plus'
import workspaceIcon from '../assets/workspace.svg'
import applicationCenterIcon from '../assets/application-center.svg'
import dialogIcon from '../assets/dialog.svg'
import robotIcon from '../assets/robot.svg'
import pluginIcon from '../assets/plugin.svg'
import knowledgeIcon from '../assets/knowledge.svg'
import modelIcon from '../assets/model.svg'
import mcpIcon from '../assets/mcp.svg'
import skillIcon from '../assets/skill.svg'
import { User, SwitchButton, Setting } from '@element-plus/icons-vue'
import { useAgentCardStore } from "../store/agent_card"
import { useUserStore } from "../store/user"
import { getAgentsAPI } from "../apis/agent"
import { logoutAPI, getUserInfoAPI } from "../apis/auth"
import { Agent } from "../type"

const agentCardStore = useAgentCardStore()
const userStore = useUserStore()
const route = useRoute()
const router = useRouter()
const itemName = ref("智能小助手")
const showAppCenterMenu = ref(false)
let appCenterHoverTimer: any = null

const openAppCenterMenu = () => {
  if (appCenterHoverTimer) clearTimeout(appCenterHoverTimer)
  showAppCenterMenu.value = true
}

const closeAppCenterMenu = () => {
  if (appCenterHoverTimer) clearTimeout(appCenterHoverTimer)
  appCenterHoverTimer = setTimeout(() => {
    showAppCenterMenu.value = false
  }, 120)
}

const goWorkspaceTop = () => {
  router.push('/workspace')
}

const appCenterColumns = ref([
  [
    { label: '会话', icon: dialogIcon, route: '/conversation' },
    { label: '工作台', icon: workspaceIcon, route: '/workspace' }
  ],
  [
    { label: '智能体', icon: robotIcon, route: '/agent' },
    { label: '工具', icon: pluginIcon, route: '/tool' }
  ],
  [
    { label: '知识库', icon: knowledgeIcon, route: '/knowledge' },
    { label: '模型', icon: modelIcon, route: '/model' }
  ],
  [
    { label: 'MCP', icon: mcpIcon, route: '/mcp-server' },
    { label: 'Skill', icon: skillIcon, route: '/agent-skill' }
  ]
])
const current = ref(route.meta.current)
const cardList = ref<Agent[]>([])

// 顶栏按钮激活态
const isWorkspaceActive = computed(() => route.path.startsWith('/workspace'))
const isAppCenterActive = computed(() => route.path.startsWith('/homepage'))

// 初始化用户状态
onMounted(async () => {
  userStore.initUserState()
  
  // 如果已登录但没有头像，则尝试获取用户信息
  if (userStore.isLoggedIn && userStore.userInfo && !userStore.userInfo.avatar) {
    try {
      const response = await getUserInfoAPI(userStore.userInfo.id)
      if (response.data.status_code === 200 && response.data.data) {
        const userData = response.data.data
        userStore.updateUserInfo({
          avatar: userData.user_avatar || userData.avatar || '/user.svg',
          description: userData.user_description || userData.description
        })
      }
    } catch (error) {
      console.error('初始化时获取用户信息失败:', error)
    }
  }
  
  updateList()
})

const godefault = () => {
  agentCardStore.clear()
  router.push("/")
}
  
const updateList = async () => {
  try {
    const response = await getAgentsAPI()
    cardList.value = response.data.data
  } catch (error) {
    console.error('获取智能体列表失败:', error)
  }
}

const goCurrent = (item: string) => {
  const routes: Record<string, string> = {
    "homepage": "/homepage",
    "conversation": "/conversation",
    "agent": "/agent",
    "mcp-server": "/mcp-server",
    "mcp-chat": "/mcp-server/chat",
    "knowledge": "/knowledge",
    "tool": "/tool",
    "agent-skill": "/agent-skill",
    "model": "/model",
    "workspace": "/workspace",
    "dashboard": "/dashboard"
  }
  
  router.push(routes[item] || "/")
}

// 处理菜单选择事件
const handleMenuSelect = (index: string) => {
  goCurrent(index)
}

// 用户下拉菜单命令处理
const handleUserCommand = async (command: string) => {
  switch (command) {
    case 'profile':
      router.push('/profile')
      break
    case 'settings':
      router.push('/configuration')
      break
    case 'logout':
      await handleLogout()
      break
  }
}

// 退出登录
const handleLogout = async () => {
  try {
    await logoutAPI()
  } catch (error) {
    console.error('调用登出接口失败:', error)
  }
  userStore.logout()
  ElMessage.success('已退出登录')
  router.push('/login')
}

// 头像加载错误处理
const handleAvatarError = (event: Event) => {
  const target = event.target as HTMLImageElement
  if (target) {
    target.src = '/user.svg'
  }
}

watch(
  route,
  (val) => {
    current.value = route.meta.current
  },
  {
    immediate: true
  }
)
</script>

<template>
  <div class="ai-body">
    <div class="ai-nav">
      <div class="left">
        <div class="item-img" @click="godefault">
          <img :src="robotIcon" alt="Logo" class="logo" />
        </div>
        <div class="nav-links">
          <img src="../assets/agentchat.svg" alt="智能小助手" class="brand-logo-img" />
        </div>
      </div>
      <div class="right">
        <!-- 用户信息区域 -->
        <div class="user-info">
          <el-dropdown @command="handleUserCommand" trigger="click">
            <div class="user-avatar-wrapper">
              <div class="user-avatar">
                <img
                  :src="userStore.userInfo?.avatar || '/user.svg'"
                  alt="用户头像"
                  style="width: 40px; height: 40px; border-radius: 50%"
                  @error="handleAvatarError"
                  referrerpolicy="no-referrer"
                />
              </div>
            </div>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item command="profile" :icon="User">
                  个人资料
                </el-dropdown-item>
<!--                <el-dropdown-item command="settings" :icon="Setting">-->
<!--                  系统设置-->
<!--                </el-dropdown-item>-->
                <el-dropdown-item divided command="logout" :icon="SwitchButton">
                  退出登录
                </el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
      </div>
    </div>
    <div class="ai-main">
      <el-col :span="2">
        <div class="sidebar-content">
          <el-menu
            active-text-color="#6b9eff"
            background-color="#f4f5f8"
            class="el-menu-vertical-demo"
            :default-active="current"
            text-color="#909399"
            @select="handleMenuSelect"
          >
            <el-menu-item index="workspace" @click="goCurrent('workspace')">
              <template #title>
                <el-icon>
                  <img src="../assets/workspace.svg" width="22px" height="22px" />
                </el-icon>
                <span>工作台</span>
              </template>
            </el-menu-item>
            <el-menu-item index="homepage" @click="goCurrent('homepage')">
              <template #title>
                <el-icon>
                  <img src="../assets/explore.svg" width="22px" height="22px" />
                </el-icon>
                <span>探索</span>
              </template>
            </el-menu-item>
            <el-menu-item index="conversation" @click="goCurrent('conversation')">
              <template #title>
                <el-icon>
                  <img src="../assets/dialog.svg" width="22px" height="22px" />
                </el-icon>
                <span>会话</span>
              </template>
            </el-menu-item>
            <el-menu-item index="agent" @click="goCurrent('agent')">
              <template #title>
                <el-icon>
                  <img src="../assets/robot.svg" width="22px" height="22px" />
                </el-icon>
                <span>智能体</span>
              </template>
            </el-menu-item>
            <el-sub-menu index="mcp">
              <template #title>
                <el-icon>
                  <img src="../assets/mcp.svg" width="22px" height="22px" />
                </el-icon>
                <span>MCP</span>
              </template>
              <el-menu-item index="mcp-server">
                <span>管理 MCP</span>
              </el-menu-item>
              <el-menu-item index="mcp-chat">
                <span>生成 MCP</span>
              </el-menu-item>
            </el-sub-menu>
            <el-menu-item index="knowledge" @click="goCurrent('knowledge')">
              <template #title>
                <el-icon>
                  <img src="../assets/knowledge.svg" width="22px" height="22px" />
                </el-icon>
                <span>知识库</span>
              </template>
            </el-menu-item>
            <el-menu-item index="tool" @click="goCurrent('tool')">
              <template #title>
                <el-icon>
                  <img src="../assets/plugin.svg" width="22px" height="22px" />
                </el-icon>
                <span>工具</span>
              </template>
            </el-menu-item>
            <el-menu-item index="agent-skill" @click="goCurrent('agent-skill')">
              <template #title>
                <el-icon>
                  <img src="../assets/skill.svg" width="22px" height="22px" />
                </el-icon>
                <span>Skill</span>
              </template>
            </el-menu-item>
            <el-menu-item index="model" @click="goCurrent('model')">
              <template #title>
                <el-icon>
                  <img src="../assets/model.svg" width="22px" height="22px" />
                </el-icon>
                <span>模型</span>
              </template>
            </el-menu-item>
            <el-menu-item index="dashboard" @click="goCurrent('dashboard')">
              <template #title>
                <el-icon>
                  <img src="../assets/dashboard.svg" width="22px" height="22px" />
                </el-icon>
                <span>数据看板</span>
              </template>
            </el-menu-item>
          </el-menu>
          
          <!-- 底部帮助链接 -->
          <div class="sidebar-footer">
            <div class="help-links">
              <a
                href="https://github.com/Shy2593666979/AgentChat"
                target="_blank"
                class="help-link"
                title="GitHub 仓库"
              >
                <img
                  src="../assets/github.png"
                  alt="GitHub"
                  class="help-icon"
                />
              </a>
              <a
                href="https://shy2593666979.github.io/agentchat-docs/"
                target="_blank"
                class="help-link"
                title="帮助文档"
              >
                <img
                  src="../assets/help.png"
                  alt="帮助文档"
                  class="help-icon"
                />
              </a>
            </div>
          </div>
        </div>
      </el-col>
      <div class="content">
        <router-view v-slot="{ Component }">
          <transition name="page-fade" mode="out-in">
            <component :is="Component" />
          </transition>
        </router-view>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import url('https://fonts.googleapis.com/css2?family=ZCOOL+KuaiLe&family=Zhi+Mang+Xing&family=Ma+Shan+Zheng&display=swap');

.ai-body {
  overflow: hidden;

  .ai-nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 64px;
    background: var(--gradient-subtle);
    padding: 0 24px;
    box-shadow: 0 1px 3px rgba(124, 58, 237, 0.06);
    position: relative;
    z-index: 3000;
    border-bottom: 1px solid var(--border-light);

    .left {
      display: flex;
      align-items: center;
      font-weight: 600;
      color: var(--text-primary);
      cursor: pointer;
      transition: all var(--transition-base);

      &:hover {
        .logo { filter: drop-shadow(0 0 8px rgba(124, 58, 237, 0.3)); }
      }

      .item-img {
        margin-right: 0;
        animation: float 3s ease-in-out infinite;

        .logo {
          width: 36px;
          height: 36px;
          filter: drop-shadow(0 2px 4px rgba(124, 58, 237, 0.15));
          transition: all var(--transition-base);
        }
      }

      .nav-links {
        display: flex;
        align-items: center;
        margin-left: 8px;
        gap: 10px;

        .brand-logo-img {
          height: 42px;
          width: auto;
          display: block;
          filter: drop-shadow(0 2px 6px rgba(124, 58, 237, 0.15));
          user-select: none;
        }

        .brand-title {
          font-family: 'Zhi Mang Xing', 'Ma Shan Zheng', 'ZCOOL KuaiLe', 'PingFang SC', 'Microsoft YaHei', sans-serif;
          font-size: 28px;
          font-weight: 700;
          letter-spacing: 0.5px;
          background: var(--gradient-primary);
          background-size: 200% 200%;
          -webkit-background-clip: text;
          -webkit-text-fill-color: transparent;
          background-clip: text;
          animation: gradient-shift 4s ease infinite;
          user-select: none;
        }
      }
    }

    .right {
      display: flex;
      align-items: center;

      .user-info {
        .user-avatar-wrapper {
          display: flex;
          align-items: center;
          padding: 3px;
          border-radius: 50%;
          background: var(--gradient-primary);
          cursor: pointer;
          transition: all var(--transition-base);

          &:hover {
            transform: translateY(-2px);
            box-shadow: var(--shadow-colorful);
          }

          .user-avatar {
            img {
              border: 2px solid var(--bg-secondary);
              object-fit: cover;
              transition: all var(--transition-base);
              border-radius: 50%;

              &:hover {
                transform: scale(1.05);
              }
            }
          }
        }
      }
    }
  }

  .ai-main {
    display: flex;
    height: calc(100vh - 64px);
    background-color: var(--bg-primary);

    :deep(.el-col-2) {
      display: flex;
      width: 190px;
      min-width: 190px;
    }

    .sidebar-content {
      display: flex;
      flex-direction: column;
      height: 100%;
      width: 100%;
      background: var(--bg-secondary);
      border-right: 1px solid var(--border-light);
      padding: 8px 0;
      box-sizing: border-box;
    }

    .sidebar-footer {
      margin-top: auto;
      padding: 0 24px 24px;
      display: none;
      justify-content: center;
      align-items: center;

      .help-links {
        display: flex;
        gap: 16px;

        .help-link {
          display: flex;
          align-items: center;
          justify-content: center;
          width: 44px;
          height: 44px;
          border: 1px solid var(--border);
          border-radius: var(--radius-md);
          padding: 8px;
          transition: all var(--transition-base);
          background: var(--bg-tertiary);

          &:hover {
            background: var(--gradient-subtle);
            border-color: var(--accent-light);
            transform: translateY(-2px);
            box-shadow: var(--shadow-colorful);
          }

          .help-icon {
            width: 24px;
            height: 24px;
            transition: all var(--transition-base);
          }

          &:hover .help-icon {
            transform: scale(1.1);
          }
        }
      }
    }

    .content {
      flex: 1;
      overflow-y: auto;
      background-color: var(--bg-primary);
    }
  }
}

// 下拉菜单样式
:deep(.el-dropdown-menu) {
  border: 1px solid var(--border);
  box-shadow: var(--shadow-lg);
  border-radius: var(--radius-md);
  overflow: hidden;
  background: var(--bg-secondary);
}

// 菜单样式优化
:deep(.el-menu-vertical-demo) {
  border-right: none;
  background: transparent;

  .el-menu-item {
    border-radius: var(--radius-md);
    margin: 4px 12px;
    padding: 0 16px !important;
    height: 46px;
    line-height: 46px;
    font-size: 14px;
    font-weight: 500;
    transition: all var(--transition-base);
    position: relative;
    color: var(--text-secondary);
    border: 1px solid transparent;

    &::before {
      content: '';
      position: absolute;
      left: 0;
      top: 50%;
      transform: translateY(-50%) scaleY(0);
      width: 3px;
      height: 20px;
      background: var(--gradient-primary);
      border-radius: 0 3px 3px 0;
      transition: transform var(--transition-base);
    }

    &:hover {
      background: var(--accent-glow);
      color: var(--accent);

      &::before { transform: translateY(-50%) scaleY(0.6); }
      .el-icon img { transform: scale(1.08); }
    }

    &.is-active {
      background: var(--accent-glow);
      color: var(--accent);
      font-weight: 700;
      border-color: transparent;

      &::before {
        transform: translateY(-50%) scaleY(1);
        background: var(--gradient-primary);
      }

      .el-icon img { filter: none; }
    }

    .el-icon {
      margin-right: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 24px;
      height: 24px;

      img {
        width: 24px;
        height: 24px;
        transition: all var(--transition-base);
      }
    }

    span {
      font-size: 14px;
      font-weight: inherit;
    }
  }

  .el-sub-menu {
    .el-sub-menu__title {
      border-radius: var(--radius-md);
      margin: 4px 12px;
      padding: 0 16px !important;
      height: 46px;
      line-height: 46px;
      font-size: 14px;
      font-weight: 500;
      transition: all var(--transition-base);
      color: var(--text-secondary);

      &:hover {
        background: var(--accent-glow);
        color: var(--accent);
      }

      .el-icon {
        margin-right: 12px;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 24px;
        height: 24px;

        img {
          width: 24px;
          height: 24px;
        }
      }
    }

    .el-menu {
      background: transparent;

      .el-menu-item {
        margin: 2px 12px 2px 40px !important;
        height: 40px !important;
        line-height: 40px !important;
        border-radius: var(--radius-sm);

        &.is-active {
          background: var(--accent-glow);
          color: var(--accent);
        }
      }
    }
  }
}

/* 页面转场动画 */
.page-fade-enter-active,
.page-fade-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}
.page-fade-enter-from {
  opacity: 0;
  transform: translateY(10px);
}
.page-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
