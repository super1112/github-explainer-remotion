---
name: github-explainer-remotion
description: 用 React + Remotion 框架将 GitHub 开源项目分析报告渲染成动态讲解视频（MP4）。当用户说"用 Remotion 制作视频解读"、"React 视频解读"、"帮我做一个动态视频讲解 GitHub 项目"、"用代码生成视频解读"、"Remotion 视频"时触发。输入一个 GitHub 项目 URL，自动完成：项目深度分析 → 结构化数据提取 → 动态场景规划 → 生成可变幕数 React 动画代码 → Remotion 渲染 MP4 → oss 上传交付。视频幕数由内容丰富度决定，内容越多视频越长。
---

# GitHub 项目 Remotion 动态视频解读

用 React 组件写动画，用代码定义每一帧，用 Remotion CLI 渲染成 MP4。
**视频长度随内容动态变化：内容越丰富，幕数越多，视频越长。**

---

## 工作流程

### 第一步：深度抓取 GitHub 项目内容

从用户输入中识别 `https://github.com/<owner>/<repo>`，使用 WebFetch 或 browser-use Skill 依次获取以下内容（必须尽量获取完整，不能只看 README 标题）：

**必须获取：**
1. **项目主页** — Star 数、Fork 数、语言、描述、Topics
2. **README 完整内容** — 逐字阅读，不要摘要跳读
3. **核心源码文件** — 寻找最能体现项目逻辑的 2-3 个文件（≤300行），例如：
   - Python: `main.py`、`__init__.py`、核心模块
   - JS/TS: `index.ts`、`src/` 下的主文件
   - 配置类: `SKILL.md`、`prompts/` 下的文件
4. **依赖清单** — `requirements.txt`、`package.json`、`pyproject.toml` 等
5. **示例/文档目录** — `docs/`、`examples/`、`demo/` 下的关键文件

**尽量获取（有则取）：**
6. **CHANGELOG / RELEASES** — 了解项目演进
7. **Issues 标题列表** — 了解用户痛点和常见问题
8. **讨论或 Wiki** — 有时比 README 更详细

> 若 WebFetch 受限，改用 browser-use Skill 抓取。

---

### 第二步：提取结构化数据（大白话优先）

**核心原则：写给完全不懂技术的人看。** 每个文字字段都要用：
- 初中生能读懂的语言（不用技术黑话）
- 生活化类比（"就像 X 一样"）
- 具体、可感知的描述（避免"提供了一个强大的解决方案"这类废话）

提取如下 JSON（字段说明见注释）：

