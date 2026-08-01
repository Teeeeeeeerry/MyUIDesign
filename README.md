# MyUI Design System
通用设计模板。

## 设计理念

- **Token-first**：所有视觉决策从 CSS 变量开始，不写裸值
- **语义化命名**：颜色按用途命名（paper/ink/accent），不按色相（green/beige/gold）
- **透明度阶固定**：每个透明度有且仅有一个用途，不随意新增
- **组件可拆卸**：每个组件独立可用，不强制依赖框架
- **隔离优先**：UI 通过 Shadow DOM 与宿主双向隔离

## 文件结构

```
MyUIDesign/
├── tokens.css        # 设计令牌 —— 颜色、字体、圆角、间距、动画
├── components.css    # 可复用 CSS 组件
├── presets.css       # 视觉风格预设系统
├── patterns.md       # 交互与架构模式（TypeScript 实现指南）
└── README.md         # 本文件
```

## 快速开始

### 1. 引入令牌

```html
<link rel="stylesheet" href="tokens.css">
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

<!-- 默认风格 -->
<div class="ds-style-default">
  <span class="ds-styled-content">Content</span>
</div>

<!-- 弱化风格 -->
<div class="ds-style-dim">
  <span class="ds-styled-content">Content</span>
</div>
```

## 色板

| Token | 值 | 用途 |
|-------|-----|------|
| `--ds-paper` | `#f5f0e6` | 底色，纸感米白 |
| `--ds-ink` | `#1f3a2e` | 主色，深绿墨色 |
| `--ds-accent` | `#b89968` | 强调色，黄铜金 |
| `--ds-surface` | `#ffffff` | 浅色卡片面 |
| `--ds-danger` | `#7a4030` | 错误态，锈红 |

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
