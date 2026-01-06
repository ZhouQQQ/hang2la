# 从夯到拉排行榜 🏆

> 一个有趣的梯度排行工具，让你轻松对任何事物进行等级分类！

🔗 **在线体验：[https://hang2la.vercel.app/](https://hang2la.vercel.app/)**

![Vue](https://img.shields.io/badge/Vue-3.x-4FC08D?style=flat-square&logo=vue.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript)
![Naive UI](https://img.shields.io/badge/Naive_UI-2.x-18A058?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=flat-square&logo=docker)

## 📖 什么是"从夯到拉"？

"从夯到拉"是网络流行词，指一种用梯度排序表达事物优劣等级的归类方式：

- **夯** 🔴 - 代表强大或优秀（最高级）
- **顶级** 🟠 - 接近顶尖水平
- **人上人** 🟡 - 优于普通水平
- **NPC** ⚪ - 中规中矩，普通水平
- **拉完了** ⚫ - 形容拉胯或低级（最低级）

适用于游戏角色强度、食物评测、动漫排行、明星颜值等多种场景！

## ✨ 功能特性

### 🎯 核心功能

- **图片上传** - 支持多图上传，拖拽上传
- **智能引导** - 三步完成排行：上传 → 选择顺序 → 拖拽排序
- **双击编辑** - 标题和层级名称均可双击自定义
- **多种排序** - 支持按上传顺序或随机顺序逐张排行

### 🎨 交互体验

- **悬浮拖拽** - 待排图片悬浮在排行榜中央，直观拖拽
- **实时预览** - 排行过程中实时显示进度
- **动画效果** - 流畅的过渡动画，提升用户体验

### 📤 导出分享

- **一键复制** - 将排行榜复制为图片到剪贴板
- **导出图片** - 下载排行榜为 PNG 图片文件

### 📱 响应式设计

- 完美适配桌面端和移动端
- 触屏友好，支持移动端拖拽操作

## 🛠️ 技术栈

| 技术                                         | 说明                            |
| -------------------------------------------- | ------------------------------- |
| [Vue 3](https://vuejs.org/)                     | 渐进式 JavaScript 框架          |
| [TypeScript](https://www.typescriptlang.org/)   | JavaScript 的超集，提供类型检查 |
| [Vite](https://vitejs.dev/)                     | 下一代前端构建工具              |
| [Naive UI](https://www.naiveui.com/)            | Vue 3 组件库                    |
| [html2canvas](https://html2canvas.hertzen.com/) | HTML 转 Canvas 截图库           |

## 🚀 快速开始

### 环境要求

- Node.js >= 20.19.0
- npm >= 8.0.0

### 安装依赖

```bash
npm install
```

### 开发模式

```bash
npm run dev
```

访问 http://localhost:5173 查看效果

### 构建生产版本

```bash
npm run build
```

构建产物将输出到 `dist` 目录

### 预览生产版本

```bash
npm run preview
```

## 🐳 Docker 部署

### 使用 Dockerfile 构建

```bash
# 构建镜像
docker build -t hang-to-la .

# 运行容器
docker run -d -p 80:80 --name hang-to-la hang-to-la
```

访问 http://localhost 即可使用

### 自定义端口

```bash
docker run -d -p 8080:80 --name hang-to-la hang-to-la
```

访问 http://localhost:8080

## 📖 使用指南

### 第一步：上传图片

1. 点击上传区域或拖拽图片到上传框
2. 支持同时上传多张图片
3. 可以预览已上传的图片，点击 × 可删除

### 第二步：选择排序方式

- **按上传顺序** - 图片按照上传的先后顺序依次出现
- **随机顺序** - 图片被打乱顺序随机出现（盲盒模式）

### 第三步：拖拽排行

1. 图片会一张张悬浮显示在排行榜中央
2. 将图片拖拽到对应的层级（夯、顶级、人上人、NPC、拉完了）
3. 完成所有图片排行后，可以复制或导出排行榜图片

### 自定义功能

- **双击标题** - 可以自定义排行榜标题
- **双击层级** - 可以自定义各层级的名称

## 📁 项目结构

```
hang-to-la/
├── public/
│   └── favicon.svg          # 网站图标
├── src/
│   ├── components/
│   │   └── HangToLaRanker.vue   # 核心排行组件
│   ├── App.vue               # 根组件
│   └── main.ts               # 入口文件
├── index.html                # HTML 模板
├── Dockerfile                # Docker 配置
├── package.json              # 项目配置
├── tsconfig.json             # TypeScript 配置
├── vite.config.ts            # Vite 配置
└── README.md                 # 项目说明
```

## 🎯 应用场景

- 🎮 **游戏角色** - 对游戏中的角色强度进行排名
- 🍔 **美食评测** - 对各种食物进行口味排行
- 📺 **动漫排行** - 对喜欢的动漫进行梯度分类
- ⭐ **明星颜值** - 对明星颜值进行主观排行
- 🎵 **音乐榜单** - 对专辑或歌曲进行评级
- 📚 **书籍推荐** - 对读过的书籍进行等级划分

## 📄 开源协议

本项目采用 [MIT](LICENSE) 开源协议

## 🙏 致谢

- 感谢 [Naive UI](https://www.naiveui.com/) 提供优秀的组件库
- 感谢 [html2canvas](https://html2canvas.hertzen.com/) 提供截图功能
- 灵感来源于网络流行的"从夯到拉"梗

---

<p align="center">
  如果觉得好玩或有用，请给个 ⭐ Star 支持一下！
</p>