```json
{
  "repoName": "项目名（原始名称）",
  "tagline": "一句话（≤25字），不要用技术词，普通人一看就懂",
  "stars": "2.9k",
  "language": "Python",

  "tldr": "用3句话，像给邻居介绍一样解释这个项目：①它是个什么东西 ②解决了什么烦恼 ③怎么用它。每句≤25字，合计≤80字。",

  "background": "项目诞生的故事/动机（可选，有趣故事优先填写，≤80字）",

  "problem": {
    "description": "核心痛点（≤60字）。用具体场景描述，如'每次想联系前任，却不知道说什么'，不要抽象",
    "before": "没有这个工具前，用户会……（≤40字）",
    "after": "用了这个工具后，可以……（≤40字）",
    "analogy": "一个生活中的比喻让人一秒懂，如'就像是给前任做了一个数字克隆'（可选）"
  },

  "keyConcepts": [
    {
      "term": "技术术语（如：Skill / Claude Code / 聊天记录解析）",
      "plain": "大白话解释（≤30字），用'就是'开头最好",
      "emoji": "🔧"
    }
  ],

  "architecture": {
    "analogy": "整体工作方式的生活类比（可选，如'像一个厨师：输入食材→加工→上菜'）",
    "layers": [
      "第1层：用户输入什么（具体说明，≤25字）",
      "第2层：系统如何处理（用大白话，≤25字）",
      "第3层：最终输出什么（具体成果，≤25字）"
    ],
    "coreComponent": "最核心的文件或目录名"
  },

  "coreFiles": [
    {
      "name": "文件名或目录名",
      "desc": "这个文件/目录做什么（≤25字，用动词开头，如'解析用户上传的微信聊天记录'）",
      "why": "为什么重要（≤25字，如'没有它就无法识别前任的说话风格'）（可选）"
    }
  ],

  "quickstart": [
    {
      "step": "操作步骤（≤25字，如'把项目克隆到 .claude/skills/ 目录下'）",
      "detail": "补充说明（≤30字，如'在 git 仓库根目录执行'）（可选）",
      "command": "具体命令（如 git clone ... 或 /create-ex）（可选）"
    }
  ],

  "useCases": [
    {
      "scene": "使用场景名（≤15字，如'深夜想念时'）",
      "example": "具体怎么用的案例（≤50字，写一个真实对话或操作示例）"
    }
  ],

  "highlights": [
    {
      "title": "亮点标题（≤15字）",
      "detail": "展开解释（≤40字，为什么这个亮点值得注意）",
      "emoji": "💡"
    }
  ],

  "audience": [
    {
      "group": "适合的人群（≤12字，如'有 Claude Code 的人'）",
      "fit": "strong",
      "reason": "为什么合适（≤20字，如'零门槛，装上即用'）（可选）"
    }
  ],

  "repoUrl": "https://github.com/owner/repo",

  "sceneNarrations": {
    "intro":        "中文解说（30-60字）：热情开场，介绍项目名称，一句话点出核心价值，结尾引发好奇心",
    "tldr":         "中文解说（30-60字）：用最接地气的语言重述项目本质，让从未听说过的人瞬间听懂",
    "context":      "（仅 background 非空时填写）中文解说（30-60字）：讲项目诞生背后的故事，制造情感共鸣",
    "problem":      "中文解说（30-60字）：描述痛点场景，用'你有没有遇到过…'的方式引发共鸣，再点出解法",
    "concepts_0":   "（仅 keyConcepts 非空时填写）中文解说（30-60字）：介绍第1-2个术语，举生活类比，让人一秒懂",
    "concepts_1":   "（keyConcepts ≥3时）中文解说（30-60字）：介绍第3-4个术语",
    "arch":         "中文解说（30-60字）：用类比描述整体工作流程，如'就像流水线一样……'",
    "files_0":      "中文解说（30-60字）：介绍核心文件存在的意义，不要只念文件名",
    "files_1":      "（coreFiles ≥5时）中文解说（30-60字）：更多文件说明",
    "quickstart_0": "中文解说（30-60字）：引导观众动手，语气积极，'跟我来，几步搞定……'",
    "quickstart_1": "（quickstart ≥4步时）中文解说（30-60字）：后续步骤引导",
    "useCases":     "（仅 useCases 非空时填写）中文解说（30-60字）：真实场景代入，让观众感受到'这正是我需要的'",
    "highlights_0": "中文解说（30-60字）：点评最出彩的1-3个亮点，用'最厉害的是……'、'这个设计真的绝'等句式",
    "highlights_1": "（highlights ≥4时）中文解说（30-60字）：更多亮点点评",
    "outro":        "中文解说（30-60字）：总结项目价值，推荐行动，'快去试试吧！链接在视频简介！'"
  }
}
```

**字段数量要求：**
- `keyConcepts`：有专业术语则填 2-4 个，简单项目可为空数组
- `coreFiles`：3-8 个，覆盖主要功能模块
- `quickstart`：2-5 步，每步清晰可执行
- `useCases`：0-3 个，有生动案例则填写
- `highlights`：3-6 个，突出真正值得称道的亮点
- `audience`：2-4 个人群

**解说词写作规范（`sceneNarrations`）：**
- **语气**：热情活泼，像朋友给朋友推荐好东西，不要念稿子
- **字数**：每条 30-60 个中文字符（TTS 约 6-12 秒），不超过 60 字
- **不要**：照搬屏幕上已有的文字；堆砌技术词汇；用"本项目提供了"等官方口吻
- **要**：补充屏幕没有的信息（为什么重要、使用感受、类比）；引导词如"你知道吗"、"想象一下"、"最厉害的是"
- `sceneNarrations` 中**只填实际会出现的场景的 key**（无 background 则不填 context；无 useCases 则不填 useCases）

