# 快速启动脚本

一键启动前后端开发服务。

## 环境要求

- Python 3.12+
- Node.js + npm

## 使用

```bash
# 在项目根目录执行
python scripts/start.py
```

脚本自动执行：
1. 安装 Python 依赖
2. 启动后端 (port 7860)
3. 启动前端 (`npm run dev`)

按 `Ctrl+C` 停止全部服务。

## 常见问题

- **npm 找不到**：确认 Node.js 已安装且加入 PATH
- **依赖安装失败**：确认 `requirements.txt` 在项目根目录下
