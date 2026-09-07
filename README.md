# 个人工作台 · Office Workbench

面向白领办公的纯工具型 Web 个人工作台。一个 HTML 文件 + localStorage，零后端、零构建、零依赖，双击即用。

## 核心特点

- **Web 优先**：标准浏览器打开即用，跨 Windows / macOS / Linux
- **纯工具**：无账号、无云服务、无订阅，数据只存在你本机浏览器
- **单文件交付**：核心程序就是一个 `workbench-desktop.html`，拷贝到哪都能跑
- **模块化 CONFIG**：所有模块定义集中在顶部 CONFIG 对象，改配置即改功能

## 6 大功能模块

| 模块 | 类型 | 能力 |
|---|---|---|
| 任务清单 | todo | 每日 / 每周 / 每月 / 自定义期限，勾选完成，完成备注 |
| AI 知识库 | note | 上传文档归档，AI 关键点提取，标签分类检索 |
| 报表助手 | note | 数据来源 / 图表类型 / 数据摘要，AI 辅助建表 |
| 日程时间 | todo | 会议 / 专注 / 提醒 / 截止，与任务清单联动 |
| 会议速记 | note | 参会人 / 决议 / 待办 / 风险，待办可同步任务 |
| 工作总结 | note | 日报 / 周报 / 月报，AI 汇总任务与产出 |

## 快速开始

1. 下载本仓库到本地
2. 双击 `workbench-desktop.html`，用任意现代浏览器打开（Chrome / Edge / Firefox / Safari）
3. 开箱即用，每个模块已内置示例数据，可自行增删改

## 数据存储说明（务必知道）

> 东西都存在你自己这台设备的这个浏览器里。换台电脑、换个浏览器、或者清了浏览记录，之前填的就没了。

- 数据键：`workbench-desktop-v1`（在 CONFIG 里可改，改 key 等于强制重置）
- 数据只在本机本浏览器，不跨设备、不跨浏览器同步
- 想备份：浏览器 F12 → Application → Local Storage 复制 JSON
- 想清空：改 `CONFIG.storageKey` 刷新即可

## 目录结构

```
个人工作台/
├── workbench-desktop.html   # 主程序（核心代码）
├── assets/
│   ├── greet-banner.jpg      # 首页 hero 横幅
│   └── avatar.jpg            # 侧栏头像
├── docs/
│   ├── design.md             # 设计思路
│   └── architecture.md       # 技术架构文档
└── README.md                 # 本文件
```

## 自定义

### 换配色
打开 `workbench-desktop.html`，找到顶部 `:root` 主题变量，改颜色值即可全站换肤。模块色 `--module-1` ~ `--module-5` 对应五个模块。

### 改模块
在 `CONFIG.modules` 数组里增删模块对象。每个模块的字段在 `fields` 数组里定义，支持 `text` / `textarea` / `select` 三种类型。详见 [docs/architecture.md](docs/architecture.md)。

### 换横幅图
替换 `assets/greet-banner.jpg`，或改 `:root` 里的 `--greet-image` 指向新路径。

## 技术栈

纯原生 HTML + CSS + JavaScript，无任何框架、无打包工具、无外部 CDN。单文件 ~97KB。

## 许可

个人自用工具，随意使用与修改。
