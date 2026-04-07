# 动态场景 React 代码模板（v2 — 支持可变幕数）

本文档包含所有场景组件的完整可运行代码。
**v2 新增场景：** TldrScene（大白话）、ContextScene（背景故事）、ConceptsScene（术语解释）、UseCasesScene（使用案例）。

## 目录
- [types.ts — 完整接口](#typests)
- [index.ts / Root.tsx 模板](#入口文件)
- [动态 GitHubExplainerVideo 生成规则](#动态主组件生成规则)
- [场景1 Intro.tsx](#intro)
- [场景2 TldrScene.tsx ★新](#tldrscene) — 大白话三句话
- [场景3 ContextScene.tsx ★新](#contextscene) — 背景故事
- [场景4 ProblemScene.tsx](#problemscene) — 痛点+analogy
- [场景5 ConceptsScene.tsx ★新](#conceptsscene) — 术语解释
- [场景6 ArchScene.tsx](#archscene)
- [场景7 FilesScene.tsx](#filesscene) — 支持分批
- [场景8 QuickstartScene.tsx](#quickstartscene) — 支持 detail/command
- [场景9 UseCasesScene.tsx ★新](#usecasesscene) — 使用案例
- [场景10 HighlightsScene.tsx](#highlightsscene) — 支持 detail/emoji
- [场景11 OutroScene.tsx](#outroscene)

---

## types.ts

```ts
// src/types.ts — v2 完整接口

export interface VideoProps {
  repoName: string;
  tagline: string;
  stars: string;
  language: string;

  // 大白话简介（3句话，合计≤80字）
  tldr: string;

  // 背景故事（可选）
  background?: string;

  // 痛点（含 before/after/analogy）
  problem: {
    description: string;
    before: string;
    after: string;
    analogy?: string;
  };

  // 术语解释（可选，2-4个）
  keyConcepts?: Array<{
    term: string;
    plain: string;
    emoji: string;
  }>;

  // 架构
  architecture: {
    analogy?: string;
    layers: string[];
    coreComponent: string;
  };

  // 核心文件（3-8个，含 why 字段）
  coreFiles: Array<{
    name: string;
    desc: string;
    why?: string;
  }>;

  // 上手步骤（含 detail/command）
  quickstart: Array<{
    step: string;
    detail?: string;
    command?: string;
  }>;

  // 使用案例（可选，0-3个）
  useCases?: Array<{
    scene: string;
    example: string;
  }>;

  // 亮点（含 detail/emoji）
  highlights: Array<{
    title: string;
    detail: string;
    emoji: string;
  }>;

  // 适合人群
  audience: Array<{
    group: string;
    fit: 'strong' | 'ok' | 'limited';
    reason?: string;
  }>;

  repoUrl: string;

  // 场景解说词（TTS 生成用），key 格式：场景名（如 intro / tldr / files_0 等）
  // 值为中文解说词（30-60字），用于 NarrationBar 字幕显示 + edge-tts 生成音频
  sceneNarrations?: Record<string, string>;
}

export const COLORS = {
  bgPrimary: '#0d1117',
  bgSecondary: '#161b22',
  bgTertiary: '#21262d',
  accentBlue: '#58a6ff',
  accentGreen: '#3fb950',
  accentOrange: '#f78166',
  accentPurple: '#bc8cff',
  accentPink: '#ff7eb3',
  accentYellow: '#d29922',
  textPrimary: '#e6edf3',
  textSecondary: '#8b949e',
  textMuted: '#484f58',
  border: '#30363d',
} as const;

export const SPACING = { xs: 8, sm: 16, md: 24, lg: 40, xl: 64 } as const;

export const fitConfig = {
  strong: { icon: '✅', label: '强烈推荐', color: '#3fb950' },
  ok:     { icon: '🔶', label: '推荐', color: '#58a6ff' },
  limited:{ icon: '⚠️', label: '有门槛', color: '#f78166' },
} as const;
```

---

## 入口文件

```ts
// src/index.ts
import { registerRoot } from 'remotion';
import { Root } from './Root';
registerRoot(Root);
```

```tsx
// src/Root.tsx — totalFrames 必须从场景计划动态计算，不要写死
import React from 'react';
import { Composition } from 'remotion';
import { GitHubExplainerVideo } from './GitHubExplainerVideo';
import type { VideoProps } from './types';

// totalFrames: 所有场景 durationInFrames 之和（由 AI 在步骤3计算后填入）
const TOTAL_FRAMES = __TOTAL_FRAMES_PLACEHOLDER__;

const defaultProps: VideoProps = { /* 第二步生成的 JSON */ };

export const Root: React.FC = () => (
  <Composition
    id="GitHubExplainer"
    component={GitHubExplainerVideo}
    durationInFrames={TOTAL_FRAMES}
    fps={30}
    width={720}
    height={1280}
    defaultProps={defaultProps}
  />
);
```

---

## NarrationBar ★新 — 底部字幕条

每个场景底部的字幕条，显示解说词文字并自动淡入淡出。

```tsx
// src/scenes/NarrationBar.tsx
import React from 'react';
import { interpolate, useCurrentFrame } from 'remotion';

interface NarrationBarProps {
  text: string;
  duration: number; // 当前场景总帧数，用于淡出时机
}

export const NarrationBar: React.FC<NarrationBarProps> = ({ text, duration }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(
    frame,
    [0, 10, duration - 10, duration],
    [0, 1, 1, 0],
    { extrapolateRight: 'clamp', extrapolateLeft: 'clamp' }
  );

  return (
    <div style={{
      position: 'absolute',
      bottom: 0, left: 0, right: 0,
      background: 'linear-gradient(to top, rgba(0,0,0,0.88) 0%, rgba(0,0,0,0.55) 65%, transparent 100%)',
      padding: '52px 32px 28px',
      opacity,
    }}>
      <p style={{
        color: '#ffffff',
        fontSize: 22,
        lineHeight: 1.7,
        margin: 0,
        fontFamily: 'PingFang SC, system-ui, sans-serif',
        textShadow: '0 1px 6px rgba(0,0,0,0.7)',
        letterSpacing: '0.03em',
      }}>
        {text}
      </p>
    </div>
  );
};
```

---

## SceneWithNarration ★新 — 带旁白的场景包装

包裹任意场景，自动叠加字幕条 + TTS 音频。
若 `narration` 为 undefined，静默跳过，不影响渲染。

```tsx
// src/scenes/SceneWithNarration.tsx
import React from 'react';
import { AbsoluteFill, Audio, staticFile } from 'remotion';
import { NarrationBar } from './NarrationBar';

interface SceneWithNarrationProps {
  narration?: string;   // 解说词文字（来自 props.sceneNarrations.xxx）
  sceneKey: string;     // 音频文件名（不含 .mp3），如 'intro'、'files_0'
  duration: number;     // 当前场景帧数，传给 NarrationBar 控制淡出时机
  children: React.ReactNode;
}

export const SceneWithNarration: React.FC<SceneWithNarrationProps> = ({
  narration,
  sceneKey,
  duration,
  children,
}) => (
  <AbsoluteFill>
    {children}
    {narration && (
      <>
        {/* 音频：来自 public/audio/{sceneKey}.mp3，由 Step 3.5 TTS 生成 */}
        <Audio src={staticFile(`audio/${sceneKey}.mp3`)} />
        {/* 字幕条：底部渐变区域显示解说词 */}
        <NarrationBar text={narration} duration={duration} />
      </>
    )}
  </AbsoluteFill>
);
```

**使用方式（在 GitHubExplainerVideo.tsx 中）：**

```tsx
import { SceneWithNarration } from './scenes/SceneWithNarration';

// 每个 Sequence 内层都用 SceneWithNarration 包裹
<Sequence from={0} durationInFrames={d_intro} name="Intro">
  <SceneWithNarration
    narration={props.sceneNarrations?.intro}
    sceneKey="intro"
    duration={d_intro}
  >
    <Intro {...props} />
  </SceneWithNarration>
</Sequence>

<Sequence from={f1} durationInFrames={d_files0} name="Files-1">
  <SceneWithNarration
    narration={props.sceneNarrations?.files_0}
    sceneKey="files_0"
    duration={d_files0}
  >
    <FilesScene repoName={props.repoName} files={props.coreFiles.slice(0,4)} />
  </SceneWithNarration>
</Sequence>
```

---

## 动态主组件生成规则

`GitHubExplainerVideo.tsx` 的内容**由 AI 在第三步根据场景计划动态生成**，不要复制固定版本。

生成规则：
1. 计算每个场景的 `startFrame`（前面所有场景 `duration` 累加）
2. 条件场景（ContextScene、ConceptsScene、UseCasesScene）仅在对应字段有数据时加入
3. 文件/步骤/亮点超过阈值时按批次拆分（FilesScene 每4个一批，Quickstart 每3个一批，Highlights 每3个一批）

**生成示例（实际场景和帧数取决于内容）：**

```tsx
// src/GitHubExplainerVideo.tsx — 示例：含 background + keyConcepts + useCases 的丰富项目
import React from 'react';
import { AbsoluteFill, Sequence } from 'remotion';
import type { VideoProps } from './types';
import { COLORS } from './types';
import { Intro } from './scenes/Intro';
import { TldrScene } from './scenes/TldrScene';
import { ContextScene } from './scenes/ContextScene';
import { ProblemScene } from './scenes/ProblemScene';
import { ConceptsScene } from './scenes/ConceptsScene';
import { ArchScene } from './scenes/ArchScene';
import { FilesScene } from './scenes/FilesScene';
import { QuickstartScene } from './scenes/QuickstartScene';
import { UseCasesScene } from './scenes/UseCasesScene';
import { HighlightsScene } from './scenes/HighlightsScene';
import { OutroScene } from './scenes/OutroScene';

// 场景计划（由 AI 在步骤3计算，直接内联帧号）
// Intro:70 + Tldr:60 + Context:55 + Problem:65 + Concepts:60 + Arch:70 + Files:55+55 + Quickstart:60 + UseCases:65 + Highlights:50+50 + Outro:20 = 735
export const GitHubExplainerVideo: React.FC<VideoProps> = (props) => (
  <AbsoluteFill style={{ backgroundColor: COLORS.bgPrimary }}>
    <Sequence from={0}   durationInFrames={70}  name="Intro">       <Intro {...props} /></Sequence>
    <Sequence from={70}  durationInFrames={60}  name="Tldr">        <TldrScene {...props} /></Sequence>
    <Sequence from={130} durationInFrames={55}  name="Context">     <ContextScene {...props} /></Sequence>
    <Sequence from={185} durationInFrames={65}  name="Problem">     <ProblemScene {...props} /></Sequence>
    <Sequence from={250} durationInFrames={60}  name="Concepts">    <ConceptsScene concepts={props.keyConcepts || []} /></Sequence>
    <Sequence from={310} durationInFrames={70}  name="Arch">        <ArchScene {...props} /></Sequence>
    <Sequence from={380} durationInFrames={55}  name="Files-1">     <FilesScene repoName={props.repoName} files={props.coreFiles.slice(0,4)} /></Sequence>
    <Sequence from={435} durationInFrames={55}  name="Files-2">     <FilesScene repoName={props.repoName} files={props.coreFiles.slice(4,8)} /></Sequence>
    <Sequence from={490} durationInFrames={60}  name="Quickstart">  <QuickstartScene steps={props.quickstart} /></Sequence>
    <Sequence from={550} durationInFrames={65}  name="UseCases">    <UseCasesScene useCases={props.useCases || []} /></Sequence>
    <Sequence from={615} durationInFrames={50}  name="Highlights-1"><HighlightsScene highlights={props.highlights.slice(0,3)} /></Sequence>
    <Sequence from={665} durationInFrames={50}  name="Highlights-2"><HighlightsScene highlights={props.highlights.slice(3,6)} /></Sequence>
    <Sequence from={715} durationInFrames={20}  name="Outro">       <OutroScene {...props} /></Sequence>
  </AbsoluteFill>
);
// TOTAL_FRAMES = 735，视频约 24.5 秒
```

---

## Intro

帧范围：0-70（内部，2.3秒）。封面：项目名弹入 + tagline + stats + 装饰光晕。

```tsx
// src/scenes/Intro.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import type { VideoProps } from '../types';
import { COLORS, SPACING } from '../types';

export const Intro: React.FC<VideoProps> = ({ repoName, tagline, stars, language }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const bgOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 180, mass: 1 } });
  const titleOpacity = interpolate(frame, [0, 10], [0, 1], { extrapolateRight: 'clamp' });
  const taglineOpacity = interpolate(frame, [15, 35], [0, 1], { extrapolateRight: 'clamp' });
  const taglineY = interpolate(frame, [15, 35], [20, 0], { extrapolateRight: 'clamp' });
  const badgeScale = spring({ frame: Math.max(0, frame - 25), fps, config: { damping: 10, stiffness: 150 } });
  const lineWidth = interpolate(frame, [30, 60], [0, 100], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ opacity: bgOpacity, backgroundColor: COLORS.bgPrimary }}>
      {/* 背景光晕 */}
      <div style={{
        position: 'absolute', top: '22%', left: '50%',
        transform: 'translate(-50%,-50%)',
        width: 450, height: 450,
        background: `radial-gradient(circle, ${COLORS.accentBlue}18 0%, transparent 70%)`,
        pointerEvents: 'none',
      }} />

      <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center', flexDirection: 'column', gap: SPACING.lg, padding: `0 ${SPACING.xl}px` }}>
        {/* 顶部标签 */}
        <div style={{ transform: `scale(${badgeScale})`, backgroundColor: `${COLORS.accentBlue}18`, border: `1px solid ${COLORS.accentBlue}44`, borderRadius: 20, padding: '6px 18px', color: COLORS.accentBlue, fontSize: 20, fontWeight: 500, fontFamily: 'system-ui, sans-serif' }}>
          GitHub 项目解读
        </div>

        {/* 项目名 */}
        <div style={{ transform: `scale(${titleScale})`, opacity: titleOpacity, textAlign: 'center' }}>
          <div style={{ fontSize: 72, fontWeight: 800, color: COLORS.textPrimary, letterSpacing: -2, lineHeight: 1.1, fontFamily: 'system-ui, sans-serif' }}>
            {repoName}
          </div>
        </div>

        {/* 装饰线 */}
        <div style={{ width: `${lineWidth}%`, height: 2, background: `linear-gradient(90deg, transparent, ${COLORS.accentBlue}, transparent)` }} />

        {/* 标语 */}
        <div style={{ opacity: taglineOpacity, transform: `translateY(${taglineY}px)`, fontSize: 30, fontWeight: 400, color: COLORS.textSecondary, textAlign: 'center', lineHeight: 1.6, maxWidth: 560, fontFamily: 'system-ui, sans-serif' }}>
          {tagline}
        </div>

        {/* Stars + Language */}
        <div style={{ display: 'flex', gap: SPACING.sm, transform: `scale(${badgeScale})`, opacity: badgeScale }}>
          <span style={{ backgroundColor: `${COLORS.accentYellow}18`, border: `1px solid ${COLORS.accentYellow}44`, borderRadius: 8, padding: '8px 18px', color: COLORS.accentYellow, fontSize: 22, fontWeight: 600, fontFamily: 'system-ui, sans-serif' }}>⭐ {stars}</span>
          <span style={{ backgroundColor: `${COLORS.accentBlue}18`, border: `1px solid ${COLORS.accentBlue}44`, borderRadius: 8, padding: '8px 18px', color: COLORS.accentBlue, fontSize: 22, fontWeight: 600, fontFamily: 'system-ui, sans-serif' }}>{language}</span>
        </div>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

---

## TldrScene

★ 新场景。大白话3句话，逐句淡入。帧范围：内部0-60。

```tsx
// src/scenes/TldrScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import type { VideoProps } from '../types';
import { COLORS, SPACING } from '../types';

export const TldrScene: React.FC<VideoProps> = ({ repoName, tldr }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // 把 tldr 拆成句子（按句号/！/？拆）
  const sentences = tldr.split(/(?<=[。！？!?])\s*/).filter(s => s.trim().length > 0).slice(0, 3);

  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, justifyContent: 'center', alignItems: 'center', flexDirection: 'column' }}>
      {/* 标题 */}
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        💬 一句话听懂 {repoName}
      </div>

      {/* 大白话卡片 */}
      <div style={{ backgroundColor: COLORS.bgSecondary, border: `1px solid ${COLORS.border}`, borderRadius: 20, padding: SPACING.lg, maxWidth: 580, width: '100%' }}>
        {sentences.map((sentence, i) => {
          const delay = 10 + i * 14;
          const sOpacity = interpolate(frame, [delay, delay + 16], [0, 1], { extrapolateRight: 'clamp' });
          const sY = interpolate(frame, [delay, delay + 16], [18, 0], { extrapolateRight: 'clamp' });
          const icons = ['①', '②', '③'];

          return (
            <div key={i} style={{ opacity: sOpacity, transform: `translateY(${sY}px)`, display: 'flex', alignItems: 'flex-start', gap: SPACING.sm, marginBottom: i < sentences.length - 1 ? SPACING.md : 0 }}>
              <span style={{ fontSize: 26, color: COLORS.accentBlue, flexShrink: 0, marginTop: 2, fontFamily: 'system-ui, sans-serif' }}>{icons[i]}</span>
              <div style={{ fontSize: 24, color: COLORS.textPrimary, lineHeight: 1.6, fontFamily: 'system-ui, sans-serif', fontWeight: 400 }}>
                {sentence.trim()}
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

---

## ContextScene

★ 新场景（条件生成）。项目背景故事。帧范围：内部0-55。

```tsx
// src/scenes/ContextScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import type { VideoProps } from '../types';
import { COLORS, SPACING } from '../types';

export const ContextScene: React.FC<VideoProps> = ({ background, repoName }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const cardOpacity = interpolate(frame, [12, 30], [0, 1], { extrapolateRight: 'clamp' });
  const cardY = interpolate(frame, [12, 30], [30, 0], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, justifyContent: 'center', alignItems: 'center', flexDirection: 'column' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentPurple, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        📖 为什么会有这个项目
      </div>

      <div style={{ opacity: cardOpacity, transform: `translateY(${cardY}px)`, backgroundColor: `${COLORS.accentPurple}12`, border: `1px solid ${COLORS.accentPurple}44`, borderRadius: 20, padding: SPACING.lg, maxWidth: 580, width: '100%' }}>
        {/* 引号装饰 */}
        <div style={{ fontSize: 60, color: COLORS.accentPurple, opacity: 0.3, lineHeight: 0.7, marginBottom: SPACING.sm, fontFamily: 'Georgia, serif' }}>"</div>
        <div style={{ fontSize: 23, color: COLORS.textPrimary, lineHeight: 1.7, fontFamily: 'system-ui, sans-serif', fontWeight: 400 }}>
          {background}
        </div>
        <div style={{ marginTop: SPACING.md, fontSize: 18, color: COLORS.textSecondary, fontFamily: 'system-ui, sans-serif' }}>
          — {repoName} 项目动机
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

---

## ProblemScene

帧范围：内部0-65。痛点：before/after + analogy。

```tsx
// src/scenes/ProblemScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import type { VideoProps } from '../types';
import { COLORS, SPACING } from '../types';

export const ProblemScene: React.FC<VideoProps> = ({ problem }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const beforeX = interpolate(frame, [8, 26], [-200, 0], { extrapolateRight: 'clamp' });
  const beforeOpacity = interpolate(frame, [8, 26], [0, 1], { extrapolateRight: 'clamp' });
  const afterX = interpolate(frame, [18, 36], [200, 0], { extrapolateRight: 'clamp' });
  const afterOpacity = interpolate(frame, [18, 36], [0, 1], { extrapolateRight: 'clamp' });
  const arrowScale = spring({ frame: Math.max(0, frame - 22), fps, config: { damping: 10, stiffness: 200 } });
  const analogyOpacity = interpolate(frame, [38, 54], [0, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.md, fontFamily: 'system-ui, sans-serif' }}>
        🤔 它解决了什么问题
      </div>

      <div style={{ fontSize: 22, color: COLORS.textSecondary, textAlign: 'center', marginBottom: SPACING.lg, lineHeight: 1.6, fontFamily: 'system-ui, sans-serif', maxWidth: 560 }}>
        {problem.description}
      </div>

      {/* Before/After */}
      <div style={{ display: 'flex', gap: SPACING.md, alignItems: 'center', width: '100%', marginBottom: SPACING.md }}>
        <div style={{ transform: `translateX(${beforeX}px)`, opacity: beforeOpacity, flex: 1, backgroundColor: `${COLORS.accentOrange}12`, border: `1px solid ${COLORS.accentOrange}44`, borderRadius: 16, padding: SPACING.md, textAlign: 'center' }}>
          <div style={{ fontSize: 40, marginBottom: 8 }}>😔</div>
          <div style={{ fontSize: 20, fontWeight: 700, color: COLORS.accentOrange, marginBottom: 8, fontFamily: 'system-ui, sans-serif' }}>之前</div>
          <div style={{ fontSize: 17, color: COLORS.textSecondary, lineHeight: 1.6, fontFamily: 'system-ui, sans-serif' }}>{problem.before}</div>
        </div>
        <div style={{ transform: `scale(${arrowScale})`, fontSize: 40, color: COLORS.accentBlue, flexShrink: 0 }}>→</div>
        <div style={{ transform: `translateX(${afterX}px)`, opacity: afterOpacity, flex: 1, backgroundColor: `${COLORS.accentGreen}12`, border: `1px solid ${COLORS.accentGreen}44`, borderRadius: 16, padding: SPACING.md, textAlign: 'center' }}>
          <div style={{ fontSize: 40, marginBottom: 8 }}>✨</div>
          <div style={{ fontSize: 20, fontWeight: 700, color: COLORS.accentGreen, marginBottom: 8, fontFamily: 'system-ui, sans-serif' }}>之后</div>
          <div style={{ fontSize: 17, color: COLORS.textSecondary, lineHeight: 1.6, fontFamily: 'system-ui, sans-serif' }}>{problem.after}</div>
        </div>
      </div>

      {/* Analogy（可选）*/}
      {problem.analogy && (
        <div style={{ opacity: analogyOpacity, backgroundColor: `${COLORS.accentBlue}10`, border: `1px solid ${COLORS.accentBlue}30`, borderRadius: 12, padding: `${SPACING.sm}px ${SPACING.md}px`, maxWidth: 560 }}>
          <span style={{ fontSize: 18, color: COLORS.accentBlue, fontFamily: 'system-ui, sans-serif' }}>💡 {problem.analogy}</span>
        </div>
      )}
    </AbsoluteFill>
  );
};
```

---

## ConceptsScene

★ 新场景（条件生成）。术语卡片，每次展示最多4个概念。帧范围：内部0-60。

```tsx
// src/scenes/ConceptsScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { COLORS, SPACING } from '../types';

interface ConceptsSceneProps {
  concepts: Array<{ term: string; plain: string; emoji: string }>;
}

export const ConceptsScene: React.FC<ConceptsSceneProps> = ({ concepts }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });

  const conceptColors = [COLORS.accentBlue, COLORS.accentPurple, COLORS.accentGreen, COLORS.accentOrange];

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        📚 术语小词典
      </div>

      <div style={{ display: 'flex', flexDirection: 'column', gap: SPACING.sm, width: '100%', maxWidth: 580 }}>
        {concepts.slice(0, 4).map((concept, i) => {
          const delay = 8 + i * 10;
          const cardSpring = spring({ frame: Math.max(0, frame - delay), fps, config: { damping: 10, stiffness: 140 } });
          const color = conceptColors[i % conceptColors.length];

          return (
            <div key={i} style={{ opacity: cardSpring, transform: `translateX(${(1 - cardSpring) * -40}px)`, backgroundColor: `${color}12`, border: `1px solid ${color}44`, borderRadius: 14, padding: SPACING.md, display: 'flex', alignItems: 'flex-start', gap: SPACING.md }}>
              <span style={{ fontSize: 32, flexShrink: 0 }}>{concept.emoji}</span>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 20, fontWeight: 700, color, marginBottom: 6, fontFamily: 'system-ui, sans-serif' }}>
                  {concept.term}
                </div>
                <div style={{ fontSize: 18, color: COLORS.textPrimary, lineHeight: 1.5, fontFamily: 'system-ui, sans-serif' }}>
                  {concept.plain}
                </div>
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

---

## ArchScene

帧范围：内部0-(50+layers.length×10)。架构层级，层数越多持续越长。

```tsx
// src/scenes/ArchScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import type { VideoProps } from '../types';
import { COLORS, SPACING } from '../types';

export const ArchScene: React.FC<VideoProps> = ({ architecture }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const analogyOpacity = interpolate(frame, [5, 18], [0, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.sm, fontFamily: 'system-ui, sans-serif' }}>
        🏗️ 工作原理
      </div>

      {/* 类比（可选）*/}
      {architecture.analogy && (
        <div style={{ opacity: analogyOpacity, fontSize: 18, color: COLORS.textSecondary, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif', fontStyle: 'italic' }}>
          💡 {architecture.analogy}
        </div>
      )}

      <div style={{ display: 'flex', flexDirection: 'column', gap: SPACING.xs, flex: 1, justifyContent: 'center', width: '100%', maxWidth: 580 }}>
        {architecture.layers.map((layer, i) => {
          const delay = (architecture.analogy ? 14 : 8) + i * 12;
          const layerSpring = spring({ frame: Math.max(0, frame - delay), fps, config: { damping: 12, stiffness: 140 } });
          const isCore = i === Math.floor(architecture.layers.length / 2);

          return (
            <React.Fragment key={i}>
              <div style={{ transform: `scale(${layerSpring})`, opacity: layerSpring, width: '100%', backgroundColor: isCore ? `${COLORS.accentBlue}18` : COLORS.bgSecondary, border: `${isCore ? 2 : 1}px solid ${isCore ? COLORS.accentBlue : COLORS.border}`, boxShadow: isCore ? `0 0 20px ${COLORS.accentBlue}30` : 'none', borderRadius: 12, padding: `${SPACING.sm}px ${SPACING.lg}px`, textAlign: 'center' }}>
                <div style={{ fontSize: isCore ? 26 : 22, fontWeight: isCore ? 700 : 500, color: isCore ? COLORS.accentBlue : COLORS.textPrimary, fontFamily: 'system-ui, sans-serif' }}>
                  {layer}
                </div>
              </div>
              {i < architecture.layers.length - 1 && (
                <div style={{ opacity: layerSpring, textAlign: 'center', fontSize: 24, color: COLORS.textMuted }}>↓</div>
              )}
            </React.Fragment>
          );
        })}
      </div>

      <div style={{ textAlign: 'center', marginTop: SPACING.md, fontSize: 19, color: COLORS.textSecondary, opacity: interpolate(frame, [40, 55], [0, 1], { extrapolateRight: 'clamp' }), fontFamily: 'system-ui, sans-serif' }}>
        核心：<span style={{ color: COLORS.accentBlue, fontWeight: 600 }}>{architecture.coreComponent}</span>
      </div>
    </AbsoluteFill>
  );
};
```

---

## FilesScene

支持批次（每批≤4个文件）。传入 `files` 子数组，而不是全量 `coreFiles`。

```tsx
// src/scenes/FilesScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, useCurrentFrame } from 'remotion';
import { COLORS, SPACING } from '../types';

interface FilesSceneProps {
  repoName: string;
  files: Array<{ name: string; desc: string; why?: string }>;
}

export const FilesScene: React.FC<FilesSceneProps> = ({ repoName, files }) => {
  const frame = useCurrentFrame();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleOpacity = interpolate(frame, [0, 12], [0, 1], { extrapolateRight: 'clamp' });

  const getIcon = (name: string) => {
    if (name.endsWith('/')) return '📁';
    if (name.endsWith('.py')) return '🐍';
    if (name.endsWith('.ts') || name.endsWith('.tsx')) return '💙';
    if (name.endsWith('.md')) return '📄';
    if (name.endsWith('.json')) return '📦';
    return '📝';
  };

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ opacity: titleOpacity, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        📂 核心文件解读
      </div>

      <div style={{ backgroundColor: COLORS.bgSecondary, border: `1px solid ${COLORS.border}`, borderRadius: 14, padding: SPACING.md, width: '100%', maxWidth: 580, fontFamily: 'monospace' }}>
        <div style={{ fontSize: 20, color: COLORS.accentBlue, fontWeight: 700, marginBottom: SPACING.sm }}>📦 {repoName}/</div>

        {files.map((file, i) => {
          const delay = 8 + i * 8;
          const fileOpacity = interpolate(frame, [delay, delay + 14], [0, 1], { extrapolateRight: 'clamp' });
          const fileX = interpolate(frame, [delay, delay + 14], [-30, 0], { extrapolateRight: 'clamp' });

          return (
            <div key={i} style={{ transform: `translateX(${fileX}px)`, opacity: fileOpacity, display: 'flex', alignItems: 'flex-start', gap: SPACING.sm, padding: `${SPACING.xs}px 0`, borderBottom: i < files.length - 1 ? `1px solid ${COLORS.bgTertiary}` : 'none' }}>
              <span style={{ color: COLORS.textMuted, fontSize: 18, flexShrink: 0 }}>{i < files.length - 1 ? '├──' : '└──'}</span>
              <span style={{ fontSize: 20, flexShrink: 0 }}>{getIcon(file.name)}</span>
              <div style={{ flex: 1, minWidth: 0 }}>
                <div style={{ fontSize: 20, color: COLORS.textPrimary, fontWeight: 600 }}>{file.name}</div>
                <div style={{ fontSize: 16, color: COLORS.textSecondary, marginTop: 2, fontFamily: 'system-ui, sans-serif' }}>{file.desc}</div>
                {file.why && (
                  <div style={{ fontSize: 14, color: COLORS.accentBlue, marginTop: 3, fontFamily: 'system-ui, sans-serif', opacity: 0.85 }}>→ {file.why}</div>
                )}
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

---

## QuickstartScene

支持 `detail` 和 `command` 字段。传入 steps 子数组（每批≤3步）。

```tsx
// src/scenes/QuickstartScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { COLORS, SPACING } from '../types';

interface QuickstartSceneProps {
  steps: Array<{ step: string; detail?: string; command?: string }>;
}

export const QuickstartScene: React.FC<QuickstartSceneProps> = ({ steps }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const stepColors = [COLORS.accentBlue, COLORS.accentPurple, COLORS.accentGreen];

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        🚀 快速上手
      </div>

      <div style={{ display: 'flex', flexDirection: 'column', gap: SPACING.md, width: '100%', maxWidth: 580 }}>
        {steps.map((s, i) => {
          const delay = 8 + i * 12;
          const cardY = interpolate(frame, [delay, delay + 20], [50, 0], { extrapolateRight: 'clamp' });
          const cardOpacity = interpolate(frame, [delay, delay + 20], [0, 1], { extrapolateRight: 'clamp' });
          const color = stepColors[i % stepColors.length];

          return (
            <div key={i} style={{ transform: `translateY(${cardY}px)`, opacity: cardOpacity, backgroundColor: `${color}10`, border: `1px solid ${color}44`, borderRadius: 16, padding: SPACING.md, display: 'flex', alignItems: 'flex-start', gap: SPACING.md }}>
              <div style={{ width: 40, height: 40, borderRadius: '50%', backgroundColor: color, display: 'flex', alignItems: 'center', justifyContent: 'center', fontSize: 20, fontWeight: 800, color: COLORS.bgPrimary, flexShrink: 0, fontFamily: 'system-ui, sans-serif' }}>{i + 1}</div>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 20, color: COLORS.textPrimary, fontWeight: 600, marginBottom: s.detail || s.command ? 6 : 0, fontFamily: 'system-ui, sans-serif' }}>{s.step}</div>
                {s.detail && <div style={{ fontSize: 16, color: COLORS.textSecondary, marginBottom: s.command ? 6 : 0, fontFamily: 'system-ui, sans-serif' }}>{s.detail}</div>}
                {s.command && (
                  <div style={{ backgroundColor: COLORS.bgTertiary, borderRadius: 8, padding: '4px 12px', fontFamily: 'monospace', fontSize: 16, color: COLORS.accentGreen, display: 'inline-block' }}>
                    {s.command}
                  </div>
                )}
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

---

## UseCasesScene

★ 新场景（条件生成）。展示 2-3 个真实使用案例。帧范围：内部0-65。

```tsx
// src/scenes/UseCasesScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { COLORS, SPACING } from '../types';

interface UseCasesSceneProps {
  useCases: Array<{ scene: string; example: string }>;
}

export const UseCasesScene: React.FC<UseCasesSceneProps> = ({ useCases }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const caseEmojis = ['🌙', '☕', '💻', '🎯'];
  const caseColors = [COLORS.accentPink, COLORS.accentYellow, COLORS.accentBlue];

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        🎬 实际使用场景
      </div>

      <div style={{ display: 'flex', flexDirection: 'column', gap: SPACING.md, width: '100%', maxWidth: 580 }}>
        {useCases.slice(0, 3).map((uc, i) => {
          const delay = 8 + i * 14;
          const cardScale = spring({ frame: Math.max(0, frame - delay), fps, config: { damping: 10, stiffness: 130 } });
          const color = caseColors[i % caseColors.length];

          return (
            <div key={i} style={{ opacity: cardScale, transform: `scale(${0.85 + cardScale * 0.15})`, backgroundColor: `${color}12`, border: `1px solid ${color}44`, borderRadius: 16, padding: SPACING.md }}>
              <div style={{ display: 'flex', alignItems: 'center', gap: SPACING.sm, marginBottom: SPACING.xs }}>
                <span style={{ fontSize: 26 }}>{caseEmojis[i]}</span>
                <div style={{ fontSize: 20, fontWeight: 700, color, fontFamily: 'system-ui, sans-serif' }}>{uc.scene}</div>
              </div>
              <div style={{ fontSize: 17, color: COLORS.textSecondary, lineHeight: 1.6, fontFamily: 'system-ui, sans-serif', paddingLeft: 38 }}>
                {uc.example}
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

---

## HighlightsScene

支持 `detail` 和 `emoji` 字段。传入 highlights 子数组（每批≤3个）。

```tsx
// src/scenes/HighlightsScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { COLORS, SPACING } from '../types';

interface HighlightsSceneProps {
  highlights: Array<{ title: string; detail: string; emoji: string }>;
}

export const HighlightsScene: React.FC<HighlightsSceneProps> = ({ highlights }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const sceneOpacity = interpolate(frame, [0, 8], [0, 1], { extrapolateRight: 'clamp' });
  const titleScale = spring({ frame, fps, config: { damping: 12, stiffness: 160 } });
  const cardColors = [COLORS.accentBlue, COLORS.accentPurple, COLORS.accentGreen];

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ transform: `scale(${titleScale})`, fontSize: 38, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        ⭐ 项目亮点
      </div>

      <div style={{ display: 'flex', flexDirection: 'column', gap: SPACING.md, width: '100%', maxWidth: 580 }}>
        {highlights.slice(0, 3).map((h, i) => {
          const delay = 6 + i * 10;
          const cardSpring = spring({ frame: Math.max(0, frame - delay), fps, config: { damping: 10, stiffness: 150 } });
          const color = cardColors[i % cardColors.length];

          return (
            <div key={i} style={{ opacity: cardSpring, transform: `scale(${0.88 + cardSpring * 0.12})`, backgroundColor: `${color}12`, border: `1px solid ${color}55`, borderRadius: 16, padding: `${SPACING.sm}px ${SPACING.md}px`, display: 'flex', alignItems: 'flex-start', gap: SPACING.md }}>
              <span style={{ fontSize: 32, flexShrink: 0 }}>{h.emoji}</span>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 21, fontWeight: 700, color, marginBottom: 5, fontFamily: 'system-ui, sans-serif' }}>{h.title}</div>
                <div style={{ fontSize: 17, color: COLORS.textSecondary, lineHeight: 1.5, fontFamily: 'system-ui, sans-serif' }}>{h.detail}</div>
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

---

## OutroScene

帧范围：内部0-20。适合人群 + 仓库链接。

```tsx
// src/scenes/OutroScene.tsx
import React from 'react';
import { AbsoluteFill, interpolate, useCurrentFrame } from 'remotion';
import type { VideoProps } from '../types';
import { COLORS, SPACING, fitConfig } from '../types';

export const OutroScene: React.FC<VideoProps> = ({ audience, repoUrl, repoName }) => {
  const frame = useCurrentFrame();
  const sceneOpacity = interpolate(frame, [0, 6], [0, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ opacity: sceneOpacity, backgroundColor: COLORS.bgPrimary, padding: SPACING.xl, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' }}>
      <div style={{ fontSize: 36, fontWeight: 700, color: COLORS.accentBlue, textAlign: 'center', marginBottom: SPACING.lg, fontFamily: 'system-ui, sans-serif' }}>
        👥 适合谁用？
      </div>

      <div style={{ display: 'flex', flexDirection: 'column', gap: SPACING.sm, marginBottom: SPACING.lg, width: '100%', maxWidth: 520 }}>
        {audience.map((item, i) => {
          const { icon, color, label } = fitConfig[item.fit];
          return (
            <div key={i} style={{ display: 'flex', alignItems: 'center', gap: SPACING.sm, backgroundColor: COLORS.bgSecondary, border: `1px solid ${COLORS.border}`, borderRadius: 12, padding: `${SPACING.sm}px ${SPACING.md}px` }}>
              <span style={{ fontSize: 24 }}>{icon}</span>
              <span style={{ fontSize: 20, color: COLORS.textPrimary, flex: 1, fontFamily: 'system-ui, sans-serif' }}>{item.group}</span>
              <span style={{ fontSize: 16, color, fontWeight: 600, fontFamily: 'system-ui, sans-serif' }}>{label}</span>
              {item.reason && <span style={{ fontSize: 14, color: COLORS.textMuted, fontFamily: 'system-ui, sans-serif' }}>— {item.reason}</span>}
            </div>
          );
        })}
      </div>

      {/* 项目链接 */}
      <div style={{ backgroundColor: `${COLORS.accentBlue}12`, border: `1px solid ${COLORS.accentBlue}44`, borderRadius: 12, padding: SPACING.md, textAlign: 'center', maxWidth: 520, width: '100%' }}>
        <div style={{ fontSize: 18, color: COLORS.textSecondary, marginBottom: 6, fontFamily: 'system-ui, sans-serif' }}>🔗 {repoName}</div>
        <div style={{ fontSize: 15, color: COLORS.accentBlue, fontFamily: 'monospace', wordBreak: 'break-all' }}>{repoUrl}</div>
      </div>
    </AbsoluteFill>
  );
};
```
