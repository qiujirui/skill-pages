---
name: ngrok-tunnel
description: "内网穿透，将本地服务暴露到公网。当用户提到'内网穿透'、'公网访问'、'外网访问'、'远程访问'、'ngrok'、'tunnel'、'让别人看到'、'外部访问'、'穿透到公网'时，使用本 Skill。"
version: 1.1.0
---

# ngrok-tunnel Skill

将本地 HTTP 服务通过 ngrok 暴露到公网，让外部用户可以通过公网 URL 访问本地页面。

## 重要更新 (v1.1.0)

⚠️ **pyngrok 库有严重 bug**：调用 `ngrok.connect()` 会返回成功消息但实际不启动 ngrok 进程！

**正确方法**：直接调用 ngrok.exe 本人，不要用 pyngrok 库。

## 前置依赖

- ngrok.exe（已安装在 `C:\Users\qiujirui\AppData\Local\ngrok\ngrok.exe`）
- ngrok 账号 + authtoken（已配置在 config.env）
- 本地 HTTP 服务器（默认端口 8099）

## 安全特性

**本 Skill 使用安全 HTTP 服务器，默认启用以下防护：**

| 防护措施 | 说明 |
|----------|------|
| 禁止目录列表 | 访问文件夹不会显示文件列表（返回 403）|
| 白名单限制 | 只允许 HTML/CSS/JS/图片等媒体文件 |
| 黑名单过滤 | 禁止访问 .env, config, 密码文件等敏感文件 |
| 路径遍历防护 | 阻止 `../` 等方式访问上级目录 |

**允许的文件类型：** `.html`, `.htm`, `.css`, `.js`, `.json`, `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.svg`, `.ico`, `.woff`, `.woff2`, `.ttf`, `.mp4`, `.webm`, `.mp3`, `.wav`, `.pdf`, `.txt`, `.xml`, `.md`

**禁止的文件类型：** `.py`, `.exe`, `.bat`, `.sh`, `.zip`, `.rar`, `.7z`, `.env` 等

## 使用流程

### ✅ 推荐：直接调用 ngrok.exe（最可靠）

```powershell
# 方法1：直接运行 ngrok.exe
C:\Users\qiujirui\AppData\Local\ngrok\ngrok.exe http 8099

# 方法2：使用启动脚本（推荐）
python D:\WorkBuddyOutput\ngrok_fixed.py
```

⚠️ **不要使用 pyngrok 库的 ngrok.connect()**，它有严重bug：显示成功但实际不启动进程！

### 一键启动（旧方法，已不推荐）

```bash
python scripts/tunnel.py start
```

## 配置说明

| 配置项 | 位置 | 说明 |
|--------|------|------|
| AUTHTOKEN | config.env | ngrok authtoken，从 dashboard.ngrok.com 获取 |
| 默认端口 | 8099 | 本地 HTTP 服务器端口 |
| 根目录 | D:\WorkBuddyOutput | HTTP 服务器根目录 |

## 注意事项

- ngrok 免费版每次重启后公网地址会变
- 首次访问 ngrok 地址会出现"Visit Site"确认页，点击即可
- authtoken 失效时需重新配置 config.env
- 重启电脑后需要重新运行启动命令

## 故障排查

| 问题 | 原因 | 解决 |
|------|------|------|
| ngrok.connect()返回成功但外部无法访问 | pyngrok库的bug，进程未实际启动 | 直接调用ngrok.exe，不要用pyngrok |
| 本地8099正常但公网无法访问 | ngrok进程未运行 | 检查tasklist是否有ngrok.exe |
| 隧道建立但无法访问 | 本地服务器未启动 | 先运行scripts/tunnel.py start-server |
| authtoken错误 | token过期或格式错误 | 去ngrok Dashboard重新获取 |
| 端口被占用 | 8099已有其他进程 | 修改脚本中的DEFAULT_PORT |
| 访问出现Visit Site页面 | ngrok免费版正常行为 | 点击确认即可 |
| ERR_NGROK_3200 | 隧道已离线 | 重新运行启动命令 |

### 如何判断问题

```powershell
# 检查本地服务器
curl http://localhost:8099/

# 检查ngrok进程
tasklist | findstr ngrok.exe

# 检查ngrok API
curl http://127.0.0.1:4040/api/tunnels
```

## 踩坑经验

- pyngrok ngrok.connect() / 连接成功但外部无法访问：pyngrok内部调用有bug，显示成功但实际未启动ngrok进程。必须直接调用ngrok.exe
- tasklist找不到ngrok.exe / 4040端口无响应：确认ngrok进程未运行，直接运行ngrok.exe即可
- 本地正常但外部失败 / 无进程：问题在ngrok侧，不是本地服务器

## 文件结构

```
ngrok-tunnel/
├── SKILL.md          # 技能说明
├── config.env        # authtoken 配置
└── scripts/
    └── tunnel.py     # 核心脚本
```