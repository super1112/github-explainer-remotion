# 动画设计规范

本文档定义 github-explainer-remotion 所有场景的统一视觉规范。

## 目录
- [色彩系统](#色彩系统)
- [字体系统](#字体系统)
- [间距系统](#间距系统)
- [动画时序规范](#动画时序规范)
- [组件样式片段](#组件样式片段)

---

## 色彩系统

| 变量名 | 色值 | 用途 |
|--------|------|------|
| `BG_PRIMARY` | `#0d1117` | 主背景（GitHub 深色主题） |
| `BG_SECONDARY` | `#161b22` | 卡片/容器背景 |
| `BG_TERTIARY` | `#21262d` | 次级容器、分隔线 |
| `ACCENT_BLUE` | `#58a6ff` | 主色调、标题、高亮 |
| `ACCENT_CYAN` | `#39d353` | 成功/绿色强调 |
| `ACCENT_ORANGE` | `#f78166` | 警告/橙色强调 |
| `ACCENT_PURPLE` | `#bc8cff` | 次要强调 |
| `TEXT_PRIMARY` | `#e6edf3` | 主要文字 |
| `TEXT_SECONDARY` | `#8b949e` | 次要文字、说明 |
| `TEXT_MUTED` | `#484f58` | 低重要性文字 |
| `BORDER` | `#30363d` | 边框、分隔线 |

**使用规则：**
- 背景永远是 `BG_PRIMARY`（`#0d1117`）
- 标题、数据、高亮用 `ACCENT_BLUE`
- 普通正文用 `TEXT_PRIMARY`
- 说明/标签用 `TEXT_SECONDARY`
- 卡片容器用 `BG_SECONDARY` + `BORDER` 边框

```ts
// 在组件顶部定义颜色常量（不要内联硬编码）
const COLORS = {
  bgPrimary: "#0d1117",
  bgSecondary: "#161b22",
  bgTertiary: "#21262d",
  accentBlue: "#58a6ff",
  accentCyan: "#39d353",
  accentOrange: "#f78166",
  accentPurple: "#bc8cff",
  textPrimary: "#e6edf3",
  textSecondary: "#8b949e",
  border: "#30363d",
} as const;
```

---

## 字体系统

Remotion 使用系统字体，无需引入额外字体文件。

| 层级 | fontSize | fontWeight | color | 用途 |
|------|----------|------------|-------|------|
| 超大标题 | 80px | 700 | `ACCENT_BLUE` | 封面项目名 |
| 大标题 | 52px | 700 | `TEXT_PRIMARY` | 场景主标题 |
| 中标题 | 36px | 600 | `TEXT_PRIMARY` | 卡片标题 |
| 正文大 | 28px | 400 | `TEXT_PRIMARY` | 主要说明文字 |
| 正文小 | 22px | 400 | `TEXT_SECONDARY` | 次要说明 |
| 标签 | 18px | 500 | `TEXT_SECONDARY` | 角标、标签 |
| 代码 | 20px | 400 | `ACCENT_CYAN` | 代码片段 |

```ts
const FONTS = {
  superTitle: { fontSize: 80, fontWeight: 700 },
  title: { fontSize: 52, fontWeight: 700 },
  subtitle: { fontSize: 36, fontWeight: 600 },
  bodyLarge: { fontSize: 28, fontWeight: 400 },
  bodySmall: { fontSize: 22, fontWeight: 400 },
  label: { fontSize: 18, fontWeight: 500 },
  code: { fontSize: 20, fontWeight: 400, fontFamily: "monospace" },
} as const;
```

---

## 间距系统

基础单位 8px，使用 8 的倍数。

| 级别 | 大小 | 用途 |
|------|------|------|
| xs | 8px | 文字间距、图标与文字 |
| sm | 16px | 标签内边距 |
| md | 24px | 卡片内边距 |
| lg | 40px | 区块间距 |
| xl | 64px | 场景边距 |

```ts
const SPACING = {
  xs: 8,
  sm: 16,
  md: 24,
  lg: 40,
  xl: 64,
} as const;
```

**容器规范（720×1280 竖版）：**
```ts
// 内容区域（去掉两侧 64px 边距）
const CONTENT_WIDTH = 720 - 64 * 2;  // = 592px
const CONTENT_PADDING = 64;
```

---

## 动画时序规范

### 场景内动画节奏

| 元素类型 | 进场方式 | 持续帧数 | 推荐配置 |
|---------|---------|---------|---------|
| 场景背景 | 淡入 | 8帧 | interpolate opacity 0→1 |
| 主标题 | spring 弹入 | 自然 | damping:12, stiffness:200 |
| 副标题 | 淡入+上移 | 20帧 | interpolate，延迟10帧 |
| 卡片列表（逐个） | 每张延迟8帧 | 20帧/张 | spring，stagger delay |
| 进度条/横线 | 从左到右展开 | 30帧 | interpolate width 0→100% |
| 角标/标签 | 缩放弹入 | 自然 | spring，延迟主标题后 |

### 逐项延迟（stagger）公式

```ts
// 列表中第 i 项的延迟
const staggerDelay = i * 8;  // 每项延迟 8 帧

const itemScale = spring({
  frame: Math.max(0, frame - staggerDelay),
  fps,
  config: { damping: 12, stiffness: 150, mass: 1 },
});
```

### 场景过渡

本 skill 使用 `<Sequence>` 硬切场景（无淡出），每个场景内部自带淡入：

```ts
// 每个场景开头的通用淡入
const sceneOpacity = interpolate(frame, [0, 8], [0, 1], {
  extrapolateRight: "clamp",
});
```

---

## 组件样式片段

### 卡片容器

```tsx
const cardStyle: React.CSSProperties = {
  backgroundColor: COLORS.bgSecondary,
  border: `1px solid ${COLORS.border}`,
  borderRadius: 12,
  padding: SPACING.md,
};
```

### 标签/角标

```tsx
const tagStyle = (color: string): React.CSSProperties => ({
  backgroundColor: `${color}22`,  // 22 = 约13% opacity
  border: `1px solid ${color}44`,
  borderRadius: 6,
  padding: "4px 10px",
  color,
  fontSize: FONTS.label.fontSize,
  fontWeight: FONTS.label.fontWeight,
});

// 使用
<span style={tagStyle(COLORS.accentBlue)}>Python</span>
<span style={tagStyle(COLORS.accentCyan)}>2.9k ⭐</span>
```

### 分隔线

```tsx
const dividerStyle: React.CSSProperties = {
  width: "100%",
  height: 1,
  backgroundColor: COLORS.border,
  margin: `${SPACING.lg}px 0`,
};
```

### fit 图标映射

```ts
const fitConfig = {
  strong: { icon: "✅", label: "强烈推荐", color: COLORS.accentCyan },
  ok: { icon: "🔶", label: "推荐", color: COLORS.accentBlue },
  limited: { icon: "⚠️", label: "有门槛", color: COLORS.accentOrange },
} as const;
```

### 架构层盒子

```tsx
const layerBoxStyle: React.CSSProperties = {
  backgroundColor: COLORS.bgTertiary,
  border: `1px solid ${COLORS.accentBlue}44`,
  borderRadius: 8,
  padding: "12px 20px",
  color: COLORS.textPrimary,
  fontSize: FONTS.bodyLarge.fontSize,
  textAlign: "center",
};

// 核心组件高亮版
const coreLayerBoxStyle: React.CSSProperties = {
  ...layerBoxStyle,
  border: `2px solid ${COLORS.accentBlue}`,
  boxShadow: `0 0 12px ${COLORS.accentBlue}44`,
  color: COLORS.accentBlue,
  fontWeight: 600,
};
```
