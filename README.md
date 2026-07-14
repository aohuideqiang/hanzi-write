# 汉字笔顺 MCP 项目

基于 xmcp 框架的 MCP 工具集，包含汉字笔顺动画练习和实时天气查询。

## 功能特性

- 汉字笔顺动画播放与描红练习
- 全球主要城市实时天气查询
- React + TypeScript + Tailwind CSS v4
- 支持 HTTP 传输模式
- Docker 容器化部署
- 内置测试框架（vitest）

## 环境要求

- Node.js >= 20.0.0
- npm

## 快速开始

```bash
npm install
npm run dev
```

开发服务器启动后，MCP 服务将通过 HTTP 模式运行。

## 可用的 MCP 工具

### 1. hanzi-writer（汉字笔顺练习）

- **描述**：汉字笔顺动画和描红练习
- **参数**：
  - `character` (string, 默认: "永") - 要练习的汉字

### 2. weather（天气查询）

- **描述**：查询全球主要城市的实时天气
- **支持城市**：Buenos Aires, San Francisco, Berlin, Tokyo, New York

## 项目脚本

| 命令 | 说明 |
|---|---|
| `npm run dev` | 启动开发服务器 |
| `npm run build` | 生产环境构建 |
| `npm start` | 启动生产服务 |
| `npm run typecheck` | TypeScript 类型检查 |
| `npm test` | 启动测试（watch 模式） |
| `npm run test:ci` | 一次性运行测试 |
| `npm run clean` | 清理构建产物 |

更多命令请查看 [commands.txt](commands.txt)。

## 项目结构

```
hanzi-write/
├── src/
│   ├── lib/           # 共享逻辑
│   │   ├── constants.ts
│   │   ├── weather-client.ts
│   │   └── hanzi-writer-loader.ts
│   └── tools/         # MCP 工具
│       ├── weather.tsx
│       └── hanzi-writer.tsx
├── tests/             # 测试文件
├── web/               # Next.js 预览端
├── Dockerfile
├── commands.txt
├── structure.txt
└── package.json
```

详细结构说明请查看 [structure.txt](structure.txt)。

## 环境变量

复制 `.env.example` 为 `.env` 并按需修改：

```bash
cp .env.example .env
```

| 变量 | 说明 | 默认值 |
|---|---|---|
| `OPEN_METEO_BASE_URL` | Open-Meteo API 地址 | `https://api.open-meteo.com` |
| `HANZI_WRITER_CDN` | HanziWriter CDN 地址 | jsDelivr |

## Docker 部署

```bash
docker build -t hanzi-write-mcp .
docker run -p 3000:3000 hanzi-write-mcp
```

## Web 预览

项目包含一个 Next.js 预览页面，用于在浏览器中查看 MCP UI：

```bash
cd web
npm install
npm run dev
```

然后访问 `http://localhost:3000`。

## 技术栈

- **框架**：xmcp
- **UI**：React 19 + TypeScript
- **样式**：Tailwind CSS v4
- **测试**：Vitest
- **部署**：Docker

## 许可证

MIT
