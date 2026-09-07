# 技术架构文档 · 个人工作台

## 1. 整体架构

```
┌─────────────────────────────────────────────────┐
│            workbench-desktop.html              │
├─────────────────────────────────────────────────┤
│  :root 主题变量（CSS 变量，全站换肤入口）        │
├─────────────────────────────────────────────────┤
│  CONFIG 对象（模块定义 / 字段定义 / 示例数据）   │
├─────────────────────────────────────────────────┤
│  ICONS 图标库（SVG path，stroke 跟随 color）     │
├─────────────────────────────────────────────────┤
│  store 数据层（localStorage 读写）              │
├─────────────────────────────────────────────────┤
│  render 渲染层（view 路由 → 模块渲染）           │
├─────────────────────────────────────────────────┤
│  交互层（新增/编辑/删除/勾选/排序）              │
└─────────────────────────────────────────────────┘
```

单文件、零依赖、无构建。所有逻辑在浏览器端运行。

## 2. CONFIG 数据结构

CONFIG 是整个应用的"配置中心"，位于 `<script>` 顶部。

### 2.1 顶层字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `storageKey` | string | localStorage 键名。改它 = 强制重置数据 |
| `owner` | string | 侧栏品牌名 / 首页问候对象 |
| `slogan` | string | 侧栏副标题 |
| `modules` | array | 模块定义数组（见 2.2） |

### 2.2 模块对象

```javascript
{
  key: "tasks",              // 唯一标识，用作 data 字段的 key
  name: "任务清单",          // 显示名
  icon: "list",              // ICONS 里的图标 key
  tint: "#efeee8",           // 模块浅底色（卡片背景）
  color: "var(--accent)",    // 模块主色（引用 CSS 变量）
  type: "todo",              // 模块类型：todo | note
  desc: "...",               // 模块描述
  priorities: [...],         // todo 型：优先级分组
  moods: [...],             // note 型：分类标签
  fields: [...],             // 字段定义
  seed: [...]                // 初始数据
}
```

### 2.3 字段定义

```javascript
{ key:"deadline", label:"截止时间", type:"text", placeholder:"..." }
{ key:"completeNote", label:"完成备注", type:"textarea", placeholder:"..." }
{ key:"fileType", label:"文件类型", type:"select", options:["Excel","Word",...] }
```

支持三种字段类型：
- `text`：单行输入
- `textarea`：多行输入
- `select`：下拉选择，`options` 数组定义选项

### 2.4 todo 型与 note 型差异

| 维度 | todo 型 | note 型 |
|---|---|---|
| 完成状态 | 有（done 布尔，可勾选） | 无 |
| 分组机制 | `priorities`（每日/每周/每月/自定义） | `moods`（分类标签数组） |
| 正文 | 无独立正文，靠 fields | 有 `content` 富文本正文 |
| 统计 | 完成率、本周完成数 | 条目数、分类分布 |
| 代表模块 | 任务清单、日程时间 | 知识库、报表、会议速记、工作总结 |

## 3. 数据存储

### 3.1 store 对象

```javascript
const store = {
  load() {
    const raw = localStorage.getItem(CONFIG.storageKey);
    if (raw) return JSON.parse(raw);
    // 首次访问：用 CONFIG.seed 初始化
    const d = {};
    CONFIG.modules.forEach(m => d[m.key] = structuredClone(m.seed || []));
    return d;
  },
  save() {
    localStorage.setItem(CONFIG.storageKey, JSON.stringify(data));
  }
};
```

### 3.2 数据结构

`data` 是一个对象，key 是模块 key，value 是该模块的记录数组：

```json
{
  "tasks":   [ {id, title, priority, done, note, ...fields}, ... ],
  "docs":    [ {id, title, content, mood, date, ...fields}, ... ],
  "reports": [ ... ],
  "schedule":[ ... ],
  "meetings":[ ... ],
  "summary": [ ... ]
}
```

### 3.3 持久化时机

任何增删改操作都调用 `persist()` → `store.save()` + `render()`。保证数据实时落盘，刷新不丢。

