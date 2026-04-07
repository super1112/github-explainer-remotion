# Remotion 核心 API 速查

## 目录
- [心智模型](#心智模型)
- [核心 Hooks](#核心-hooks)
- [interpolate — 线性映射](#interpolate)
- [spring — 弹簧动画](#spring)
- [Sequence — 时间偏移](#sequence)
- [AbsoluteFill — 全屏层](#absolutefill)
- [Audio + staticFile — 音频播放](#audio--staticfile)
- [Composition — 注册视频](#composition)
- [渲染命令](#渲染命令)
- [package.json 模板](#packagejson-模板)

---

## 心智模型

```
视频 = f(frame) → React UI
```

- `frame` 是从 0 到 `durationInFrames-1` 的整数
- 每一帧就是一次 React 渲染的截图
- 动画 = 用 `frame` 驱动 CSS 属性（opacity、transform 等）

---

## 核心 Hooks

```ts
import { useCurrentFrame, useVideoConfig } from "remotion";

const frame = useCurrentFrame();
// → 0, 1, 2, ..., durationInFrames-1

const { width, height, fps, durationInFrames } = useVideoConfig();
// → 从 Composition 定义中读取
```

---

## interpolate

线性映射：将 frame 值映射为任意属性值。

```ts
import { interpolate } from "remotion";

// 基本用法：帧 0~30，透明度 0→1
const opacity = interpolate(frame, [0, 30], [0, 1], {
  extrapolateLeft: "clamp",   // 超出左边界时夹住
  extrapolateRight: "clamp",  // 超出右边界时夹住（最常用）
});

// X 轴位移：帧 0~40，从 -100 移动到 0
const x = interpolate(frame, [0, 40], [-100, 0], {
  extrapolateRight: "clamp",
});

// 缩放：帧 10~50，从 0.8 放大到 1
const scale = interpolate(frame, [10, 50], [0.8, 1], {
  extrapolateRight: "clamp",
});
```

**常用缓动（easing）：**
```ts
import { Easing } from "remotion";

const value = interpolate(frame, [0, 30], [0, 1], {
  easing: Easing.out(Easing.cubic),  // 缓出（推荐大多数场景）
  extrapolateRight: "clamp",
});
```

---

## spring

物理弹簧动画，自带自然弹性，value 从 0 → 1（可能短暂超过 1 再回弹）。

```ts
import { spring, useCurrentFrame, useVideoConfig } from "remotion";

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

// 基本弹入（从第 0 帧开始）
const scale = spring({
  frame,
  fps,
  config: {
    damping: 12,     // 阻尼：越大收敛越快，无回弹
    stiffness: 150,  // 刚度：越大弹得越猛
    mass: 1,         // 质量：越大运动越慢
  },
});
// scale: 0 → ~1.05 → 1（带轻微回弹）

// 延迟开始（从第 20 帧开始弹入）
const delayedScale = spring({
  frame: Math.max(0, frame - 20),  // 减去延迟帧数
  fps,
  config: { damping: 12, stiffness: 150, mass: 1 },
});
```

**推荐配置：**
| 场景 | damping | stiffness | mass |
|------|---------|-----------|------|
| 标题弹入（快） | 12 | 200 | 1 |
| 卡片弹入（中） | 10 | 120 | 1 |
| 菜单展开（慢） | 8 | 80 | 1.5 |

**用于 CSS transform：**
```tsx
<div style={{
  transform: `scale(${scale}) translateY(${interpolate(scale, [0, 1], [50, 0])}px)`,
  opacity: scale,
}}>
```

---

## Sequence

让子组件在特定时间段内渲染，内部 `frame` 从 0 重新开始计数。

```tsx
import { Sequence } from "remotion";

// 从第 30 帧开始，持续 60 帧（第 30-90 帧可见）
<Sequence from={30} durationInFrames={60}>
  <MyAnimation />  {/* 内部 frame: 0, 1, ..., 59 */}
</Sequence>

// 只设置 from，不限制结束（直到视频结束）
<Sequence from={60}>
  <LongAnimation />
</Sequence>

// 命名（调试时显示在时间轴）
<Sequence from={0} durationInFrames={60} name="Intro">
  <IntroScene />
</Sequence>
```

**动态场景排列（帧号由 AI 在步骤3计算后内联写入）：**

> 不要写死 270 帧！根据场景计划表累加得出 totalFrames，在 Root.tsx 中填入。

```tsx
// 示例：10幕场景，totalFrames = 560
<>
  <Sequence from={0}   durationInFrames={70}  name="Intro">      <Intro {...props} /></Sequence>
  <Sequence from={70}  durationInFrames={60}  name="Tldr">       <TldrScene {...props} /></Sequence>
  <Sequence from={130} durationInFrames={55}  name="Context">    <ContextScene {...props} /></Sequence>
  <Sequence from={185} durationInFrames={65}  name="Problem">    <ProblemScene {...props} /></Sequence>
  <Sequence from={250} durationInFrames={60}  name="Concepts">   <ConceptsScene concepts={props.keyConcepts || []} /></Sequence>
  <Sequence from={310} durationInFrames={70}  name="Arch">       <ArchScene {...props} /></Sequence>
  <Sequence from={380} durationInFrames={55}  name="Files">      <FilesScene repoName={props.repoName} files={props.coreFiles.slice(0,4)} /></Sequence>
  <Sequence from={435} durationInFrames={60}  name="Quickstart"> <QuickstartScene steps={props.quickstart} /></Sequence>
  <Sequence from={495} durationInFrames={50}  name="Highlights"> <HighlightsScene highlights={props.highlights.slice(0,3)} /></Sequence>
  <Sequence from={545} durationInFrames={15}  name="Outro">      <OutroScene {...props} /></Sequence>
</>
// totalFrames = 560，视频约 18.7 秒
```

---

## AbsoluteFill

等同于 `position:absolute; top:0; left:0; right:0; bottom:0`，用于层叠全屏布局。

```tsx
import { AbsoluteFill } from "remotion";

<AbsoluteFill style={{ backgroundColor: "#0d1117" }}>
  {/* 全屏内容 */}
</AbsoluteFill>

// 多层叠加
<AbsoluteFill style={{ backgroundColor: "#0d1117" }} />  {/* 背景层 */}
<AbsoluteFill style={{ justifyContent: "center", alignItems: "center" }}>
  <h1>前景内容</h1>
</AbsoluteFill>
```

---

## Audio + staticFile

在 Sequence 内播放音频文件，用于 TTS 解说词配音。

```tsx
import { Audio, staticFile } from "remotion";

// 播放 public/audio/intro.mp3
<Audio src={staticFile("audio/intro.mp3")} />

// 条件播放（有解说词才加音频）
{narration && <Audio src={staticFile(`audio/${sceneKey}.mp3`)} />}
```

**关键说明：**
- `staticFile("audio/intro.mp3")` 引用 `public/audio/intro.mp3`
- 音频会从 `<Sequence>` 的第 0 帧开始播放
- 渲染时加 `--muted` 会跳过所有音频（降级为纯视觉版）
- 音频文件若不存在，Remotion 会报错，需确保 TTS 已生成

---

## Composition

在 Root.tsx 中注册视频规格：

```tsx
import { Composition } from "remotion";
import { GitHubExplainerVideo } from "./GitHubExplainerVideo";

export const Root = () => (
  <Composition
    id="GitHubExplainer"          // render 命令中使用的 ID
    component={GitHubExplainerVideo}
    durationInFrames={270}         // 总帧数 = 9秒 × 30fps
    fps={30}
    width={720}                    // 竖版
    height={1280}
    defaultProps={{                // 开发预览时的默认数据
      repoName: "ex-skill",
      tagline: "把前任蒸馏成AI",
      stars: "2.9k",
      language: "Python",
      problem: "无法重现和前任的对话体验",
      architecture: {
        layers: ["输入层", "分析层", "输出层"],
        coreComponent: "prompts/"
      },
      coreFiles: [
        { name: "SKILL.md", desc: "Skill入口" },
        { name: "prompts/", desc: "AI提示词模板" }
      ],
      highlights: ["五层性格模型", "记忆与性格分离", "对话自我纠正"],
      quickstart: ["git clone到.claude/skills/", "运行/create-ex", "用/{slug}对话"],
      audience: [
        { group: "Claude Code用户", fit: "strong" },
        { group: "Python开发者", fit: "ok" },
        { group: "普通用户", fit: "limited" }
      ],
      repoUrl: "https://github.com/therealXiaomanChu/ex-skill"
    }}
  />
);
```

---

## 渲染命令

```bash
# 标准渲染（720p 竖版）
npx remotion render GitHubExplainer out/video.mp4 \
  --props='{"repoName":"...","tagline":"..."}' \
  --codec=h264 \
  --crf=23 \
  --concurrency=4 \
  --muted

# 降分辨率（输出 360×640，文件更小）
npx remotion render GitHubExplainer out/video.mp4 \
  --props='...' \
  --scale=0.5 \
  --codec=h264

# 只渲染前 60 帧（调试 Intro 场景）
npx remotion render GitHubExplainer out/intro-test.mp4 \
  --frames=0-59 \
  --props='...'

# 开发预览
npx remotion studio
```

**输出文件大小参考（720×1280，30fps，9秒）：**
- `--crf=18`（高质量）：~8-15MB
- `--crf=23`（均衡，推荐）：~4-8MB
- `--crf=28`（较小）：~2-4MB

---

## package.json 模板

```json
{
  "name": "github-explainer-video",
  "version": "1.0.0",
  "scripts": {
    "start": "npx remotion studio",
    "render": "npx remotion render GitHubExplainer out/video.mp4 --codec=h264 --crf=23 --concurrency=4 --muted"
  },
  "dependencies": {
    "remotion": "^4.0.0",
    "@remotion/cli": "^4.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "typescript": "^5.0.0"
  }
}
```

**注意：** Remotion 4.x 需要 Node.js 18+。ffmpeg **无需系统安装**，Remotion 自带渲染能力（会自动下载 Chrome Headless Shell，首次约 108MB，后续缓存）。npm install 推荐加 `--registry=https://registry.npmmirror.com`。
