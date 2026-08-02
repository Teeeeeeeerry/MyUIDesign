# MyUI Design System
通用设计模板。

## 设计理念

- **Token-first**：所有视觉决策从 CSS 变量开始，不写裸值
- **按颜色命名**：Token 名描述颜色本身（cream/forest/brass），不绑定语义角色（paper/ink/accent）
- **透明度阶固定**：每个透明度有且仅有一个用途，不随意新增
- **组件可拆卸**：每个组件独立可用，不强制依赖框架
- **隔离优先**：UI 通过 Shadow DOM 与宿主双向隔离

## 文件结构

```
MyUIDesign/
├── colors.css         # 配色系统 —— 色板 + 透明度阶（可独立使用）
├── typography.css     # 字体选用 —— 字体族 + 字号阶梯 + 字重/字距（可独立使用）
├── tokens.css         # 设计令牌入口 —— 圆角 + 间距 + 动画，聚合 colors + typography
├── components.css     # 可复用 CSS 组件
├── presets.css        # 视觉风格预设系统
├── patterns.md        # 交互与架构模式（TypeScript 实现指南）
└── README.md          # 本文件
```

## 快速开始

### 1. 引入令牌

```html
<!-- 一次性引入所有令牌（颜色 + 字体 + 间距 + 圆角 + 动画） -->
<link rel="stylesheet" href="tokens.css">

<!-- 或按需引入 -->
<link rel="stylesheet" href="colors.css">
<link rel="stylesheet" href="typography.css">
```

### 2. 使用组件

```html
<link rel="stylesheet" href="components.css">

<!-- 卡片 -->
<div class="ds-card">
  <div class="ds-card-label">SETTINGS</div>
  <div class="ds-content-stack">
    <div class="ds-option-row">
      <span class="ds-option-label">Enable feature</span>
      <button class="ds-toggle ds-on"></button>
    </div>
  </div>
</div>

<!-- 按钮 -->
<button class="ds-btn-primary">Save</button>
<button class="ds-btn-secondary">Cancel</button>
```

### 3. 应用预设

```html
<link rel="stylesheet" href="presets.css">

<div class="ds-style-default">
  <span class="ds-styled-content">Content</span>
</div>
```

---

## 配色系统 `colors.css`

### 主色板

| Token | 值 | 色样 | 颜色 | 典型用法 |
|-------|-----|------|------|----------|
| `--ds-cream` | `#f5f0e6` | ██ | 米白 | 页面/面板背景 |
| `--ds-forest` | `#1f3a2e` | ██ | 森林绿 | 正文、主按钮、标题 |
| `--ds-brass` | `#b89968` | ██ | 黄铜 | 边框装饰、完成态标记 |
| `--ds-white` | `#ffffff` | ██ | 纯白 | 卡片、弹窗、输入框 |
| `--ds-rust` | `#7a4030` | ██ | 锈红 | 错误提示、删除按钮 |

### 透明度阶

从 `--ds-forest` 派生，每个透明度只有一个语义用途。

| Token | 透明度 | 用途 |
|-------|--------|------|
| `--ds-forest-08` | 8% | 分隔线 |
| `--ds-forest-09` | 9% | 卡片描边 |
| `--ds-forest-15` | 15% | 次级按钮描边、开关关闭态 |
| `--ds-forest-22` | 22% | 输入框描边 |
| `--ds-forest-40` | 40% | 页脚文字、占位符 |
| `--ds-forest-55` | 55% | 次级文字、标签 |

### 命名规则

- 按**颜色本身**命名，不按语义角色 —— `--ds-rust` 而非 `--ds-danger`，`--ds-forest` 而非 `--ds-primary`
- Token 名回答"这是什么颜色？"，不回答"这个颜色用来干什么？"
- 新增颜色前先确认：这个颜色本身叫什么？无法用一个词描述 → 可能是现有 token 的透明度阶变体
- 换品牌色时：token 名跟着变（森林绿变海军蓝 → `--ds-forest` 变 `--ds-navy`），但组件 CSS 不用改