---

### 第三步：动态规划场景并生成 Remotion 代码

#### 3.1 计算场景计划

根据第二步提取的数据，按以下规则确定场景列表和帧数：

| 场景类型 | 触发条件 | 每个持续帧数 | 说明 |
|---------|---------|------------|------|
| `Intro` | 始终 | 70f | 封面：项目名 + tagline + stats |
| `TldrScene` | 始终 | 60f | 大白话：tldr 3句话动画展示 |
| `ContextScene` | background 非空 | 55f | 背景故事 |
| `ProblemScene` | 始终 | 65f | 痛点：before/after + analogy |
| `ConceptsScene` | keyConcepts 非空，每2个一幕 | 60f/幕 | 术语解释 |
| `ArchScene` | 始终 | 50 + layers.length×10f | 架构层级 |
| `FilesScene` | 每4个文件一幕 | 55f/幕 | 核心文件解读 |
| `QuickstartScene` | 每3步一幕 | 60f/幕 | 上手步骤 |
| `UseCasesScene` | useCases 非空 | 65f | 使用场景案例 |
| `HighlightsScene` | 每3个亮点一幕 | 50f/幕 | 项目亮点 |
| `OutroScene` | 始终 | 20f | 适合人群 + 链接 |

**计算 totalFrames：** 将所有场景持续帧数相加。
**估算视频时长：** totalFrames / 30（秒）

> 示例：10 个场景 × 平均 58f = 580f ≈ 19 秒的视频

#### 3.2 生成 Remotion 项目

在 `.ark/tmp/remotion-{repoName}-{timestamp}/` 下创建完整 Remotion 项目：

```
remotion-{repoName}-{timestamp}/
├── package.json
├── tsconfig.json
├── public/
│   └── audio/              ← TTS 音频文件（Step 3.5 生成）
│       ├── intro.mp3
│       ├── tldr.mp3
│       └── ...（每个场景一个 mp3）
├── src/
│   ├── index.ts
│   ├── types.ts            ← 包含 VideoProps 接口（含 sceneNarrations）
│   ├── Root.tsx            ← 使用计算出的 totalFrames
│   ├── GitHubExplainerVideo.tsx  ← 根据场景计划动态生成
│   └── scenes/
│       ├── NarrationBar.tsx     ← 新增：底部字幕条组件
│       ├── SceneWithNarration.tsx ← 新增：带旁白的场景包装组件
│       ├── Intro.tsx
│       ├── TldrScene.tsx
│       ├── ContextScene.tsx
│       ├── ProblemScene.tsx
│       ├── ConceptsScene.tsx
│       ├── ArchScene.tsx
│       ├── FilesScene.tsx
│       ├── QuickstartScene.tsx
│       ├── UseCasesScene.tsx
│       ├── HighlightsScene.tsx
│       └── OutroScene.tsx
└── out/
```

**生成 `GitHubExplainerVideo.tsx` 时，直接内联计算好的帧号，例如：**

```tsx
// 假设场景计划为：Intro(d_intro) + Tldr(d_tldr) + Problem(d_problem) + Arch(d_arch) + Files(d_files) + Quickstart(d_qs) + Highlights(d_hl) + Outro(d_outro)
// 帧数由 Step 3.5 TTS 校准后填入
export const GitHubExplainerVideo: React.FC<VideoProps> = (props) => (
  <AbsoluteFill style={{ backgroundColor: COLORS.bgPrimary }}>
    <Sequence from={0}   durationInFrames={d_intro}   name="Intro">
      <SceneWithNarration narration={props.sceneNarrations?.intro} sceneKey="intro" duration={d_intro}>
        <Intro {...props} />
      </SceneWithNarration>
    </Sequence>
    <Sequence from={f1}  durationInFrames={d_tldr}    name="Tldr">
      <SceneWithNarration narration={props.sceneNarrations?.tldr} sceneKey="tldr" duration={d_tldr}>
        <TldrScene {...props} />
      </SceneWithNarration>
    </Sequence>
    {/* 条件场景、文件批次等按计划追加，每个 Sequence 都用 SceneWithNarration 包裹 */}
    <Sequence from={fN}  durationInFrames={d_outro}   name="Outro">
      <SceneWithNarration narration={props.sceneNarrations?.outro} sceneKey="outro" duration={d_outro}>
        <OutroScene {...props} />
      </SceneWithNarration>
    </Sequence>
  </AbsoluteFill>
);
```

