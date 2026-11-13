# Akumy Music — 项目上下文与交接文档 (Project Context & Handover)

> 本文档用于在切换 Antigravity / AI 账号、新对话会话或跨设备开发时，快速让 AI 助手完全恢复全部项目上下文与设计决策。

---

## 1. 项目概览

- **项目名称**：Akumy Music
- **定位**：Windows 平台纯净灵动毛玻璃质感、本地高解析度音乐播放器。
- **本地项目路径**：`C:\Users\Administrator\.gemini\antigravity\scratch\akumy-music`
- **GitHub 仓库**：`https://github.com/Akumy-01/akumy-music`
- **当前版本**：v1.0.0（安装包已发布到 GitHub Releases）
- **技术栈**：
  - **核心骨架**：Electron 41 + Vite 7 + electron-vite + TypeScript
  - **UI 与动效**：React 19 + Framer Motion（物理弹簧、共享布局动画）+ Lucide React 图标
  - **状态管理**：Zustand 5（持久化配置 `akumy-settings`、用户数据 `akumy-user`、播放器状态 `akumy-player`）
  - **音频管线**：Web Audio API（HTMLAudioElement + AudioContext 增益淡入淡出 + AnalyserNode 2048 点 FFT 实时频谱）
  - **音源元数据**：`music-metadata` + `node-taglib-sharp`（双重元数据与封面提取，支持 MP3/FLAC/M4A/OGG/WAV 等）
  - **离线日语注音**：`kuroshiro` + `kuroshiro-analyzer-kuromoji`（离线词典分词，日文汉字/假名 → 平文式罗马音）
  - **歌词检索**：网易云音乐 + LRCLIB + 酷狗音乐多源 API 检索与加权匹配
  - **音频标签内嵌**：`node-taglib-sharp`（MP3 USLT、FLAC Vorbis Comment、M4A ©lyr 写入）

---

## 2. 关键业务设计与避坑决策（必读）

### ① 品牌标识与命名规范
- **Logo**：已采用用户指定的官方 V6 回旋声带 Logo（位于 `src/renderer/src/assets/logo.png`、`resources/icon.png`）。
- **禁止使用 Apple / Apple Music 命名**：
  - 核心播放模式已命名为：
    1. **灵动歌词（`lyric`）**：流体封面背景 + 弹簧物理级联滚动歌词
    2. **黑胶唱片（`vinyl`）**：旋转黑胶 + 环绕音频可视化（频谱柱/波形环/粒子）
    3. **流光舞台（`aurora`）**：基于封面主色的响应式极光流体 + 卡拉 OK 逐字流光歌词
  - 窗口按钮样式命名为：**经典圆点（`mac`）** 与 **Windows 风格（`windows`）**。

### ② 专辑封面加载保障机制
- 过去曾遇到全屏切换或多组件并发请求时偶发封面不加载的问题，现有解决方案如下：
  1. **主进程协议**：`akumy://cover/<filename>` 在 `src/main/index.ts` 中直接以内存 `Buffer` 响应，带有 `resolveCoverPath` 机制（若磁盘文件丢失自动触发重新提取）；
  2. **前端 Cover 组件**：位于 `src/renderer/src/components/Cover.tsx`，移除了延迟加载，内置 `img.complete` 检测和 1 次自动重试；
  3. **无封面兜底**：任何无内嵌封面或提取失败的歌曲，必须兜底渲染回响官方彩色 Logo 图标，不可留白或显示破碎图标。

### ③ 在线歌词检索与多音源选择
- `LyricsPanel.tsx` 中内置分段音源选择器：**全部音源 (`all`) / 网易云 (`netease`) / LRCLIB (`lrclib`) / 酷狗 (`kugou`)**。
- 支持调节 `±0.5s` 时间偏移、自动/手动识别日语注音、一键嵌入音乐文件标签或导出 `.lrc` 文件。

---

## 3. 核心目录结构

```
akumy-music/
├── electron-builder.yml        # Windows 打包配置 (NSIS x64)
├── electron.vite.config.ts     # 构建配置 (主进程/预加载/渲染层)
├── resources/                  # Windows 图标 (icon.png)
├── src/
│   ├── shared/
│   │   └── types.ts            # 前后端共享类型 (Track, LyricCandidate, etc.)
│   ├── main/
│   │   ├── index.ts            # Electron 主进程、窗口、IPC、akumy:// 自定义协议
│   │   ├── library.ts          # 曲库扫描、双重提取、封面持久化、增量更新
│   │   ├── lyrics.ts           # 网易云/LRCLIB/酷狗三源检索与智能打分
│   │   ├── romaji.ts           # Kuromoji + Kuroshiro 离线日语罗马音生成
│   │   └── tags.ts             # node-taglib-sharp 歌词写入与同名 .lrc 导出
│   ├── preload/
│   │   └── index.ts            # contextBridge API 暴露 (window.akumy)
│   └── renderer/
│       ├── index.html
│       └── src/
│           ├── assets/logo.png # 官方 V6 彩色 Logo
│           ├── components/     # Cover, PlayerBar, Sidebar, Controls, TrackList, Dialogs
│           ├── lib/            # audio(WebAudio封装), color(调色板提取), lrc(解析序列化)
│           ├── nowplaying/     # NowPlaying(全屏), Modes(三大模式), Visualizer, Backgrounds, LyricsView, LyricsPanel
│           ├── store/          # player, library, settings, user, ui (Zustand)
│           ├── views/          # Library(主页/歌曲/专辑/艺人/歌单/喜欢/搜索), Settings(外观/歌词/曲库)
│           └── styles/         # global.css (毛玻璃材质系统)
└── PROJECT_CONTEXT.md          # 本文档
```

---

## 4. 常用运行与构建指令

```bash
# 安装依赖
npm install

# 启动开发环境（带热更新）
npm run dev

# 类型检查与生产编译
npm run typecheck
npm run build

# 打包 Windows 安装程序
npm run dist
```
