# 汉字书写练习应用 — 完整实现文档

## 目录

1. [项目概述](#1-项目概述)
2. [文件结构与功能说明](#2-文件结构与功能说明)
3. [技术架构](#3-技术架构)
4. [完整代码实现：hanzi-writer.html](#4-完整代码实现hanzi-writerhtml)
5. [功能模块详解](#5-功能模块详解)
6. [汉字数据加载流程](#6-汉字数据加载流程)
7. [用户交互事件处理机制](#7-用户交互事件处理机制)
8. [状态管理逻辑](#8-状态管理逻辑)
9. [从零实现教程](#9-从零实现教程)
10. [调试技巧与常见问题](#10-调试技巧与常见问题)

---

## 1. 项目概述

本项目是一个基于 **Hanzi Writer** 开源库的汉字笔顺动画与书写练习 Web 应用。项目采用 **xmcp**（MCP 框架）+ **Next.js** 双轨架构，既可以作为 MCP 工具被 AI 调用，也可以作为独立 Web 页面在浏览器中运行。

### 核心功能

- **笔顺动画**：逐笔展示汉字书写顺序，支持单次播放和循环播放
- **书写练习**：用户在画布上手写，系统实时检测笔顺正确性并给出反馈
- **汉字切换**：支持输入任意简体/繁体汉字并即时加载
- **速度调节**：动画播放速度 x1 / x2 / x5 三档可调
- **显示控制**：可切换是否显示汉字本体、轮廓线
- **练习模式**：简单模式（带轮廓提示）/ 困难模式（无轮廓）

### 两套实现

项目包含两套功能等价的实现，分别用于不同场景：

| 实现方式 | 文件路径 | 使用场景 |
|---------|---------|---------|
| xmcp 工具（React） | `src/tools/hanzi-writer.tsx` | 作为 MCP 工具被 AI 调用，嵌入在 MCP 客户端中 |
| 独立 HTML | `web/public/mcp-apps/hanzi-writer.html` | 通过 iframe 嵌入 Next.js 页面，或直接访问 |

---

## 2. 文件结构与功能说明

### 2.1 完整项目结构

```
f:\hanzi-write\
├── .xmcp/                          # xmcp 框架内部文件（自动生成）
│   ├── headers.js
│   ├── http.js
│   └── import-map.js
├── dist/                           # xmcp 构建输出（自动生成）
│   ├── client/                     # 客户端 bundle
│   ├── http.js                      # HTTP 服务器入口
│   └── src_tools_*.js              # 各工具的编译输出
├── src/
│   └── tools/                       # MCP 工具定义目录
│       ├── get-weather.ts           # 天气查询工具（示例）
│       ├── hanzi-writer.tsx         # ★ 汉字书写工具（React 版本）
│       └── weather.tsx              # 天气工具（React 版本）
├── web/                             # Next.js Web 前端
│   ├── app/
│   │   ├── globals.css              # 全局样式（Tailwind）
│   │   ├── layout.tsx               # 根布局
│   │   └── page.tsx                 # 首页（含 Inspector 面板）
│   ├── components/
│   │   └── McpUiRenderer.tsx        # UI 资源渲染器（iframe 模态框）
│   ├── public/
│   │   └── mcp-apps/
│   │       └── hanzi-writer.html    # ★ 汉字书写页面（独立 HTML 版本）
│   ├── package.json                 # Web 项目依赖
│   └── tsconfig.json
├── globals.css                      # xmcp 工具全局样式
├── package.json                     # 根项目依赖（xmcp）
├── xmcp.config.ts                   # xmcp 配置
└── xmcp.svg                         # 项目图标
```

### 2.2 核心文件功能详解

| 文件 | 路径 | 功能说明 |
|-----|------|---------|
| **hanzi-writer.html** | `web/public/mcp-apps/hanzi-writer.html` | 独立 HTML 版本，集成 Hanzi Writer 库，包含笔顺动画和书写练习的完整功能 |
| **hanzi-writer.tsx** | `src/tools/hanzi-writer.tsx` | xmcp MCP 工具版本，用 React + TypeScript 实现，作为 MCP Tool 被注册和调用 |
| **McpUiRenderer.tsx** | `web/components/McpUiRenderer.tsx` | Next.js 中的 UI 资源渲染组件，将 `ui://app/xxx` 协议的 URI 映射为 `/mcp-apps/xxx.html` 并通过 iframe 加载 |
| **page.tsx** | `web/app/page.tsx` | Next.js 首页，提供 Inspector 面板用于粘贴 JSON 数据并加载 UI 资源 |
| **layout.tsx** | `web/app/layout.tsx` | Next.js 根布局组件 |
| **get-weather.ts** | `src/tools/get-weather.ts` | xmcp 工具示例，演示标准工具的定义方式 |
| **xmcp.config.ts** | `xmcp.config.ts` | xmcp 框架配置，指定工具路径、HTTP 模式等 |

---

## 3. 技术架构

### 3.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                           用户 / AI 客户端                            │
└─────────────┬───────────────────────────────────┬───────────────────┘
              │                                   │
              │ MCP 协议调用                     │ 浏览器访问
              ▼                                   ▼
┌─────────────────────────┐           ┌───────────────────────────────┐
│   xmcp 服务器 (:port)   │           │   Next.js Dev Server (:3000)  │
│  - 注册 MCP Tools       │           │  - 提供页面路由               │
│  - 编译工具代码         │           │  - 渲染 Inspector 面板        │
│  - 提供 HTTP 传输       │           │  - 通过 iframe 加载 HTML 资源 │
└──────────┬──────────────┘           └──────────────┬────────────────┘
           │                                           │
           │ 工具组件                                   │ iframe src
           ▼                                           ▼
┌──────────────────────────┐              ┌──────────────────────────────┐
│ src/tools/hanzi-writer.tsx│              │ public/mcp-apps/            │
│  (React + TypeScript)    │              │   hanzi-writer.html         │
└──────────────────────────┘              │  (原生 HTML + JS)           │
                                            └──────────────┬───────────────┘
                                                           │
                                                           │ CDN 加载
                                                           ▼
                                            ┌──────────────────────────────┐
                                            │  jsDelivr CDN                │
                                            │  - hanzi-writer@3.7 库      │
                                            │  - hanzi-writer-data@2.0    │
                                            │    (汉字笔画 JSON 数据)      │
                                            └──────────────────────────────┘
```

### 3.2 技术栈

| 层级 | 技术 | 版本 | 用途 |
|-----|------|------|------|
| 框架（MCP） | xmcp | ^0.6.13 | MCP 工具开发与服务框架 |
| 框架（Web） | Next.js | 15.1.6 | React 全栈框架，提供页面路由 |
| UI 库 | React | 19.2.3 | 组件化 UI 开发 |
| 样式 | Tailwind CSS | 4.x | 原子化 CSS 框架 |
| 核心库 | hanzi-writer | 3.5 / 3.7 | 汉字笔顺动画与描红练习引擎 |
| 数据源 | hanzi-writer-data | 2.0 | 汉字笔画 SVG 路径数据 |
| 类型校验 | Zod | 4.0.0 | MCP 工具参数 Schema 校验 |
| 语言 | TypeScript | 5.x | 类型安全 |

### 3.3 数据流

```
用户输入汉字
     │
     ▼
loadCharData(char) ──► 检查 charDataCache 缓存
     │                       │
     │                       ├─ 命中 ──► 直接返回缓存数据
     │                       │
     │                       └─ 未命中 ──► fetch(CDN) ──► 存入缓存
     │
     ▼
初始化/更新 HanziWriter 实例
     │
     ├─ charWriter (预览区) ──► animateCharacter() / loopCharacterAnimation()
     │
     └─ quizWriter (练习区) ──► quiz() / cancelQuiz()
```

---

## 4. 完整代码实现：hanzi-writer.html

以下是 `web/public/mcp-apps/hanzi-writer.html` 的完整代码。这是一个自包含的单文件 HTML，直接在浏览器中打开即可使用。

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hanzi Writer</title>
    <!-- 引入 Hanzi Writer 库（CDN） -->
    <script src="https://cdn.jsdelivr.net/npm/hanzi-writer@3.5/dist/hanzi-writer.min.js"></script>
    <style>
      /* ===== CSS 变量与主题 ===== */
      :root {
        color-scheme: dark;
        --bg: #09090b;
        --panel: #111113;
        --panel-2: #18181b;
        --border: rgba(255, 255, 255, 0.1);
        --text: #f4f4f5;
        --muted: #a1a1aa;
        --accent: #ffffff;
        --accent-2: #27272a;
      }

      * { box-sizing: border-box; }
      html, body {
        margin: 0;
        min-height: 100%;
        background: var(--bg);
        color: var(--text);
        font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
      }
      body { display: grid; place-items: center; padding: 24px; }

      /* ===== 主容器 ===== */
      .shell {
        width: min(100%, 960px);
        background: linear-gradient(180deg, var(--panel), #0c0c0d);
        border: 1px solid var(--border);
        border-radius: 20px;
        box-shadow: 0 24px 80px rgba(0, 0, 0, 0.45);
        overflow: hidden;
      }

      /* ===== 头部 ===== */
      .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        gap: 16px;
        padding: 16px 20px;
        border-bottom: 1px solid var(--border);
        background: rgba(255, 255, 255, 0.02);
      }
      .title { font-size: 14px; font-weight: 600; letter-spacing: 0.02em; }
      .subtitle {
        margin-top: 4px;
        font-size: 12px;
        color: var(--muted);
        font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
      }

      /* ===== 内容区布局 ===== */
      .content {
        display: grid;
        grid-template-columns: 1.2fr 0.8fr;
        gap: 20px;
        padding: 20px;
      }
      @media (max-width: 820px) {
        .content { grid-template-columns: 1fr; }
      }
      .card {
        background: var(--panel-2);
        border: 1px solid var(--border);
        border-radius: 16px;
        padding: 18px;
      }

      /* ===== 汉字输入 ===== */
      .char-input-row {
        display: flex;
        gap: 10px;
        margin-bottom: 14px;
      }
      .char-input-row input {
        flex: 1;
        background: #0f0f11;
        color: var(--text);
        border: 1px solid var(--border);
        border-radius: 10px;
        height: 42px;
        padding: 0 14px;
        font-size: 16px;
        outline: none;
      }
      .char-input-row input:focus { border-color: rgba(255, 255, 255, 0.3); }

      /* ===== 预览区 ===== */
      .character-area {
        display: grid;
        place-items: center;
        min-height: 300px;
      }
      #character-target {
        width: 260px;
        height: 260px;
      }
      .meta {
        display: flex;
        flex-wrap: wrap;
        gap: 12px;
        margin-top: 16px;
        color: var(--muted);
        font-size: 13px;
      }
      .loading {
        display: grid;
        place-items: center;
        color: var(--muted);
        font-size: 13px;
        min-height: 260px;
      }

      /* ===== 练习区 ===== */
      .quiz-area {
        margin-top: 16px;
        display: grid;
        gap: 12px;
      }
      #quiz-target {
        width: 100%;
        height: 280px;
        border-radius: 16px;
        background:
          linear-gradient(transparent 49%, rgba(255, 255, 255, 0.05) 50%),
          linear-gradient(90deg, transparent 49%, rgba(255, 255, 255, 0.05) 50%);
        background-size: 32px 32px;
        border: 1px solid rgba(255, 255, 255, 0.08);
      }
      .actions { display: flex; gap: 10px; flex-wrap: wrap; }
      .actions .btn { flex: 1 1 120px; }

      /* ===== 控件 ===== */
      .controls { display: grid; gap: 14px; }
      .btn {
        appearance: none;
        border: 1px solid var(--border);
        background: var(--accent-2);
        color: var(--text);
        height: 42px;
        padding: 0 14px;
        border-radius: 10px;
        cursor: pointer;
        transition: 0.2s ease;
        font-size: 14px;
      }
      .btn:hover { background: #34343a; }
      .btn:disabled { opacity: 0.5; cursor: not-allowed; }

      .field {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 12px;
        padding: 12px 14px;
        border-radius: 12px;
        background: rgba(255, 255, 255, 0.03);
        border: 1px solid rgba(255, 255, 255, 0.08);
        font-size: 14px;
      }
      select, input[type="checkbox"] { accent-color: white; }
      select {
        width: 120px;
        background: #0f0f11;
        color: var(--text);
        border: 1px solid var(--border);
        border-radius: 10px;
        height: 36px;
        padding: 0 10px;
      }
      .hint { color: var(--muted); font-size: 12px; line-height: 1.6; }
    </style>
  </head>
  <body>
    <main class="shell">
      <!-- 头部 -->
      <div class="header">
        <div>
          <div class="title">Hanzi Writer</div>
          <div class="subtitle">ui://app/hanzi-writer.html</div>
        </div>
        <div class="subtitle" id="status-text">准备中...</div>
      </div>

      <section class="content">
        <!-- 左侧：预览区 + 练习区 -->
        <div class="card">
          <!-- 汉字输入 -->
          <div class="char-input-row">
            <input type="text" id="char-input" value="字" maxlength="1" placeholder="输入汉字" />
            <button class="btn" id="load-char-btn">加载</button>
          </div>

          <!-- 笔顺预览区 -->
          <div class="character-area">
            <div id="character-target">
              <div class="loading" id="char-loading">加载中...</div>
            </div>
          </div>

          <!-- 元信息 -->
          <div class="meta">
            <span id="stroke-count">笔画数：--</span>
            <span id="mode-text">模式：预览</span>
          </div>

          <!-- 书写练习区 -->
          <div class="quiz-area">
            <div id="quiz-target"></div>
            <div class="actions">
              <button class="btn" id="start-quiz-btn">开始练习</button>
              <button class="btn" id="stop-quiz-btn" disabled>停止练习</button>
              <button class="btn" id="hint-btn">显示提示</button>
            </div>
          </div>
        </div>

        <!-- 右侧：控制面板 -->
        <aside class="card controls">
          <button class="btn" id="animate-btn">播放笔顺动画</button>
          <button class="btn" id="loop-btn">循环播放</button>

          <label class="field">
            <span>显示轮廓</span>
            <input type="checkbox" id="show-outline" checked />
          </label>

          <label class="field">
            <span>显示汉字</span>
            <input type="checkbox" id="show-character" checked />
          </label>

          <label class="field">
            <span>动画速度</span>
            <select id="speed">
              <option value="1">x1</option>
              <option value="2" selected>x2</option>
              <option value="5">x5</option>
            </select>
          </label>

          <label class="field">
            <span>练习模式</span>
            <select id="quiz-mode">
              <option value="outline">显示轮廓</option>
              <option value="no-outline">无轮廓（难）</option>
            </select>
          </label>

          <div class="hint">
            在左侧输入汉字并点击加载，然后可以播放笔顺动画或开始书写练习。
          </div>
        </aside>
      </section>
    </main>

    <script>
      // ============================================================
      //  全局状态变量
      // ============================================================
      let charWriter = null;      // 预览区 HanziWriter 实例
      let quizWriter = null;      // 练习区 HanziWriter 实例
      let isLooping = false;      // 是否正在循环播放
      let quizActive = false;     // 是否正在进行练习
      let currentChar = "字";     // 当前汉字
      let totalStrokes = 0;       // 当前汉字总笔画数
      let charDataCache = {};     // 汉字数据缓存

      // DOM 元素引用（静态元素，在页面生命周期内不变）
      const statusText = document.getElementById("status-text");
      const strokeCount = document.getElementById("stroke-count");
      const modeText = document.getElementById("mode-text");

      // ============================================================
      //  工具函数
      // ============================================================

      /** 更新状态栏文字 */
      function setStatus(text) {
        statusText.textContent = text;
      }

      /**
       * 动态获取 loading 元素
       * 注意：不能缓存此引用，因为清空 innerHTML 后元素会被重建
       */
      function getLoadingEl() {
        return document.getElementById("char-loading");
      }

      /** 获取当前控件配置（速度、显示选项） */
      function getOptions() {
        const speed = parseFloat(document.getElementById("speed").value);
        const showOutline = document.getElementById("show-outline").checked;
        const showCharacter = document.getElementById("show-character").checked;
        return { speed, showOutline, showCharacter };
      }

      // ============================================================
      //  汉字数据加载层
      // ============================================================

      /**
       * 加载指定汉字的笔画数据
       * 优先从缓存读取，否则从 CDN 加载
       * @param {string} char - 单个汉字
       * @returns {Promise<Object>} 汉字笔画数据对象
       */
      function loadCharData(char) {
        // 缓存命中：直接返回
        if (charDataCache[char]) {
          return Promise.resolve(charDataCache[char]);
        }

        // 从 CDN 加载
        return fetch(
          "https://cdn.jsdelivr.net/npm/hanzi-writer-data@2.0/" +
            encodeURIComponent(char) +
            ".json"
        )
          .then(function (res) {
            if (!res.ok) throw new Error("Character not found");
            return res.json();
          })
          .then(function (data) {
            charDataCache[char] = data; // 写入缓存
            return data;
          });
      }

      // ============================================================
      //  预览区初始化
      // ============================================================

      /**
       * 初始化或更新预览区的 HanziWriter 实例
       * 首次调用时创建实例，后续调用使用 setCharacter 切换汉字
       */
      function initCharWriter(char) {
        const loadingEl = getLoadingEl();
        if (loadingEl) loadingEl.style.display = "grid";
        setStatus("加载中...");

        const opts = getOptions();

        loadCharData(char)
          .then(function (data) {
            // 更新笔画数
            totalStrokes = data.strokes.length;
            strokeCount.textContent = "笔画数：" + totalStrokes;

            if (charWriter) {
              // 已有实例：切换汉字（推荐方式，避免 SVG 重叠）
              charWriter.setCharacter(char);
            } else {
              // 首次：创建实例
              charWriter = HanziWriter.create("character-target", char, {
                width: 260,
                height: 260,
                padding: 10,
                showOutline: opts.showOutline,
                showCharacter: opts.showCharacter,
                strokeAnimationSpeed: opts.speed,
                delayBetweenStrokes: 200,
                strokeColor: "#f4f4f5",
                outlineColor: "#3f3f46",
                radicalColor: "#60a5fa",
                // 使用自定义数据加载器（带缓存）
                charDataLoader: function (c, onComplete) {
                  loadCharData(c).then(onComplete);
                },
              });
            }

            // 隐藏 loading
            const loadingEl2 = getLoadingEl();
            if (loadingEl2) loadingEl2.style.display = "none";
            setStatus("已加载：" + char);
          })
          .catch(function () {
            const loadingEl3 = getLoadingEl();
            if (loadingEl3) {
              loadingEl3.textContent = "加载失败，请换一个汉字";
            }
            strokeCount.textContent = "笔画数：--";
            setStatus("加载失败：" + char);
          });
      }

      // ============================================================
      //  练习区初始化
      // ============================================================

      /**
       * 初始化或更新练习区的 HanziWriter 实例
       */
      function initQuizWriter(char) {
        const quizTarget = document.getElementById("quiz-target");

        if (quizWriter) {
          // 已有实例：先取消正在进行的 quiz，再切换汉字
          quizWriter.cancelQuiz();
          quizWriter.setCharacter(char);
          return;
        }

        const quizMode = document.getElementById("quiz-mode").value;
        const showOutline = quizMode === "outline";

        // 首次创建
        quizWriter = HanziWriter.create("quiz-target", char, {
          width: 280,
          height: 280,
          padding: 10,
          showCharacter: false,          // 练习区默认不显示汉字本体
          showOutline: showOutline,
          strokeColor: "#f4f4f5",
          outlineColor: "#3f3f46",
          highlightColor: "#60a5fa",     // 提示高亮颜色
          drawingColor: "#f4f4f5",        // 用户书写笔迹颜色
          showHintAfterMisses: 3,          // 错 3 次后显示提示
          highlightOnComplete: true,        // 完成后闪烁提示
          charDataLoader: function (c, onComplete) {
            loadCharData(c).then(onComplete);
          },
          onLoadCharDataError: function () {
            quizTarget.innerHTML = '<div class="loading">无法加载练习数据</div>';
          },
        });
      }

      // ============================================================
      //  汉字切换主函数
      // ============================================================

      /**
       * 加载新汉字：重置状态、更新两个区域
       */
      function loadCharacter(char) {
        if (!char || char.length === 0) return;

        currentChar = char;
        modeText.textContent = "模式：预览";
        quizActive = false;
        isLooping = false;

        // 重置按钮状态
        document.getElementById("start-quiz-btn").disabled = false;
        document.getElementById("stop-quiz-btn").disabled = true;

        // 同时初始化预览区和练习区
        initCharWriter(char);
        initQuizWriter(char);
      }

      // ============================================================
      //  练习模式控制
      // ============================================================

      /** 开始书写练习 */
      function startQuiz() {
        if (!quizWriter) initQuizWriter(currentChar);
        if (quizWriter) {
          quizActive = true;
          modeText.textContent = "模式：练习中";
          document.getElementById("start-quiz-btn").disabled = true;
          document.getElementById("stop-quiz-btn").disabled = false;
          setStatus("开始练习：" + currentChar);

          // 调用 quiz() 并传入三个事件回调
          quizWriter.quiz({
            // 笔画书写正确
            onCorrectStroke: function (strokeData) {
              setStatus(
                "正确！第 " +
                  (strokeData.strokeNum + 1) +
                  " / " +
                  totalStrokes +
                  " 笔"
              );
            },
            // 笔画书写错误
            onMistake: function (strokeData) {
              setStatus(
                "错误：第 " +
                  (strokeData.strokeNum + 1) +
                  " 笔（已错 " +
                  strokeData.mistakesOnStroke +
                  " 次）"
              );
            },
            // 全部笔画完成
            onComplete: function (summaryData) {
              setStatus(
                "完成！共 " +
                  totalStrokes +
                  " 笔，错误 " +
                  summaryData.totalMistakes +
                  " 次"
              );
              quizActive = false;
              modeText.textContent = "模式：已完成";
              document.getElementById("start-quiz-btn").disabled = false;
              document.getElementById("stop-quiz-btn").disabled = true;
            },
          });
        }
      }

      /** 停止书写练习 */
      function stopQuiz() {
        if (quizWriter && quizActive) {
          quizWriter.cancelQuiz();
          quizActive = false;
          modeText.textContent = "模式：预览";
          document.getElementById("start-quiz-btn").disabled = false;
          document.getElementById("stop-quiz-btn").disabled = true;
          setStatus("练习已停止");
        }
      }

      // ============================================================
      //  事件绑定
      // ============================================================

      // 加载按钮
      document.getElementById("load-char-btn").addEventListener("click", function () {
        const input = document.getElementById("char-input").value.trim();
        if (input) {
          stopQuiz();
          loadCharacter(input.charAt(0));
        }
      });

      // 输入框回车
      document.getElementById("char-input").addEventListener("keydown", function (e) {
        if (e.key === "Enter") {
          document.getElementById("load-char-btn").click();
        }
      });

      // 播放笔顺动画（单次）
      document.getElementById("animate-btn").addEventListener("click", function () {
        if (charWriter) {
          isLooping = false;
          setStatus("播放动画：" + currentChar);
          charWriter.animateCharacter();
        }
      });

      // 循环播放 / 停止循环
      document.getElementById("loop-btn").addEventListener("click", function () {
        if (charWriter) {
          if (isLooping) {
            charWriter.stopAnimation();
            isLooping = false;
            setStatus("循环已停止");
          } else {
            isLooping = true;
            setStatus("循环播放：" + currentChar);
            charWriter.loopCharacterAnimation();
          }
        }
      });

      // 开始/停止练习
      document.getElementById("start-quiz-btn").addEventListener("click", startQuiz);
      document.getElementById("stop-quiz-btn").addEventListener("click", stopQuiz);

      // 显示提示
      document.getElementById("hint-btn").addEventListener("click", function () {
        if (quizWriter && quizActive) {
          setStatus("提示：观察高亮的笔画");
        } else if (charWriter) {
          charWriter.animateCharacter();
          setStatus("提示：观看笔顺动画");
        }
      });

      // 显示/隐藏轮廓（预览区）
      document.getElementById("show-outline").addEventListener("change", function () {
        if (charWriter) {
          if (this.checked) charWriter.showOutline();
          else charWriter.hideOutline();
        }
      });

      // 显示/隐藏汉字本体（预览区）
      document.getElementById("show-character").addEventListener("change", function () {
        if (charWriter) {
          if (this.checked) charWriter.showCharacter();
          else charWriter.hideCharacter();
        }
      });

      // 切换练习模式（轮廓显示）
      document.getElementById("quiz-mode").addEventListener("change", function () {
        if (quizWriter) {
          stopQuiz();
          const showOutline = this.value === "outline";
          if (showOutline) quizWriter.showOutline();
          else quizWriter.hideOutline();
        }
      });

      // ============================================================
      //  启动：加载默认汉字
      // ============================================================
      loadCharacter(currentChar);
    </script>
  </body>
</html>
```

---

## 5. 功能模块详解

### 5.1 笔顺动画区（预览区）

#### 功能说明
在页面左上方显示汉字，支持播放笔顺动画和循环播放。用户可以通过选项控制是否显示汉字本体、轮廓线，以及动画播放速度。

#### 核心 API 调用

```javascript
// 创建实例
charWriter = HanziWriter.create("character-target", char, {
  width: 260,                       // 画布宽度（px）
  height: 260,                      // 画布高度（px）
  padding: 10,                       // 内边距
  showOutline: true,                 // 是否显示汉字轮廓
  showCharacter: true,               // 是否显示汉字本体
  strokeAnimationSpeed: 2,           // 笔画动画速度倍数
  delayBetweenStrokes: 200,          // 笔画间延迟（ms）
  strokeColor: "#f4f4f5",            // 笔画颜色
  outlineColor: "#3f3f46",           // 轮廓颜色
  radicalColor: "#60a5fa",           // 偏旁部首颜色
  charDataLoader: function(c, cb) {  // 自定义数据加载器
    loadCharData(c).then(cb);
  },
});

// 播放一次笔顺动画
charWriter.animateCharacter();

// 循环播放笔顺动画
charWriter.loopCharacterAnimation();

// 停止动画
charWriter.stopAnimation();

// 切换显示选项
charWriter.showOutline();
charWriter.hideOutline();
charWriter.showCharacter();
charWriter.hideCharacter();

// 切换汉字（推荐方式，避免重建实例）
charWriter.setCharacter("新汉字");
```

#### 参数配置说明

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| `width` | number | - | 画布宽度（像素） |
| `height` | number | - | 画布高度（像素） |
| `padding` | number | 0 | 汉字与画布边缘的距离 |
| `showOutline` | boolean | true | 是否显示灰色轮廓 |
| `showCharacter` | boolean | true | 是否显示完整汉字 |
| `strokeAnimationSpeed` | number | 1 | 笔画动画速度倍数，1 为正常速度 |
| `delayBetweenStrokes` | number | 300 | 笔画之间的间隔时间（毫秒） |
| `strokeColor` | string | '#555' | 笔画颜色 |
| `outlineColor` | string | '#DDD' | 轮廓颜色 |
| `radicalColor` | string | null | 偏旁部首颜色，null 表示与笔画同色 |
| `charDataLoader` | function | CDN 加载 | 自定义汉字数据加载器 |

### 5.2 书写练习区

#### 功能说明
在页面左下方提供手写画布，用户用鼠标或触屏按正确笔顺书写汉字。系统实时检测每一笔的正确性，错误 3 次后自动高亮显示提示笔画。全部写完后显示统计结果。

#### 核心 API 调用

```javascript
// 创建练习区实例
quizWriter = HanziWriter.create("quiz-target", char, {
  width: 280,
  height: 280,
  showCharacter: false,              // 练习时不显示完整汉字
  showOutline: true,                  // 显示轮廓作为提示
  highlightColor: "#60a5fa",         // 提示高亮颜色
  drawingColor: "#f4f4f5",           // 用户书写的笔迹颜色
  showHintAfterMisses: 3,             // 错几次后显示提示
  highlightOnComplete: true,           // 完成后是否闪烁
});

// 开始练习（传入事件回调）
quizWriter.quiz({
  onCorrectStroke: function(strokeData) {
    console.log("正确笔画:", strokeData.strokeNum);
    console.log("本笔错误次数:", strokeData.mistakesOnStroke);
    console.log("总错误次数:", strokeData.totalMistakes);
    console.log("剩余笔画数:", strokeData.strokesRemaining);
  },
  onMistake: function(strokeData) {
    console.log("写错了:", strokeData.strokeNum);
  },
  onComplete: function(summaryData) {
    console.log("完成！汉字:", summaryData.character);
    console.log("总错误次数:", summaryData.totalMistakes);
  },
});

// 取消练习
quizWriter.cancelQuiz();

// 切换汉字
quizWriter.setCharacter("新汉字");
```

#### 回调事件数据结构

**`onCorrectStroke` 和 `onMistake` 接收的 strokeData：**

```javascript
{
  strokeNum: 0,           // 当前笔画序号（从 0 开始）
  mistakesOnStroke: 1,    // 本笔画已错次数
  totalMistakes: 3,       // 整个练习累计错误次数
  strokesRemaining: 5,    // 还剩多少笔画没写
}
```

**`onComplete` 接收的 summaryData：**

```javascript
{
  character: "字",        // 完成的汉字
  totalMistakes: 2,       // 总错误次数
}
```

---

## 6. 汉字数据加载流程

### 6.1 数据来源

汉字笔画数据来自开源项目 **hanzi-writer-data**，托管在 jsDelivr CDN 上。每个汉字对应一个 JSON 文件，包含该汉字的所有笔画 SVG 路径数据。

数据 URL 格式：
```
https://cdn.jsdelivr.net/npm/hanzi-writer-data@2.0/{汉字}.json
```

例如"字"字的数据地址：
```
https://cdn.jsdelivr.net/npm/hanzi-writer-data@2.0/%E5%AD%97.json
```

### 6.2 数据流图

```
用户触发 loadCharacter("字")
        │
        ▼
  initCharWriter("字")          initQuizWriter("字")
        │                               │
        └───────────────┬───────────────┘
                        ▼
               loadCharData("字")
                        │
         ┌──────────────┴──────────────┐
         ▼                             ▼
  charDataCache["字"]?         fetch(CDN URL)
    存在？                          │
    │  └─ 是 ──► 返回缓存          ├─► res.json()
    │                               ├─► charDataCache["字"] = data
    └─ 否 ──────────────────────────┘
                        │
                        ▼
              Promise.resolve(data)
                        │
         ┌──────────────┴──────────────┐
         ▼                             ▼
  HanziWriter 内部使用         HanziWriter 内部使用
  渲染 SVG 笔画路径           渲染 SVG 笔画路径
```

### 6.3 缓存策略

实现了一个简单的内存缓存 `charDataCache`，键为汉字字符，值为数据对象。

**优点：**
- 避免重复的网络请求（如切换到之前看过的汉字）
- 预览区和练习区使用同一份数据，只加载一次
- 加快切换速度

**局限性：**
- 仅存在于内存中，刷新页面后清空
- 如果需要离线使用，建议配合 Service Worker 或 localStorage 做持久化缓存

---

## 7. 用户交互事件处理机制

### 7.1 事件绑定总览

| 触发元素 | 事件 | 处理函数 | 功能 |
|---------|------|---------|------|
| `#load-char-btn` | click | 匿名函数 → `loadCharacter()` | 加载输入框中的汉字 |
| `#char-input` | keydown | 匿名函数 → 触发按钮 click | 回车快捷加载 |
| `#animate-btn` | click | 匿名函数 → `animateCharacter()` | 播放单次笔顺动画 |
| `#loop-btn` | click | 匿名函数 → 切换循环状态 | 循环播放 / 停止循环 |
| `#start-quiz-btn` | click | `startQuiz()` | 开始书写练习 |
| `#stop-quiz-btn` | click | `stopQuiz()` | 停止书写练习 |
| `#hint-btn` | click | 匿名函数 | 显示提示（练习中高亮/预览中播放动画） |
| `#show-outline` | change | 匿名函数 | 切换预览区轮廓显示 |
| `#show-character` | change | 匿名函数 | 切换预览区汉字显示 |
| `#quiz-mode` | change | 匿名函数 | 切换练习区轮廓显示 |
| `quizWriter` 内部 | 用户手写 | HanziWriter 内部处理 | 检测笔画正确性，触发回调 |

### 7.2 关键交互流程

#### 流程一：切换汉字

```
用户在输入框输入"学" → 点击"加载"
        │
        ▼
  stopQuiz() ──► 停止正在进行的练习（如果有）
        │
        ▼
  loadCharacter("学")
        │
        ├─► 更新状态变量（currentChar, quizActive=false 等）
        ├─► 更新 UI（按钮禁用状态、模式文字）
        ├─► initCharWriter("学") ──► 预览区加载/切换汉字
        └─► initQuizWriter("学") ──► 练习区加载/切换汉字
```

#### 流程二：书写练习

```
用户点击"开始练习"
        │
        ▼
  startQuiz()
        │
        ├─► quizWriter.quiz() ──► 进入笔画检测模式
        │       │
        │       ├─► 用户写对 ──► onCorrectStroke ──► 更新状态文字
        │       ├─► 用户写错 ──► onMistake ──► 更新状态文字（累计错 3 次高亮提示）
        │       └─► 全部写完 ──► onComplete ──► 显示统计，恢复按钮
        │
        └─► 更新 UI（按钮禁用状态、模式文字）
```

---

## 8. 状态管理逻辑

### 8.1 状态变量

| 变量 | 类型 | 初始值 | 说明 |
|-----|------|--------|------|
| `charWriter` | Object \| null | `null` | 预览区 HanziWriter 实例 |
| `quizWriter` | Object \| null | `null` | 练习区 HanziWriter 实例 |
| `isLooping` | boolean | `false` | 是否正在循环播放动画 |
| `quizActive` | boolean | `false` | 是否正在进行书写练习 |
| `currentChar` | string | `"字"` | 当前显示的汉字 |
| `totalStrokes` | number | `0` | 当前汉字的总笔画数 |
| `charDataCache` | Object | `{}` | 已加载汉字数据的缓存 |

### 8.2 状态转换图

```
                    加载汉字
  ┌───────────────────────────────────────────────┐
  │                                               ▼
  │    ┌──────────┐     播放动画      ┌──────────────┐
  │    │  预览态  │ ───────────────► │  动画播放态   │
  │    │ (默认)   │ ◄─────────────── │              │
  │    └────┬─────┘     动画结束      └──────┬───────┘
  │         │                                  │
  │         │ 开始练习                          │ 开始练习
  │         ▼                                  ▼
  │    ┌──────────┐                      ┌──────────────┐
  │    │  练习态  │ ◄──────────────────── │ 练习+动画态   │
  │    │          │ ───────────────►     │              │
  │    └────┬─────┘  完成/取消 练习      └──────────────┘
  │         │
  │         │ 全部写完
  │         ▼
  │    ┌──────────┐
  │    │  完成态  │ ──► 切换汉字或重新开始 ──► 预览态
  │    └──────────┘
  │
  └───────────────────────────────────────────────────┘
```

### 8.3 状态一致性保障

**关键原则：切换汉字前必须重置所有进行中的操作。**

```javascript
// 在 loadCharacter 中：
function loadCharacter(char) {
  // ...
  quizActive = false;    // 重置练习状态
  isLooping = false;     // 重置循环状态

  stopQuiz();            // 停止正在进行的练习
  // ...
}
```

如果不这样做，可能出现以下问题：
- 练习区还在检测上一个汉字的笔画
- 循环播放动画还在运行，与新汉字冲突
- 按钮状态与实际状态不匹配

---

## 9. 从零实现教程

### 9.1 环境配置

#### 前置条件

| 工具 | 最低版本 | 检查命令 |
|-----|---------|---------|
| Node.js | 20.0.0 | `node --version` |
| npm | 9.0.0 | `npm --version` |

#### 克隆/初始化项目

```bash
# 如果已有项目，直接进入目录
cd f:\hanzi-write

# 安装根项目依赖（xmcp）
npm install

# 安装 Web 项目依赖（Next.js）
cd web
npm install
```

#### 启动开发服务器

```bash
# 方式一：启动 xmcp 服务（MCP 工具模式）
cd f:\hanzi-write
npm run dev

# 方式二：启动 Next.js 服务（Web 页面模式）
cd f:\hanzi-write\web
npm run dev
# 访问 http://localhost:3000
```

### 9.2 库文件引入方法

Hanzi Writer 库有两种引入方式：

#### 方式 A：CDN 引入（推荐，用于独立 HTML 页面）

```html
<!-- 在 <head> 中加入 -->
<script src="https://cdn.jsdelivr.net/npm/hanzi-writer@3.5/dist/hanzi-writer.min.js"></script>

<!-- 加载后，全局对象 window.HanziWriter 可用 -->
<script>
  console.log(window.HanziWriter); // 可用
  const writer = HanziWriter.create("target", "字", { width: 200, height: 200 });
</script>
```

#### 方式 B：npm 引入（用于 React/Vue 等打包项目）

```bash
npm install hanzi-writer
```

```javascript
// 在 JS/TS 文件中
import HanziWriter from "hanzi-writer";

// 或者 CommonJS
const HanziWriter = require("hanzi-writer");
```

#### 动态加载（用于 xmcp 工具的 React 组件）

```javascript
// 动态创建 <script> 标签并等待加载完成
function loadHanziWriter() {
  if (window.HanziWriter) return Promise.resolve();

  return new Promise((resolve, reject) => {
    const script = document.createElement("script");
    script.src = "https://cdn.jsdelivr.net/npm/hanzi-writer@3.7/dist/hanzi-writer.min.js";
    script.async = true;
    script.onload = () => resolve();
    script.onerror = () => reject(new Error("加载失败"));
    document.head.appendChild(script);
  });
}

// 使用
await loadHanziWriter();
const writer = window.HanziWriter.create("target", "字", { ... });
```

### 9.3 核心功能实现步骤

#### 步骤 1：创建 HTML 骨架

```html
<!doctype html>
<html>
  <head>
    <title>汉字书写</title>
    <script src="https://cdn.jsdelivr.net/npm/hanzi-writer@3.5/dist/hanzi-writer.min.js"></script>
  </head>
  <body>
    <!-- 预览区容器 -->
    <div id="character-target"></div>

    <!-- 练习区容器 -->
    <div id="quiz-target"></div>

    <!-- 控制按钮 -->
    <button id="animate-btn">播放动画</button>
    <button id="quiz-btn">开始练习</button>

    <script>
      // 步骤 2-4 的代码写在这里
    </script>
  </body>
</html>
```

#### 步骤 2：初始化预览区实例

```javascript
// 在 <script> 中
const charWriter = HanziWriter.create("character-target", "字", {
  width: 260,
  height: 260,
  showOutline: true,
  showCharacter: true,
  strokeAnimationSpeed: 2,
  strokeColor: "#333",
  radicalColor: "#2563eb",
});

// 绑定动画播放按钮
document.getElementById("animate-btn").addEventListener("click", () => {
  charWriter.animateCharacter();
});
```

#### 步骤 3：初始化练习区实例

```javascript
const quizWriter = HanziWriter.create("quiz-target", "字", {
  width: 280,
  height: 280,
  showCharacter: false,    // 练习时不显示汉字
  showOutline: true,        // 显示轮廓提示
  showHintAfterMisses: 3,   // 错 3 次显示提示
});

// 绑定练习开始按钮
document.getElementById("quiz-btn").addEventListener("click", () => {
  quizWriter.quiz({
    onCorrectStroke: (data) => {
      console.log("正确！第", data.strokeNum + 1, "笔");
    },
    onMistake: (data) => {
      console.log("写错了");
    },
    onComplete: (data) => {
      console.log("完成！共错", data.totalMistakes, "次");
    },
  });
});
```

#### 步骤 4：添加汉字切换功能

```javascript
// 添加输入框和按钮到 HTML
// <input id="char-input" value="字" />
// <button id="load-btn">切换</button>

document.getElementById("load-btn").addEventListener("click", () => {
  const char = document.getElementById("char-input").value.trim();
  if (!char) return;

  // 关键：使用 setCharacter 切换，而不是重新 create
  charWriter.setCharacter(char);
  quizWriter.cancelQuiz();  // 先停止练习
  quizWriter.setCharacter(char);
});
```

### 9.4 调试技巧

#### 技巧 1：检查 HanziWriter 是否正确加载

```javascript
// 在浏览器控制台输入
console.log(window.HanziWriter);
// 如果输出 undefined，说明 CDN 脚本未加载成功
// 检查网络面板（F12 → Network），看 hanzi-writer.min.js 是否 200
```

#### 技巧 2：查看汉字数据是否加载成功

```javascript
// 手动测试加载某个汉字
fetch("https://cdn.jsdelivr.net/npm/hanzi-writer-data@2.0/字.json")
  .then(r => r.json())
  .then(data => {
    console.log("笔画数:", data.strokes.length);
    console.log("笔画路径:", data.strokes);
  })
  .catch(err => console.error("加载失败:", err));
```

#### 技巧 3：检查 DOM 结构

Hanzi Writer 会在目标容器内创建 SVG。如果看不到汉字：

1. 检查目标容器是否有尺寸（width/height > 0）
2. 用开发者工具检查容器内是否有 `<svg>` 元素
3. 检查笔画颜色是否与背景色太接近

#### 技巧 4：监听加载错误事件

```javascript
HanziWriter.create("target", "字", {
  width: 200,
  height: 200,
  onLoadCharDataSuccess: (data) => {
    console.log("加载成功，笔画数:", data.strokes.length);
  },
  onLoadCharDataError: (reason) => {
    console.error("加载失败:", reason);
    // 常见原因：生僻字无数据、网络问题、CDN 被墙
  },
});
```

#### 技巧 5：练习模式调试

如果练习时笔画总是判错：

1. 检查 `showHintAfterMisses` 是否设为较小值（如 1），观察提示笔画
2. 确认画布尺寸足够大（建议至少 200x200）
3. 检查是否正确使用了触屏/鼠标事件（Hanzi Writer 内部已处理）

---

## 10. 调试技巧与常见问题

### 10.1 常见问题与解决方案

#### Q1: 汉字一直显示"加载中..."，不显示内容

**可能原因：**
- CDN 无法访问（网络问题或被墙）
- 汉字数据文件不存在（生僻字）
- 容器没有尺寸（width/height 为 0）

**排查步骤：**
1. 打开浏览器控制台（F12），看是否有报错
2. 打开 Network 面板，检查 `hanzi-writer.min.js` 和 `{字}.json` 是否加载成功
3. 尝试访问：`https://cdn.jsdelivr.net/npm/hanzi-writer-data@2.0/字.json` 看能否打开

#### Q2: 切换汉字后，两个字的笔画重叠在一起

**原因：** 使用 `HanziWriter.create()` 反复创建新实例，而不是用 `setCharacter()`

**解决方案：**
```javascript
// ❌ 错误：每次都创建新实例
charWriter = HanziWriter.create("target", newChar, { ... });

// ✅ 正确：已有实例时切换汉字
if (charWriter) {
  charWriter.setCharacter(newChar);  // 内部会清理并重新渲染
} else {
  charWriter = HanziWriter.create("target", newChar, { ... });
}
```

#### Q3: 用 `innerHTML` 清空容器后，loading 元素的引用失效

**原因：** `innerHTML = ""` 会销毁所有子元素，缓存的 DOM 引用变成悬浮引用

**解决方案：**
```javascript
// ❌ 错误：在页面开头缓存引用
const loadingEl = document.getElementById("char-loading");
// ... 中间某代码清空了 innerHTML ...
loadingEl.style.display = "none";  // ← 报错或无效

// ✅ 正确：每次使用时动态获取
function getLoadingEl() {
  return document.getElementById("char-loading");
}
// 使用时：
const el = getLoadingEl();
if (el) el.style.display = "none";
```

#### Q4: package.json 报 `Unexpected token ''` 错误

**原因：** 文件被保存为 UTF-8 带 BOM 格式，Next.js 将 BOM 字符视为无效 JSON

**解决方案：**
```powershell
# PowerShell 中移除 BOM
$bytes = [System.IO.File]::ReadAllBytes("path\to\package.json");
if ($bytes[0] -eq 0xEF -and $bytes[1] -eq 0xBB -and $bytes[2] -eq 0xBF) {
  $bytes = $bytes[3..($bytes.Length-1)];
  [System.IO.File]::WriteAllBytes("path\to\package.json", $bytes);
}
```

#### Q5: xmcp 工具中 `import { tool } from "xmcp"` 报错

**原因：** xmcp v0.6.x 没有导出 `tool` 函数

**解决方案：** 使用 xmcp 标准工具格式：
```typescript
// ✅ 正确格式
import { z } from "zod";
import { type ToolMetadata, type InferSchema } from "xmcp";

export const schema = {
  character: z.string().default("永"),
};

export const metadata: ToolMetadata = {
  name: "hanzi-writer",
  description: "汉字笔顺动画和描红练习",
};

export default function handler(params: InferSchema<typeof schema>) {
  // 返回 React JSX
  return <div>...</div>;
}
```

#### Q6: 练习模式下无法手写

**可能原因：**
- 容器上有 `pointer-events: none` 样式
- iframe 被 sandbox 限制了指针事件
- 画布尺寸太小（建议至少 200x200）

**排查：**
1. 检查 CSS 中是否有 `pointer-events` 相关设置
2. 检查 iframe 的 `sandbox` 属性是否缺少 `allow-pointer-lock`
3. 临时调大容器尺寸测试

---

## 附录

### A. 相关资源链接

| 资源 | 链接 |
|-----|------|
| Hanzi Writer 官网 | https://hanziwriter.org/ |
| Hanzi Writer 文档 | https://hanziwriter.org/docs.html |
| Hanzi Writer GitHub | https://github.com/chanind/hanzi-writer |
| 汉字数据仓库 | https://github.com/chanind/hanzi-writer-data |
| 数据在线预览 | https://chanind.github.io/hanzi-writer-data/ |
| xmcp 框架 | https://github.com/teamxmcp/xmcp |

### B. HanziWriter API 速查表

| 方法 | 说明 |
|-----|------|
| `HanziWriter.create(el, char, opts)` | 创建新实例 |
| `.animateCharacter({ onComplete? })` | 播放一次笔顺动画 |
| `.loopCharacterAnimation()` | 循环播放笔顺动画 |
| `.stopAnimation()` | 停止正在播放的动画 |
| `.quiz({ onCorrectStroke?, onMistake?, onComplete? })` | 进入书写练习模式 |
| `.cancelQuiz()` | 取消正在进行的练习 |
| `.setCharacter(newChar)` | 切换显示的汉字 |
| `.showCharacter()` | 显示汉字本体 |
| `.hideCharacter()` | 隐藏汉字本体 |
| `.showOutline()` | 显示汉字轮廓 |
| `.hideOutline()` | 隐藏汉字轮廓 |
| `.updateColor(name, value)` | 动态更新颜色配置 |
| `HanziWriter.loadCharacterData(char)` | （静态）加载汉字原始数据 |
| `HanziWriter.getScalingTransform(w, h, pad)` | （静态）计算 SVG 缩放变换 |
