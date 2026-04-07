# github-explainer-remotion

用 **React + Remotion** 把一个 GitHub 开源仓库的“项目分析报告”渲染成 **动态讲解视频（MP4）** 的工作流与代码模板集合。

这个仓库当前包含：
- **Skill 工作流定义**：`SKILL.md`（从抓取仓库 → 提取结构化数据 → 动态规划场景 → 生成 Remotion 项目 → 渲染 → 可选上传交付）
- **Remotion 场景代码模板（可运行）**：`references/scene-templates.md`
- **Remotion API 速查**：`references/remotion-api-guide.md`
- **统一视觉/动效规范**：`references/animation-styles.md`

> 注意：本仓库目前 **不是一个可直接 `npm install` 运行的 Remotion 项目**（没有 `package.json`）。它更像“生成 Remotion 项目”的说明书 + 模板库。

---

## 适用场景

- 你希望输入一个 `https://github.com/<owner>/<repo>`，自动产出一段 **竖屏 720×1280、30fps** 的项目解读视频
- 你希望视频内容随信息丰富度自动变长：**内容越多 → 场景越多 → 视频越长**
- 你希望统一成 GitHub Dark 风格视觉、卡片式排版、清晰的节奏与进场动画

---

## 快速开始（阅读路线）

如果你只想最快上手，建议按下面顺序阅读：

1. `SKILL.md`  
   了解完整端到端流程与输入/输出约束（尤其是“结构化 JSON”与“动态场景规划”规则）。
2. `references/scene-templates.md`  
   直接拿走完整可运行的 Remotion 场景组件模板（含 `types.ts`、`Root.tsx`、`GitHubExplainerVideo.tsx` 生成规则、11 个场景）。
3. `references/remotion-api-guide.md`  
   写动画时随查 `interpolate` / `spring` / `Sequence` / `Audio` 等关键 API。
4. `references/animation-styles.md`  
   统一颜色、字体、间距、动效时序，避免“每个场景像不同项目拼起来”。

---

## 使用方式（概念版）

`SKILL.md` 定义的核心流程可以概括为：

- **输入**：一个 GitHub 仓库 URL（例如 `https://github.com/super1112/github-explainer-remotion`）
- **抓取**：主页信息、README、核心源码文件、依赖清单、示例/文档、（可选）Releases/Issues/Wiki
- **提炼**：生成一份面向非技术观众的结构化 JSON（含 `problem / architecture / coreFiles / quickstart / highlights / audience` 等）
- **规划**：按规则把 JSON 映射为一组场景（Intro / Tldr / Problem / Arch / Files / Quickstart / Highlights / Outro…），并计算 `durationInFrames`
- **生成**：创建一个临时 Remotion 项目目录（例如 `.ark/tmp/remotion-{repoName}-{timestamp}/`），写入 `src/` 下的 React 场景代码
- **（可选）TTS**：用 `edge-tts` 生成每幕旁白音频，并回填“每幕需要的帧数”
- **渲染**：Remotion CLI 输出 `out/video.mp4`

---

## 目录结构

```
.
├── README.md
├── SKILL.md
└── references/
    ├── animation-styles.md      # 统一视觉/动效规范（颜色、字体、间距、时序）
    ├── remotion-api-guide.md    # Remotion API 速查
    └── scene-templates.md       # 场景组件完整模板（可变幕数，支持分批）
```

---

## 关键约定（做对了会省很多时间）

- **结构化 JSON 是“唯一真实来源”**：先把内容组织成 JSON，再用 JSON 生成场景与代码，避免边写代码边改文案导致结构崩坏。
- **场景可变 + 分批**：`ConceptsScene`、`FilesScene`、`QuickstartScene`、`HighlightsScene` 等支持按数量分批生成多幕。
- **旁白 key 对齐**：`sceneNarrations` 的 key（如 `intro`、`files_0`）需要与 `public/audio/<key>.mp3` 对应（如果你启用 TTS）。
- **统一视觉**：颜色/字号/间距尽量只从 `animation-styles.md` 的规范和 `scene-templates.md` 的常量体系取值。

---

## 常见问题

### 1) 为什么仓库里没有 `package.json`？

因为这里提供的是 **“生成 Remotion 项目并渲染视频”的方法论与模板**。真正的 Remotion 项目会在流程里按需生成到临时目录（`SKILL.md` 里有示例目录结构与模板）。

### 2) 我想把它变成一个可直接运行的 Remotion 项目

可以。你可以用 `references/scene-templates.md` 里的 `package.json 模板` 与 `Root.tsx`/场景组件，直接落地一个标准 Remotion 项目骨架（后续我也可以帮你把这个仓库升级成“可直接 `npm install && npm run`”的工程结构）。

---

## 贡献

欢迎提交 PR，建议优先从这些方向完善：
- 增加更多场景风格（但保持视觉规范一致）
- 把模板抽成真正的可运行 Remotion starter 项目
- 增加示例 JSON / 示例渲染结果（含截图或短视频）

