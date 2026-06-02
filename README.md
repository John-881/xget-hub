# Xget Hub UI 优化说明

> **文件**: `xget-ui.html`
> **日期**: 2026-06-02
> **基于**: [Xget v2.x 文档](https://github.com/xixu-me/Xget)（80+ 平台支持）

纯静态开发者资源加速仪表盘，提供 URL 转换、配置生成、实例管理和连接诊断。

---

## 目录

- [快速开始](#快速开始)
- [功能模块](#功能模块)
- [修复清单](#修复清单)
- [架构说明](#架构说明)
- [平台支持](#平台支持)
- [浏览器兼容性](#浏览器兼容性)

---

## 快速开始

1. 直接在浏览器打开 `xget-ui.html`
2. 粘贴原始 URL → 自动识别平台 → 生成 Xget 加速链接
3. 通过侧栏「实例设置」管理自建 Xget 实例

无需构建工具、无需后端，零依赖运行（Bootstrap + Icons 从 CDN 加载）。

---

## 功能模块

### 1. URL 转换器

- 输入原始 URL，自动检测所属平台（50+ 域名精确匹配）
- 支持手动切换平台覆盖自动检测
- 生成 Xget 加速链接 + 一键复制
- 快速生成 `wget` / `curl` 下载命令

**转换示例**:

| 原始 URL | Xget 加速链接 |
|----------|--------------|
| `https://github.com/user/repo/releases/download/v1.0/file.zip` | `https://xget.xi-xu.me/gh/user/repo/releases/download/v1.0/file.zip` |
| `https://huggingface.co/microsoft/DialoGPT-medium/resolve/main/model.bin` | `https://xget.xi-xu.me/hf/microsoft/DialoGPT-medium/resolve/main/model.bin` |
| `docker pull nginx:latest` | `https://xget.xi-xu.me/cr/docker/library/nginx:latest` |
| `https://api.openai.com/v1/chat/completions` | `https://xget.xi-xu.me/ip/openai/v1/chat/completions` |

### 2. 配置生成器

自动使用当前默认实例域名，为以下工具生成一键配置：

| 工具 | 配置方式 |
|------|----------|
| Git | `url.*.insteadOf` 全局替换 |
| pip / PyPI | `index-url` + `trusted-host` |
| npm / Bun | `registry` 配置 / `.npmrc` |
| Docker | `docker pull` 命令 + `daemon.json` registry-mirrors |
| Conda | `channel_alias` + `default_channels` |

切换默认实例后所有配置自动更新。

### 3. 测速·诊断

- **连接诊断**: 双重探测策略（CORS HEAD → Image ping），准确区分可达/不可达
- **模拟测速**: 内置 VS Code / HuggingFace / Apache Kafka 测试文件链接
- **加载状态**: 按钮在探测期间显示 spinner 并禁用，完成后恢复

### 4. 实例管理（Offcanvas 侧滑面板）

- 添加/删除自定义 Xget 实例（自建 Workers/Pages 域名）
- 设默认实例，一键切换
- 实例域名格式校验 + 去重
- 所有数据持久化到 `localStorage`
- 新实例连接测试（带 loading 状态）

---

## 修复清单

### 关键功能（7 项）

| # | 修复项 | 问题描述 | 修复方案 |
|---|--------|----------|----------|
| 1 | **Ping 假阳性** | `fetch(no-cors)` 无法读取 HTTP 状态码，任何可达地址都显示成功 | 双层策略：先 CORS HEAD（8s 超时），失败回退 `Image()` 探测 |
| 2 | **平台误识别** | `host.includes()` 导致 `gist.github.com` 匹配到 GitHub | 改为 `Map` 精确 host 匹配 + 子域名最长前缀排序 |
| 3 | **Homebrew 路径错误** | `/Homebrew/` 前缀未剥离，生成 `/homebrew/Homebrew/...` | `pathRewrite` 自动剥离 `/Homebrew` 前缀 |
| 4 | **递归渲染** | `deleteInstance` → `loadInstances()` → `renderInstanceTable()` 导致双重序列化 | 直接操作数组 + `refreshAllUI()` |
| 5 | **双重初始化** | `loadInstances()` 和 `window.load` 都触发 `convertUrl()` | 移除 `window.load` 监听 |
| 6 | **输入未清空** | 添加实例后 alias/url 输入框残留旧值 | `addInstance()` 成功后重置输入 |
| 7 | **Docker 路径错误** | `registry-1.docker.io/v2/...` 的 `/v2/` 前缀未剥离 | `pathRewrite` 自动剥离 `/v2/` |

### UX / 可访问性（5 项）

| # | 修复项 | 问题描述 | 修复方案 |
|---|--------|----------|----------|
| 8 | **嵌套标签无 ARIA** | 配置面板的子标签缺少无障碍属性 | 所有标签添加 `role` / `aria-controls` / `aria-selected` / `aria-labelledby` |
| 9 | **alert() 弹窗** | 复制、添加、报错全部使用 `alert()` | Bootstrap Toast 组件，2.5s 自动消失 |
| 10 | **剪贴板无回退** | 非 HTTPS / 旧浏览器 `navigator.clipboard` 不可用 | 添加 `document.execCommand('copy')` fallback |
| 11 | **无加载态** | 异步操作期间按钮无反馈 | `btn-loading` 样式切换 + spinner 指示器 |
| 12 | **空输入显示垃圾 URL** | 未输入时显示 `https://xget.xi-xu.me/gh` | 显示引导文本"请输入 URL 查看转换结果"，复制按钮置灰 |

### 平台准确性（3 项）

| # | 修复项 | 问题描述 | 修复方案 |
|---|--------|----------|----------|
| 13 | **缺少 7 个平台** | OpenRouter、Perplexity、xAI 等未收录 | 新增至 55 个平台条目，对齐 xget 文档 |
| 14 | **Homebrew host 映射错误** | `host: "github.com/Homebrew"` 无法通过 hostname 检测 | 分离为独立条目，`detectPlatform()` 检测 `/Homebrew/` 路径 |
| 15 | **重写覆盖手动选择** | `pathRewrite` 无条件执行，覆盖用户手动选择的平台 | 仅当 `detected.prefix === selectedPrefix` 时应用 |

### 代码质量（4 项）

| # | 修复项 | 问题描述 | 修复方案 |
|---|--------|----------|----------|
| 16 | **CSS transition: all** | 所有属性变化都触发动画，性能差 | 改为 `transform 0.15s, box-shadow 0.15s` |
| 17 | **重复绑定事件** | 每次 `renderInstanceTable()` 重新 `querySelectorAll` + `addEventListener` | 事件委托：`tbody` 上单次监听 `[data-action]` |
| 18 | **按钮缺少 type** | 部分 `<button>` 未声明 `type="button"` | 所有非提交按钮添加 |
| 19 | **无输入防抖** | URL 每次击键都触发完整转换流程 | 200ms 防抖 |

---

## 架构说明

### CSS 组织

```
Tokens (设计变量) → Base (基础) → Navbar → Cards → Buttons
→ Pills/Tabs → Forms → Code → Table → Indicators → Footer
→ Hover → Toast → Loading → Responsive
```

### JavaScript 模块

```
XgetApp (IIFE, "use strict")
├── CONSTANTS        — STORAGE_KEY, DEFAULT_INSTANCES, DEBOUNCE_MS
├── PLATFORM_REGISTRY — 55 个平台定义（name/prefix/host/pathRewrite）
├── HOST_LOOKUP      — host → platform[] 精确匹配 Map
├── State            — instances[], currentInstanceUrl
├── UI Helpers       — showToast(), copyToClipboard(), copyAndNotify(),
│                       setButtonLoading(), escapeHtml()
├── Storage          — loadInstances(), saveInstances()
├── InstanceManager  — setDefaultInstance(), deleteInstance(), addInstance()
├── Converter        — detectPlatform(), convertUrl()
├── ConfigGenerator  — CONFIG_TEMPLATES, updateConfigSnippets()
├── PingService      — pingInstance()（CORS → Image 双层策略）
├── UI Refreshers    — refreshAllUI(), refreshInstanceBadge(),
│                       renderInstanceTable(), populatePlatformSelect()
├── Event Delegation — tbody[data-action] 监听
└── EventBindings    — bindEvents() 集中式事件注册
```

### 数据流

```
localStorage ←→ instances[] ←→ currentInstanceUrl
                                  ↓
           ┌──────────────────────┼──────────────────────┐
           ↓                      ↓                      ↓
     URL 转换器              配置生成器              测速面板
     convertUrl()       updateConfigSnippets()   updateSpeedTestUrl()
```

### 关键设计决策

**Ping 双层探测**：Cloudflare Workers 可能设置也可能不设置 CORS 头。先尝试 CORS HEAD（最准确），失败后回退 Image ping（无 CORS 限制，通过 onload/onerror 判断连通性）。

**平台检测优先级**：
1. 路径特殊检测（github.com + /Homebrew/ → Homebrew）
2. host 精确匹配（HOST_LOOKUP Map）
3. 子域名后缀匹配（最长优先）

**pathRewrite 保护**：仅当检测到的平台与用户当前选择的平台一致时才应用路径重写，防止覆盖用户手动选择。

---

## 平台支持

共计 **55 个平台**，覆盖 6 大类别：

### 代码仓库（7）
GitHub · GitHub Gist · GitLab · Gitea · Codeberg · SourceForge · AOSP

### AI/ML 模型托管（2）
Hugging Face · Civitai

### 包管理器（14）
npm · PyPI · Conda · Maven · Gradle · Homebrew · RubyGems · CRAN · Go Proxy · NuGet · Crates (Rust) · Packagist · Flathub · Apache

### 容器镜像仓库（4）
Docker Hub · Quay.io · GCR · GHCR

### AI 推理 API（12）
OpenAI · Anthropic · Gemini · Cohere · Mistral · Groq · Together · OpenRouter · Perplexity · xAI · Replicate · HuggingFace Inference

### Linux 发行版（6）
Debian · Ubuntu · Fedora · Rocky Linux · openSUSE · Arch Linux

### 其他（3）
arXiv · F-Droid · Jenkins

---

## 浏览器兼容性

| 特性 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| 基础功能 | 90+ | 90+ | 15+ | 90+ |
| Clipboard API | 66+ | 63+ | 13.1+ | 79+ |
| 剪贴板回退 | ✅ | ✅ | ✅ | ✅ |
| Fetch + AbortController | 66+ | 57+ | 12.1+ | 79+ |
| CSS 自定义属性 | 49+ | 31+ | 9.1+ | 15+ |

> 所有现代浏览器均支持。旧浏览器通过剪贴板 fallback 和 Image ping 回退正常工作。