---

## 字体选用 `typography.css`

### 字体族

| Token | 字体栈 | 用途 |
|-------|--------|------|
| `--ds-font-mono` | JetBrains Mono, SF Mono, Cascadia Code, Fira Code, monospace | 标签、按钮、标题、数据、导航 |
| `--ds-font-ui` | system-ui, -apple-system, Segoe UI, Roboto, sans-serif | 正文、描述、通知、长句 |

### 选用原则

- **等宽字**：需要结构感、对齐、精确性的场景 —— 按钮文字、标签、标题、代码/数据
- **系统字**：需要阅读流畅性的场景 —— 正文段落、Toast 提示、描述文字
- **不引入 Web Font**：零外部依赖，首屏无 FOIT/FOUT，离线可用

### 字号阶梯

7 档，从标签到标题全覆盖。

| Token | 值 | 用途 |
|-------|-----|------|
| `--ds-text-2xs` | 7px | 标签、徽章、overline |
| `--ds-text-xs` | 8px | 页脚、辅助信息 |
| `--ds-text-sm` | 10px | 次级正文、placeholder |
| `--ds-text-base` | 11px | 正文、选项标签 |
| `--ds-text-md` | 13px | 强调正文、toast |
| `--ds-text-lg` | 14px | 标题、面板名 |
| `--ds-text-xl` | 16px | 大标题、Logo |

### 字重 & 字距

| Token | 值 | 用途 |
|-------|-----|------|
| `--ds-weight-normal` | 400 | 正文 |
| `--ds-weight-bold` | 700 | 标题、按钮、标签 |
| `--ds-tracking-tight` | -.02em | 大标题 |
| `--ds-tracking-normal` | 0 | 正文 |
| `--ds-tracking-wide` | .1em | 按钮 |
| `--ds-tracking-wider` | .18em | 页脚 |
| `--ds-tracking-widest` | .2em | 标签、overline |

### 排版基类

| 类 | 效果 |
|----|------|
| `.ds-text-label` | 全大写、宽字距、小字号、等宽加粗 —— 用于卡片标签、section header |
| `.ds-text-body` | 系统字、常规字重、1.5 行高 —— 用于正文段落 |
| `.ds-text-heading` | 等宽、加粗、收紧字距 —— 用于标题 |

---

## 组件清单

| 组件 | CSS 类 | 说明 |
|------|--------|------|
| Card | `.ds-card` | 卡片容器 |
| Header | `.ds-header` | Logo + 标题 + 副标题 |
| Toggle | `.ds-toggle` | 开关（含 `.ds-on` 开启态） |
| Select | `.ds-select` | 下拉选择 |
| Primary Button | `.ds-btn-primary` | 主操作按钮 |
| Secondary Button | `.ds-btn-secondary` | 次级按钮 |
| Toast | `.ds-toast` | 短暂提示（info/error） |
| FAB | `.ds-fab` | 悬浮操作球（idle/loading/done/error） |
| Inline Button | `.ds-inline-btn` | 内联悬停按钮 |
| Footer | `.ds-footer` | 页脚 |

## 预设清单

| 预设 | CSS 类 | 效果 |
|------|--------|------|
| Default | `.ds-style-default` | 左侧彩色细线 |
| Dim | `.ds-style-dim` | 悬停才显示 |
| Underline | `.ds-style-underline` | 实线下划线 |
| Bold | `.ds-style-bold` | 加粗 |
| Italic | `.ds-style-italic` | 斜体 |
| Fade | `.ds-style-fade` | 半透明弱化 |

## 交互模式

详见 [patterns.md](./patterns.md)：

1. Shadow DOM 隔离挂载
2. UI 状态机（idle → loading → done → error）
3. 悬停意图检测（延迟显示 + 延迟隐藏）
4. 拖拽移动 + 位置持久化
5. 用户 CSS 安全注入
6. Toast 通知系统
7. 修饰键 + 拖选
8. 组件生命周期管理

## 许可

MIT
