---
name: sdd:reverse-engineer
description: 逆向文档工程 — 对无源码的已有系统进行截图、内容抓取，生成结构化功能文档，为后续原型重建提供输入
---

# SDD Reverse Engineer — 逆向文档工程

## 触发方式
用户输入 `/sdd:reverse-engineer`

## 适用场景
- 需要对没有源码的已有系统进行功能摸底
- 需要为原型重建提供参考文档
- 需要记录一个运行中系统的现状快照

---

## 工具选择

**首选：`agent-browser`**（Vercel Labs 出品的原生 Rust CLI）

```bash
# 检查是否已安装
agent-browser --version

# 未安装则执行
npm install -g agent-browser
agent-browser install   # 下载 Chrome for Testing（如系统已有 Chrome 会自动检测）
```

**备选：`playwright` Node.js 脚本**（agent-browser 不可用时降级）

---

## 执行步骤

### 第0步：收集基本信息

向用户询问：
1. **目标系统地址**：完整 URL（如 `http://localhost:5174`）
2. **是否需要登录**：是/否；如需登录，询问账号密码（或告知手动登录）

---

### 第1步：创建工作目录

```bash
mkdir -p SDD/snapshot/screenshots
```

---

### 第2步：启动浏览器并登录

```bash
# 打开目标系统（agent-browser 守护进程会持续运行）
agent-browser open <URL>
```

#### 登录策略

**策略A — 自动登录**（已知账号密码）：
```bash
agent-browser find placeholder "用户名" fill "admin"
agent-browser find placeholder "密码" fill "Admin123"
agent-browser find role button click --name "登 录"
agent-browser wait --url "**/[^login]*"
```

**策略B — 手动登录**（未知密码，需弹出可视浏览器）：
```bash
# agent-browser 默认有界面，用户直接在窗口中操作
# 等待 URL 变为非 login 页
agent-browser wait --url "**/[^login]*" --timeout 120000
```

**登录后提取 token（供后续页面注入）**：
```bash
agent-browser storage local auth_token
# 记录返回的 token 值
```

---

### 第3步：路由发现

**优先：读取源码**
如果能定位到前端源码目录，直接读取路由定义文件（`App.tsx`、`router/index.ts` 等），提取所有路由路径，这是最准确的方式。

定位方式：
```bash
# 通过 script src 推断源码路径
agent-browser eval "JSON.stringify(Array.from(document.querySelectorAll('script[src]')).map(s=>s.src))"
```

**其次：探测框架路由**
```bash
# React Router
agent-browser eval "window.__reactRouterVersion"

# 如果是 React Router，从 fiber 树提取路由
agent-browser eval "
  const el = document.getElementById('root');
  const k = Object.keys(el).find(k => k.startsWith('__reactFiber'));
  // 遍历 fiber 找 router.routes
"
```

**兜底：点击导航发现路由**
```bash
agent-browser snapshot   # 获取无障碍树，找到所有可点击的导航项
# 逐一点击，记录 URL 变化
agent-browser get url
```

---

### 第4步：批量截图

已知路由后，用 `batch` 命令一次性完成所有页面截图：

```bash
agent-browser batch \
  "open <BASE_URL>/" \
  "wait --load networkidle" \
  "screenshot SDD/snapshot/screenshots/00_home.png --full" \
  "open <BASE_URL>/assistant" \
  "wait --load networkidle" \
  "screenshot SDD/snapshot/screenshots/01_assistant.png --full" \
  "open <BASE_URL>/skills" \
  "wait --load networkidle" \
  "screenshot SDD/snapshot/screenshots/02_skills.png --full" \
  "open <BASE_URL>/cards" \
  "wait --load networkidle" \
  "screenshot SDD/snapshot/screenshots/03_cards.png --full"
```

如页面需要登录验证（JWT 存在 localStorage），在每次 `open` 后注入 token：
```bash
agent-browser storage local set auth_token "<token>"
agent-browser open <URL>
agent-browser wait --load networkidle
```

#### 截图补充：Tab / 子状态

