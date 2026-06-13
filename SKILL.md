---
name: hras-training-video
description: 自动生成 HRAS 条线系统敏捷培训视频。使用 HyperFrames 将飞书文档中的操作步骤和截图转为 50 秒内的中英双语培训短视频。当用户提到 HRAS 培训视频、系统敏捷培训、操作培训视频、生成培训视频、安装/配置/设置 HRAS 视频环境时使用。需要飞书文档链接（含截图）和用户提供的系统名称、主题、期数。
version: 1.1.0
---

# HRAS 系统敏捷培训视频生成器

## 概述

根据飞书文档中的操作步骤和截图，自动生成 50 秒以内的中英双语培训短视频。使用 HyperFrames 框架（HTML + GSAP → MP4），风格参考飞书/Apple 官方产品介绍：现代简约、深色背景、流畅动画。

## 工作流

### Setup Mode（首次安装引导）

当用户提到以下关键词时，进入 **setup-only 模式**（只做环境准备，不生成视频）：
- "安装 HRAS 视频 skill"
- "配置 HRAS 视频环境"
- "设置培训视频工具"
- "check HRAS video setup"

**Setup-only 模式流程**：执行 Step 0 全部检测 + 自动安装 → 向用户报告环境状态 → 结束。不进入 Step 1 及后续步骤。

如果用户说"生成/制作 HRAS 培训视频"等正常触发词 → 先跑 Step 0（确保环境就绪），再进入完整工作流。

### Step 0: 环境检测 & 自动安装（每次执行必须先跑）

**此步骤在收集用户输入之前执行，确保运行环境就绪。agent 应主动安装缺失依赖，而不仅仅是提示用户。**

依次检测 5 项依赖，能自动装的直接装：

#### 检测 1: Node.js >= 22

```bash
node --version
```
- ✅ 存在且版本 >= 22 → 通过
- ❌ 不存在或版本 < 22 → 自动安装：
  ```bash
  brew install node
  ```
  安装后重新验证 `node --version`
- 如果 brew 也不存在 → 告知用户需先安装 Homebrew（`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`）

#### 检测 2: FFmpeg

```bash
ffmpeg -version 2>&1 | head -1
```
- ✅ 存在 → 通过
- ❌ 不存在 → 自动安装：
  ```bash
  brew install ffmpeg
  ```
  安装后重新验证

#### 检测 3: HyperFrames 渲染引擎

```bash
npx --yes hyperframes@latest doctor 2>&1
```
- 此命令会自动下载 HyperFrames 包并运行环境检查
- Chrome 缺失 → HyperFrames 会自动下载捆绑版（约 300MB，首次较慢）
- 只需确认命令最终输出无 critical error 即可

#### 检测 4: lark-cli（飞书文档读取）

```bash
which lark-cli 2>&1
```
- ✅ 存在 → 继续验证授权：
  ```bash
  lark-cli auth status 2>&1
  ```
  - 如果显示已授权 → 通过
  - 如果显示未授权 → 引导用户在对话中发送一个飞书文档链接，触发授权流程
- ❌ 不存在 → 告知用户：lark-cli 是 QoderWork 的飞书集成工具，通常在 QoderWork 中使用任意 lark-* skill 时会自动安装。请在对话中发送一个飞书文档链接来触发初始化。

#### 检测 5: lark-doc skill 文件

```bash
ls ~/.qoderworkcn/skills/lark-doc/SKILL.md 2>&1
```
- ✅ 存在 → 通过
- ❌ 不存在 → 告知用户需要安装 `lark-doc` skill：在 QoderWork Skill 市场中搜索安装，或请同事分享该 skill 文件夹放到 `~/.qoderworkcn/skills/lark-doc/`

#### 汇总报告

检测全部完成后，向用户输出环境状态摘要：

```
环境检测完成：
  ✓ Node.js v22.x.x
  ✓ FFmpeg x.x
  ✓ HyperFrames x.x.x
  ✓ lark-cli (已授权)
  ✓ lark-doc skill (已安装)
环境就绪，可以开始生成视频。
```

如有 ❌ 项，列出具体问题和修复建议，**不要跳过直接进入视频制作**。

**setup-only 模式下**：输出报告后结束，不继续后续步骤。
**正常模式下**：输出报告后继续 Step 1。

### Step 1: 收集用户输入（必须使用 AskUserQuestion）

**每次执行 skill 时，第一步就是用 AskUserQuestion 工具向用户收集以下信息。** 即使对话上下文中可能已有部分信息，也要主动确认。

需要收集的 5 个参数：

