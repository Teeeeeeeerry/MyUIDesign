# UI Patterns — 交互与架构模式

从浏览器扩展实战中提炼的通用模式。每种模式独立可用，可按需组合。

---

## 1. Shadow DOM 隔离挂载

**问题：** 注入宿主页面的 UI 需要双向样式隔离 —— 宿主 CSS 不能污染我们的组件，我们的 CSS 也不能泄露到宿主。

**方案：** 每个 UI 单元挂在自己的 Shadow Root 中，用一个固定的 `data-*` 属性标记，方便页面遍历器跳过。

```ts
// 挂载
function mountIsolated(id: string): ShadowRoot {
  const host = document.createElement('div');
  host.id = `ds-host-${id}`;
  host.dataset.dsUi = '1'; // 标记：遍历器/观察者依此跳过

  // 抵御宿主页面的 all: initial / position: static !important 等规则
  host.style.cssText = 'all: initial; position: fixed; z-index: 2147483647;';

  const shadow = host.attachShadow({ mode: 'open' });

  const style = document.createElement('style');
  style.textContent = TOKENS_CSS + COMPONENTS_CSS;
  shadow.appendChild(style);

  document.body.appendChild(host);
  return shadow;
}

// 卸载
function unmountIsolated(id: string): void {
  document.getElementById(`ds-host-${id}`)?.remove();
}
```

**关键决策：**
- `all: initial` 做 CSS 属性级的兜底，防止宿主全局规则穿透
- `z-index: 2147483647`（32 位有符号整数上限）确保始终在最顶层
- `data-ds-ui="1"` 标记让内容遍历器跳过整棵子树

---

## 2. UI 状态机

**问题：** 一个 UI 控件有多个互斥状态（闲置/加载中/完成/出错），多个入口都能触发状态切换，需要保证状态一致。

**方案：** 集中管理状态，暴露唯一的 `setState()` 入口。所有触发点（按钮点击、快捷键、API 回调）都通过这个入口切换。

```ts
type UIState = 'idle' | 'loading' | 'done' | 'error';

let currentState: UIState = 'idle';

// 状态切换的唯一入口
function setUIState(s: UIState): void {
  currentState = s;

  // 找到 DOM 并同步视觉
  const el = document.querySelector('[data-ds-state]') as HTMLElement | null;
  if (el) {
    el.dataset.state = s;
    el.textContent = GLYPH_MAP[s];
  }

  // error 是瞬时态：停留 N 秒后自动回到 idle
  if (s === 'error') {
    setTimeout(() => {
      if (currentState === 'error') setUIState('idle');
    }, 3000);
  }
}
```

**关键决策：**
- `currentState` 是模块级变量，不存 DOM、不存 React state —— 避免多源分歧
- `loading` 状态下忽略重复点击（防止重复提交）
- `error` 自动回退到 `idle` —— 用户不需要手动关闭错误提示

---

## 3. 悬停意图检测

**问题：** 鼠标划过页面时，每个经过的元素都触发 `mouseover`。如果不加节制，悬停按钮会在划过的每个目标上闪一次。

**方案：** 引入"意图延迟"——指针在同一元素上停留超过阈值才浮出按钮。同时，离开后延迟隐藏，给用户时间从目标移动到按钮上。

```ts
const SHOW_DELAY = 140;  // ms —— 低于人眼把"停顿"与"路过"区分开的阈值
const HIDE_DELAY = 1500; // ms —— 覆盖用户从段落边缘移动到按钮上的全程

let showTimer: number | undefined;
let hideTimer: number | undefined;

const onMouseOver = (e: MouseEvent) => {
  const el = (e.target as Element)?.closest?.(TARGET_SELECTOR);
  if (!el) return;

  clearTimeout(hideTimer);
  if (el === currentTarget && isVisible()) return; // 同一目标，不重定位

  clearTimeout(showTimer);
  showTimer = setTimeout(() => show(el), SHOW_DELAY);
};

const onMouseOut = (e: MouseEvent) => {
  // 正移向我们的按钮？不开始隐藏倒计时
  if (isOurUI(e.relatedTarget)) return;
  // 仍在同一目标内部移动？不开始隐藏倒计时
  if (currentTarget && e.relatedTarget instanceof Node && currentTarget.contains(e.relatedTarget)) return;
  scheduleHide();
};
```

**关键决策：**
- `SHOW_DELAY = 140ms` —— 有意停留时几乎无感，划过时完全不触发
- `HIDE_DELAY = 1500ms` —— 偏长，但"按钮多停一会儿"远好于"够不着"
- `relatedTarget` 检查 —— 区分"离开目标"和"在目标内部移动"
- `visibilityState` 检查 —— 标签页在后台时跳过淡入动画，直接到位

---

## 4. 拖拽移动 + 位置持久化

**问题：** 悬浮按钮需要可拖动，且刷新后位置保持不变。

**方案：** mousedown/mousemove/mouseup 三部曲，3px 死区区分"点击"与"拖动"，拖动结束后写入 `storage.local`。

```ts
let dragging = false;
let dragged = false;
const DEAD_ZONE = 3; // px

const onMouseDown = (e: MouseEvent) => {
  dragging = true;
  dragged = false;
  startX = e.clientX;
  startY = e.clientY;
  // 记录当前位置
  const rect = el.getBoundingClientRect();
  origX = rect.left;
  origY = rect.top;
};

const onMouseMove = (e: MouseEvent) => {
  if (!dragging) return;
  const dx = e.clientX - startX;
  const dy = e.clientY - startY;
  if (Math.abs(dx) > DEAD_ZONE || Math.abs(dy) > DEAD_ZONE) {
    dragged = true;
    el.style.left = `${origX + dx}px`;
    el.style.top = `${origY + dy}px`;
  }
};

const onMouseUp = () => {
  if (dragging && dragged) {
    // 持久化
    const rect = el.getBoundingClientRect();
    storage.set({ position: { x: rect.left, y: rect.top } });
  }
  dragging = false;
};

// 点击处理器中检查 dragged，防止拖动结束时误触发 click
el.addEventListener('click', () => {
  if (dragged) return;
  handleClick();
});
```