对每个页面内的 Tab、下拉、弹窗等状态，额外截图：
```bash
agent-browser find role tab click --name "自选"
agent-browser wait 500
agent-browser screenshot SDD/snapshot/screenshots/home_tab_自选.png --full
```

#### 截图补充：注解截图

生成带元素标注的截图，便于后续原型对照：
```bash
agent-browser screenshot SDD/snapshot/screenshots/home_annotated.png --annotate
```

---

### 第5步：提取结构化内容

每个页面截图后，用 `snapshot` 获取无障碍树（比 DOM 更语义化）：

```bash
agent-browser open <URL>/<route>
agent-browser wait --load networkidle
agent-browser snapshot > SDD/snapshot/content/<label>.txt
```

用 `eval` 提取关键信息：
```bash
agent-browser eval "JSON.stringify({
  title: document.title,
  url: location.href,
  headings: Array.from(document.querySelectorAll('h1,h2,h3,h4')).map(h=>({tag:h.tagName,text:h.innerText.trim()})).filter(h=>h.text),
  buttons: [...new Set(Array.from(document.querySelectorAll('button')).map(b=>b.innerText.trim()).filter(t=>t&&t.length<50))],
  inputs: Array.from(document.querySelectorAll('input,textarea')).map(i=>({type:i.type,placeholder:i.placeholder,label:i.getAttribute('aria-label')||''}))
}, null, 2)"
```

---

### 第6步：读取源码（可选增强）

如能定位前端源码，额外读取：
- 路由文件（`App.tsx`、`router/index.ts`）
- API 客户端（`api/client.ts`、`services/*.ts`）
- 数据类型（`types.ts`、`interfaces/*.ts`）
- 核心页面组件

源码补充截图无法获取的内容：API 接口列表、数据结构、业务逻辑、未实现占位功能。

---

### 第7步：生成系统功能文档

读取所有截图和内容，结合源码（如有），生成 `SDD/snapshot/system-overview.md`。

文档结构：

```markdown
# <系统名> — 系统功能文档

## 1. 系统定位

## 2. 整体架构
- 技术栈
- 主题色彩规范
- 页面路由结构（ASCII 树）
- 整体布局结构（ASCII 图）

## 3. 全局布局组件
每个组件：职责 / UI 内容表 / 主要交互

## 4. 页面详细说明
每个页面：路由 / 功能定位 / UI 区域表 / 主要交互 / API 接口表

## 5. 数据类型定义

## 6. 认证机制

## 7. 全量 API 接口汇总表

## 8. 待实现功能
```

---

### 第8步：关闭浏览器 & 输出结果

```bash
agent-browser close
```

完成后告知用户：

```
✅ 逆向工程完成

产出物：
- 功能文档：SDD/snapshot/system-overview.md
- 截图：SDD/snapshot/screenshots/（N 张）
- 无障碍树：SDD/snapshot/content/（N 个文件）

下一步：
通过 /sdd:new → /sdd:propose 生成 prototypes.html，开始原型重建。
```

---

## 常见问题

| 问题 | 原因 | 解决方式 |
|------|------|----------|
| 页面只截到登录页 | Token 未注入 | `agent-browser storage local set auth_token <token>` 后 reload |
| batch 中途失败 | 某步命令出错 | 加 `--bail` 参数，逐步调试失败命令 |
| 页面内容未渲染完 | JS 渲染慢 | 在 `screenshot` 前加 `wait --load networkidle` 或 `wait 2000` |
| SPA 路由发现不了 | 无 `<a href>` | 优先读源码路由文件，或用 `snapshot` + 点击导航逐一发现 |
| agent-browser 未安装 | 环境缺失 | `npm install -g agent-browser && agent-browser install` |

---

## 注意事项

- 账号密码仅用于本地自动化，不对外传输
- 截图保存于本地 `SDD/snapshot/screenshots/`，完成后可按需清理
- `agent-browser` 守护进程会持续运行直到 `agent-browser close`，多步操作共享同一浏览器会话