| 参数 | 说明 | 示例 |
|------|------|------|
| 系统名称（中英文） | 系统全称 | 供应商协同平台 / Supplier Collaboration Platform |
| 本期主题（中英文） | 本期培训的核心内容 | 创建员工 / Create Employees |
| 期数 | 第几期 | 第1期 |
| 飞书文档链接 | 含操作截图的文档 URL | https://ztn.feishu.cn/wiki/xxx |
| 目标章节 | 文档中要提取的具体章节 | 1.2 创建员工 |

**AskUserQuestion 示例**（可合并为 1-2 个问题）：
- Q1: "请提供系统名称（中英文）、本期主题（中英文）和期数"（Other 让用户自由输入）
- Q2: "请提供飞书文档链接和目标章节"（Other 让用户自由输入）

**唯一例外**：如果用户在触发 skill 时的消息中已经明确给出了全部 5 个参数，可跳过 AskUserQuestion 直接进入 Step 2。

### Step 2: 读取飞书文档

1. 先读取 `lark-doc` 的 fetch 参考（`~/.qoderworkcn/skills/lark-doc/references/lark-doc-fetch.md`）
2. 用 `outline` 获取文档目录，定位目标章节的 block-id
3. 用 `section --start-block-id <id> --detail with-ids` 读取章节全文
4. 提取所有 `<img>` 的 `href` URL 和 `alt` 描述

### Step 3: 下载截图素材

```bash
# 按顺序下载，命名格式：01_描述.png, 02_描述.png ...
curl -sL -o 01_login.png "<img_href_url>"
```

下载后逐一 Read 查看图片内容，识别：
- 图片对应的操作步骤
- **是否有红框/红箭头标注**（关键操作位置）
- 图片中的关键 UI 元素（按钮、菜单、输入框）

### Step 4: 规划分镜脚本

根据截图数量和内容规划分镜。总时长 **≤ 50 秒**（上限，不要求必须 50s），结构固定：

| 场景 | 时长 | 内容 |
|------|------|------|
| 封面（Title Card） | 固定 4.5s | 系统名称 + 主题 + 期数 |
| 操作步骤 × N | **按文字量动态计算** | 左文右图，中英双语 |
| 结尾（Closing） | 固定 5.5s | 固定结束语 |

#### 阅读时长计算公式

每个步骤的展示时长根据其文字内容自动计算，确保观众有足够时间阅读：

```
raw_read_time = cn_chars × 0.3 + en_words × 0.25 + 3.0

其中：
  cn_chars  = 该步骤所有中文文字（标题+描述）的字符数
  en_words  = 该步骤所有英文文字（标题+描述）的单词数
  3.0       = 动画入场 + 视觉消化基础时间（秒）
```

**约束条件**：
- 每步最短 5 秒（即使文字很少，也要留足动画展示时间）
- 每步最长 10 秒（避免单步过长导致节奏拖沓）
- 转场：每段 0.8s（重叠在步骤时长内，不额外计时）

**50 秒封顶规则**：
1. 先按公式计算各步 raw_read_time，clamp 到 [5, 10]
2. 计算 total = 4.5 + Σ(step_dur) + 5.5
3. 如果 total > 50：按比例压缩各步时长（等比缩放），每步不低于 5s
4. 如果 total ≤ 50：保持原始计算值，视频总时长 < 50s（完全允许）

**示例**（5 步，每步约 30 字中文 + 15 词英文）：
```
raw = 30 × 0.3 + 15 × 0.25 + 3.0 = 9 + 3.75 + 3 = 15.75 → clamp → 10s
total = 4.5 + 5×10 + 5.5 = 60s > 50 → 压缩: 每步 = 40/5 = 8s
```

**示例**（3 步，每步约 15 字中文 + 8 词英文）：
```
raw = 15 × 0.3 + 8 × 0.25 + 3.0 = 4.5 + 2 + 3 = 9.5s → clamp → 9.5s
total = 4.5 + 3×9.5 + 5.5 = 38.5s < 50 → 保持 38.5s ✓
```

### Step 5: 生成 HTML Composition

参照 [template-reference.md](./template-reference.md) 中的完整模板代码生成 `index.html`。

**核心规则**：

1. **封面三行布局**（居中对齐）：
   - 第一行：系统名称（中文），font-size: 48px，color: #94a3b8
   - 第二行：本期主题（中文），font-size: 80px，color: #fff，font-weight: 700（最大字体）
   - 第三行：HRAS系统敏捷培训 第X期，font-size: 24px，color: #60a5fa
   - 底部装饰：英文主题名 badge

2. **所有步骤场景统一左文右图**（不切换方向）：
   - 左侧文字区：`flex: 0 0 520px`
   - 右侧截图区：`flex: 1`，高度 720px，圆角 16px
   - 步骤标签：`Step 0X` 蓝色胶囊
   - 中文标题 + 英文标题（小一号）+ 中文描述 + 英文描述