**关键决策：**
- `mousemove`/`mouseup` 挂在 `document` 上而非元素上 —— 拖动跨越整个视口
- 3px 死区 —— 低于此距离的移动视为点击抖动
- 卸载时必须摘除 `document` 上的监听器，否则内存泄漏

---

## 5. 用户 CSS 安全注入

**问题：** 允许用户自定义样式，但不能让他们改动宿主页面或扩展自身 UI。

**方案：** 只收 CSS 声明块（属性: 值），拒绝选择器和危险构造，然后包进一个固定的作用域选择器注入。

```ts
const FORBIDDEN = [
  { pattern: /[{}]/,   msg: '只需填写 CSS 属性，无需选择器与花括号' },
  { pattern: /@import/i,      msg: '不支持 @import' },
  { pattern: /<\/?style/i,    msg: '不允许 style 标签' },
  { pattern: /javascript:/i,  msg: '不允许 javascript: 协议' },
  { pattern: /expression\s*\(/i, msg: '不允许 expression()' },
];

function validate(input: string): { ok: true } | { ok: false; msg: string } {
  for (const { pattern, msg } of FORBIDDEN) {
    if (pattern.test(input)) return { ok: false, msg };
  }
  return { ok: true };
}

function apply(input: string): void {
  const css = input.trim();
  if (!css || !validate(css).ok) return;

  const el = document.createElement('style');
  el.textContent = `.scoped-target { ${css} }`; // 包进作用域
  document.head.appendChild(el);
}
```

**关键决策：**
- 禁止选择器 —— 限制了用户能影响的范围
- 禁止 `@import` / `javascript:` / `expression()` —— 阻断 CSS 注入攻击面
- 包进 `.scoped-target` —— 声明块只作用于指定元素

---

## 6. Toast 通知系统

**问题：** 需要向用户展示短暂的状态提示（成功/失败/进度），且不干扰宿主页面。

**方案：** 复用 Shadow Root 挂载，同类型 toast 替换而非堆叠，自动超时消失。

```ts
const TOAST_DURATION = 3000;
let toastTimer: number | undefined;
let toastRoot: ShadowRoot | null = null;

function toast(msg: string, kind: 'info' | 'error' = 'info'): void {
  if (!toastRoot) toastRoot = mountIsolated('toast');

  // 移除旧 toast —— 替换而非堆叠
  toastRoot.querySelector('.ds-toast')?.remove();
  clearTimeout(toastTimer);

  const el = document.createElement('div');
  el.className = 'ds-toast';
  el.dataset.kind = kind;
  el.textContent = msg;
  toastRoot.appendChild(el);

  toastTimer = setTimeout(() => el.remove(), TOAST_DURATION);
}
```

**关键决策：**
- 复用 Shadow Root —— 只挂载一次，后续只换内容
- 替换而非堆叠 —— 多个 toast 同时出现会互相遮挡
- `data-kind` 驱动颜色 —— CSS 中用属性选择器 `[data-kind='error']`

---

## 7. 修饰键 + 拖选

**问题：** 需要在不干扰普通文本选中的前提下，提供"选中即翻译"的快捷操作。

**方案：** 全程按住修饰键（Alt/Ctrl/Meta）的选中才触发动作。mousedown 时检查修饰键，mouseup 时再次确认（防止中途松开）。

```ts
let armed = false;

const isModKey = (e: MouseEvent): boolean => e.altKey || e.ctrlKey || e.metaKey;

const onDown = (e: MouseEvent) => { armed = isModKey(e); };
const onUp = (e: MouseEvent) => {
  if (!armed) return;
  armed = false;
  if (!isModKey(e)) return; // 中途松开了修饰键
  const text = window.getSelection()?.toString().trim();
  if (text && text.length >= 2) handleSelection(text);
};

document.addEventListener('mousedown', onDown, true);
document.addEventListener('mouseup', onUp, true);
```

**关键决策：**
- 两次检查修饰键 —— mousedown 和 mouseup 都必须按住
- `>= 2` 字符阈值 —— 过滤误触
- 捕获阶段 (`true`) —— 在页面自身处理器之前拦截

---

## 8. 组件生命周期管理

**问题：** 多个 UI 组件需要统一的挂载/卸载管理，避免内存泄漏和重复挂载。

**方案：** 每个 `create*()` 返回一个清理函数。顶层持有所有清理函数，在适当时机统一调用。

```ts
// 每个组件暴露 create + cleanup
function createComponent(deps): () => void {
  // 挂载 DOM、绑定事件
  return () => {
    // 移除事件、清除计时器、卸载 DOM
  };
}

// 顶层管理
let cleanupA: (() => void) | null = null;
let cleanupB: (() => void) | null = null;

function mountAll() {
  cleanupA?.(); // 先摘旧的
  cleanupA = createComponentA();
  cleanupB = createComponentB();
}

function unmountAll() {
  cleanupA?.(); cleanupA = null;
  cleanupB?.(); cleanupB = null;
}
```

**关键决策：**
- 返回清理函数而非依赖外部 `dispose()` —— 每个组件对自己的生命周期完全负责
- 重复挂载时先调旧的清理函数 —— 防止事件监听器叠加
- 计时器必须在清理函数中 `clearTimeout` —— 否则组件卸载后回调仍触发