## 4. 主题系统

所有颜色定义在 `:root`，按角色分组：

| 角色 | 变量 | 用途 |
|---|---|---|
| 页面表面 | `--page-bg` `--surface-card` `--surface-nested` | 底色/卡片/嵌套区 |
| 边框 | `--border` `--border-input` | 常规边框/强边框 |
| 文本 | `--text` `--text-secondary` `--text-tertiary` | 主/次/弱 |
| 主色 | `--accent` `--accent-muted` `--on-accent` | 按钮/选中/主色前景 |
| 模块色 | `--module-1` ~ `--module-5` | 五色轮转，CONFIG 引用 |
| 危险色 | `--danger` `--danger-muted` | 删除/危险操作 |
| 侧栏 | `--drawer-bg` `--drawer-text` 等 | 深色侧栏面板 |
| 圆角 | `--radius-control/tile/card/sheet` | 按组件角色，勿裸写数字 |
| 图片 | `--greet-image` `--page-texture` | hero 背景/页面纹理 |

**换肤流程**：改 `:root` 变量值 → 全站生效，无需改其他代码。

## 5. 图标系统

`ICONS` 对象存放 SVG path（不含 `<svg>` 外壳）：

```javascript
const ICONS = {
  home: '<path d="M4 11.5 12 5l8 6.5"/>...',
  list: '<path d="M8.5 6h11"/>...',
  // ...
};
```

使用时套 `<svg>` 外壳，`stroke` 跟随 `color`。加图标只需在 ICONS 里加一个 key，CONFIG 引用该 key。

## 6. 渲染层

`render()` 根据 `view` 变量决定渲染什么：

- `view === "home"`：首页 dashboard（问候 + 统计卡片 + 番茄钟）
- `view === moduleKey`：对应模块的列表视图

模块渲染按 `type` 分流：
- `todo` 型：按 `priorities` 分组渲染，每条有勾选框
- `note` 型：按 `moods` 筛选，卡片网格布局

## 7. 扩展指南

### 7.1 加一个新模块

在 `CONFIG.modules` 数组末尾追加：

```javascript
{
  key: "ideas",
  name: "灵感速记",
  icon: "bulb",              // 需在 ICONS 加 "bulb" 图标
  tint: "#f0eef5",
  color: "var(--module-5)",  // 取一个未占用的模块色
  type: "note",
  desc: "随时记下灵感碎片",
  moods: ["产品","运营","生活"],
  fields: [
    { key:"source", label:"来源", type:"text", placeholder:"..." },
    { key:"action", label:"下一步", type:"textarea", placeholder:"..." }
  ],
  seed: [
    { id:71, title:"示例", content:"...", mood:"产品", date:"2026-09-01", source:"", action:"" }
  ]
}
```

刷新页面，侧栏自动出现新模块，无需改任何渲染代码。

### 7.2 加一个新字段类型

当前支持 text/textarea/select。若要加 `date` 类型：

1. 在渲染表单的 `fieldHTML()` 函数里加 `case "date":` 分支
2. 渲染 `<input type="date">`
3. 数据自动随 record 存入 localStorage

### 7.3 加一个新主题

复制 `:root` 整段，改色值，或用 `[data-theme="dark"]` 覆盖，加个切换按钮即可。

## 8. 文件清单

| 文件 | 作用 | 大小 |
|---|---|---|
| `workbench-desktop.html` | 主程序（HTML+CSS+JS 单文件） | ~97KB |
| `assets/greet-banner.jpg` | 首页 hero 横幅图 | - |
| `assets/avatar.jpg` | 侧栏头像 | - |
| `docs/design.md` | 设计思路文档 | - |
| `docs/architecture.md` | 本技术文档 | - |
| `README.md` | 使用说明 | - |

## 9. 已知边界

- 不支持多设备数据同步（localStorage 本地隔离）
- AI 字段为预留位，未接入实际 AI 接口
- 无导出功能（可通过 F12 控制台复制 localStorage JSON 手动备份）
- 单文件 ~97KB，结构在文件内用 `/* === 分区标题 === */` 注释划分