3. **中英双语格式**：
   - 标题：中文（56px 白色）+ 英文（28px 灰色 #64748b）
   - 描述：中文（22px 灰色 #94a3b8）+ 英文（18px 浅灰 #64748b）

4. **关键操作标注处理**（红框/红箭头）：
   - 如果截图包含红框或红箭头标注，该截图展示时需要额外添加「关键帧高亮」动画
   - 具体做法：截图入场后，延迟 1.5s 添加一次轻微缩放脉冲（scale 1.0 → 1.03 → 1.0），持续 0.8s
   - 同时在截图上方显示一个 "Key Step" 提示标签，从右侧滑入
   - 详见 [template-reference.md](./template-reference.md) 中的 `keyframe-highlight` 代码段

5. **结尾固定文案**：
   - 主标题：本期结束
   - 副标题：This episode is now complete
   - 描述：有任何心愿选题、优化建议，欢迎反馈！
   - 英文：Got topic ideas or suggestions? We'd love your feedback!

### Step 6: 初始化项目 & 编写代码

```bash
# 初始化 HyperFrames 项目
npx hyperframes init hras-training-ep<期数> --example blank --non-interactive

# 复制截图到项目目录
cp /path/to/images/*.png /path/to/hras-training-ep<N>/
```

然后将生成的 HTML 写入 `index.html`。

### Step 7: 验证 & 渲染

```bash
cd hras-training-ep<N>

# Lint 检查（修复所有 error，warning 中 gsap_exit_missing_hard_kill 必须修复）
npx hyperframes lint

# 渲染
npx hyperframes render --quality high --output renders/hras-ep<N>.mp4
```

修复 lint 中的 `gsap_exit_missing_hard_kill` 警告：在每个场景退出动画之后添加 `tl.set` 硬杀帧。

### Step 8: 交付

将 MP4 复制到输出目录并提供文件链接。

## GSAP 动画规范

### 入场动画（每个场景）
- 步骤标签：`x: -30, opacity: 0` → `power3.out`
- 中文标题：`y: 40, opacity: 0` → `power3.out`
- 英文标题：`y: 30, opacity: 0` → `power2.out`
- 中文描述：`y: 30, opacity: 0` → `power2.out`，延迟 +0.2s
- 英文描述：`y: 20, opacity: 0` → `power2.out`，延迟 +0.1s
- 截图：`x: 60, opacity: 0, scale: 0.95` → `expo.out`

### 退场动画（每个场景，非结尾）
- 文字元素：`opacity: 0, y: -20`，stagger 0.05s
- 截图：`opacity: 0, x: 30`
- 退出完成后添加 `tl.set` 硬杀

### 转场
- 使用全屏黑色遮罩淡入淡出，duration 各 0.4s
- z-index 必须高于所有场景

### 关键帧高亮（有红框/红箭头的截图）
- 截图入场 1.5s 后，执行脉冲缩放
- 同时滑入 "Key Step" 标签

## 设计风格

| 属性 | 值 |
|------|-----|
| 背景色 | #0a0a0a（近黑） |
| 主标题色 | #ffffff |
| 中文描述色 | #94a3b8 |
| 英文副标题色 | #64748b |
| 强调色 | #3b82f6（蓝） |
| 步骤标签背景 | rgba(59,130,246,0.15) |
| 字体 | Inter, Helvetica Neue, Arial, sans-serif |
| 分辨率 | 1920x1080 |
| 帧率 | 30fps（standard）/ 60fps（high） |

## 注意事项

- 所有 GSAP timeline 必须 `{ paused: true }` 并注册到 `window.__timelines`
- 不使用 `Math.random()`、`Date.now()` 等非确定性逻辑
- 不使用 `repeat: -1`，计算精确重复次数
- 同步构建 timeline，不用 async/await
- 不直接 animate video 元素，用 wrapper div
- 最后一帧可做 fade out，其他场景只用转场遮罩退出

## 环境依赖说明（给复用者）

本 skill 依赖以下运行时环境，Step 0 会自动检测和安装：

| 依赖 | 最低版本 | 用途 | 安装方式 |
|------|---------|------|---------|
| Node.js | >= 22 | HyperFrames 运行时 | `brew install node` |
| FFmpeg | 任意 | 视频编码（帧→MP4） | `brew install ffmpeg` |
| HyperFrames | latest (npx) | HTML→视频渲染引擎 | npx 自动下载 |
| Chrome | 捆绑版 | 无头浏览器逐帧截图 | `npx hyperframes browser` 自动下载 |

**首次运行**时 `npx hyperframes` 会下载约 300MB（含 Chrome + 渲染依赖），后续运行会使用缓存。

**跨平台**：macOS 和 Linux 均支持。Windows 需要 WSL2 环境。