**SceneWithNarration 包装规则：**
- 每个 `<Sequence>` 内层都用 `<SceneWithNarration narration={...} sceneKey="场景key" duration={场景帧数}>` 包裹
- `sceneKey` 必须与 `sceneNarrations` 的 key 以及 `public/audio/` 下的 mp3 文件名一致
- 若某场景无解说词（`narration` 为 undefined），SceneWithNarration 会静默跳过音频和字幕，不影响渲染

**Root.tsx 使用计算出的 totalFrames（不要硬写 270）：**

```tsx
<Composition
  id="GitHubExplainer"
  component={GitHubExplainerVideo}
  durationInFrames={totalFrames}  // 动态计算
  fps={30}
  width={720}
  height={1280}
  defaultProps={videoProps}
/>
```

各文件完整代码模板见 **references/scene-templates.md**。
动画 API 见 **references/remotion-api-guide.md**，颜色/间距见 **references/animation-styles.md**。

---

### 第三步半（3.5）：TTS 生成 + 帧数校准

在 Remotion 项目目录内运行以下 Python 脚本，生成每个场景的 MP3 音频并输出建议帧数：

```bash
cd .ark/tmp/remotion-{repoName}-{timestamp}/
mkdir -p public/audio
pip3 install mutagen -q 2>&1 | tail -1

python3 << 'PYEOF'
import asyncio, os
import edge_tts
from mutagen.mp3 import MP3

# === 从第二步生成的 sceneNarrations 填入 ===
narrations = {
  "intro":        "...",
  "tldr":         "...",
  # "context":    "...",   # 仅 background 非空时取消注释
  "problem":      "...",
  # "concepts_0": "...",   # 仅 keyConcepts 非空时
  "arch":         "...",
  "files_0":      "...",
  "quickstart_0": "...",
  # "useCases":   "...",   # 仅 useCases 非空时
  "highlights_0": "...",
  "outro":        "...",
}
# =============================================

VOICE = "zh-CN-XiaoxiaoNeural"
RATE  = "+10%"  # 适当加速，让语速自然不拖沓

async def gen(key, text):
    comm = edge_tts.Communicate(text, VOICE, rate=RATE)
    await comm.save(f"public/audio/{key}.mp3")

async def main():
    tasks = [gen(k, v) for k, v in narrations.items()]
    await asyncio.gather(*tasks)
    print("=== 音频时长 → 建议场景帧数（含 20 帧收尾余量）===")
    for key in narrations:
        audio  = MP3(f"public/audio/{key}.mp3")
        secs   = audio.info.length
        frames = int(secs * 30) + 20
        print(f"  {key:20s}  {secs:.1f}s  →  {frames}f")

asyncio.run(main())
PYEOF
```

**读取输出，用实际 TTS 帧数重写 GitHubExplainerVideo.tsx 和 Root.tsx：**
- 每个场景的 `durationInFrames` = max(该场景默认帧数, TTS 建议帧数)
- 重新累加所有帧数得到新的 `totalFrames`，更新 `Root.tsx` 中的 `TOTAL_FRAMES`
- **必须在写完 TSX 文件后再执行此步骤**，TTS 生成与代码写入顺序：
  1. 先写好所有 `.tsx` 文件（Step 3.6 / 第三步的最后）
  2. 再运行上方 Python 脚本生成音频
  3. 根据输出帧数更新 `GitHubExplainerVideo.tsx` 和 `Root.tsx` 中的帧数

> **注意：** edge-tts 需联网调用微软 TTS 服务。若网络不通，跳过此步骤，帧数使用 Step 3.1 默认值，并在渲染时加 `--muted`。

---

### 第四步：安装依赖并渲染

```bash
# 进入项目目录
cd .ark/tmp/remotion-{repoName}-{timestamp}/

# 安装依赖（使用国内镜像加速）
npm install --registry=https://registry.npmmirror.com 2>&1 | tail -10

# 渲染视频（音频从 public/audio/ 目录加载，Remotion 自带 Chrome + ffmpeg）
npx remotion render GitHubExplainer out/video.mp4 \
  --props='<第二步生成的JSON>' \
  --codec=h264 \
  --crf=23 \
  --concurrency=4 2>&1
# 注意：不加 --muted，让 TTS 音频和字幕都渲染进视频
# 若 Step 3.5 TTS 未成功，可加 --muted 降级为纯视觉版
```

> **注意：** 首次运行会自动下载 Chrome Headless Shell（~108MB），请耐心等待，无需手动安装 ffmpeg。
> 渲染耗时参考：500帧 concurrency=4 约 60-120 秒。

---

### 第五步：上传并交付

### 第五步：上传并交付

```bash
# 1. 检查文件大小
ls -lh .ark/tmp/remotion-{repoName}-{timestamp}/out/video.mp4

# === 核心修改部分：直接集成阿里云 OSS 上传逻辑 ===

# 2. 定义变量 (请根据你的实际情况修改这三个变量)
# 将 'amara' 替换为你的 Bucket 名称
BUCKET_NAME="amara"
# 将 'oss-cn-shanghai' 替换为你的 Region ID
REGION_ID="oss-cn-shanghai"
# 定义 OSS 上的存储目录 (例如：videos/)
OSS_DIR="videos/"

# 3. 获取文件名和本地路径
LOCAL_FILE=".ark/tmp/remotion-{repoName}-{timestamp}/out/video.mp4"
FILE_NAME=$(basename "$LOCAL_FILE")
OSS_PATH="oss://${BUCKET_NAME}/${OSS_DIR}${FILE_NAME}"

echo "正在上传视频到阿里云 OSS: ${OSS_PATH} ..."

# 4. 使用系统已配置的 ossutil 进行上传
# 我们不仅上传文件，还同时设置文件的 ACL 为公共读 (public-read)，确保生成的链接可直接访问。
ossutil cp "$LOCAL_FILE" "$OSS_PATH" --meta x-oss-object-acl:public-read 2>&1

# 5. 检查上传是否成功
if [ $? -ne 0 ]; then
  echo "❌ 视频上传到 OSS 失败，请检查 ossutil 配置或网络。"
  exit 1
fi

echo "✅ 视频上传成功！"

# 6. 生成并获取公网 URL
# 这里使用 Region ID 和 Bucket 名称动态拼写出公网链接
VIDEO_URL="https://${BUCKET_NAME}.${REGION_ID}.aliyuncs.com/${OSS_DIR}${FILE_NAME}"

```

交付格式：
```
🎬 Remotion 动态视频解读已生成！

▶️ 视频链接：<VIDEO_URL>

视频规格：720×1280 竖版 | 30fps | <N>秒 | h264
场景数：<N> 幕（含大白话/背景/术语等动态场景）
```

---

## 错误处理与降级

| 错误 | 判断方式 | 处理 |
|------|---------|------|
| npm install 失败 | exit code 非 0 | 已加 npmmirror，再失败则降级 |
| render 超时（>600s） | 进程未返回 | 终止，降级交付源码 zip |
| 输出文件 > 50MB | `ls -lh out/video.mp4` | 加 `--scale=0.5` 重新渲染（360×640） |
| WebFetch 全部失败 | 404/网络错误 | 换 browser-use Skill 重试 |
| Chrome 下载被中断 | "Getting Headless Shell" 后中断 | 重新运行渲染命令，会继续下载 |

**降级交付（源码 zip）：**
```bash
cd .ark/tmp/
zip -r remotion-{repoName}.zip remotion-{repoName}-{timestamp}/
dodo_cli bos upload --short-url remotion-{repoName}.zip
```

---

## 参考资料

- **Remotion API 速查**：见 references/remotion-api-guide.md
- **动态场景代码模板**：见 references/scene-templates.md（含新增场景 TldrScene / ContextScene / ConceptsScene / UseCasesScene）
- **动画设计规范**：见 references/animation-styles.md
