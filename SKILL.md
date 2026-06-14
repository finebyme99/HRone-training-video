---
name: hras-training-video
description: 自动生成 HRAS 条线系统敏捷培训视频。使用 HyperFrames 将飞书文档中的操作步骤和截图转为 60 秒内的中英双语培训短视频，支持中文或英文 AI 配音、可选背景音乐和转场音效。当用户提到 HRAS 培训视频、系统敏捷培训、操作培训视频、生成培训视频、安装/配置/设置 HRAS 视频环境时使用。需要飞书文档链接（含截图）和用户提供的系统名称、主题、期数。
version: 2.0.0
---

# HRAS 系统敏捷培训视频生成器

## 概述

根据飞书文档中的操作步骤和截图，自动生成 **60 秒以内**的中英双语培训短视频，支持中文或英文 AI 配音。使用 HyperFrames 框架（HTML + GSAP → MP4）+ edge-tts（免费 TTS）生成配音，截图**下载到本地**使用（飞书图片 URL 有效期仅几分钟），风格参考飞书/Apple 官方产品介绍：现代简约、深色背景、流畅动画。

---

## 核心规则

### 配音语言一致性（最高优先级）

**用户在 Step 1 选择的配音语言是最终决定，后续任何环节不得更改。**

- 用户选了中文配音 → 所有场景（封面、步骤、结尾）一律朗读中文文本，使用中文声音
- 用户选了英文配音 → 所有场景一律朗读英文文本，使用英文声音
- **禁止**因文本内容、场景类型或 agent 主观判断而切换语言
- **所有 TTS 文件必须在同一批次中生成**，使用相同的声音和语速参数，确保音色一致

### 封面配音规则

封面场景的配音**只朗读本期主题**（如"查看派遣需求"），不朗读系统名称、培训名称、期数等信息。封面的视觉元素保持不变（系统名称、主题、期数、英文 badge 全部显示）。

---

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

依次检测 6 项依赖，能自动装的直接装：

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

#### 检测 6: edge-tts（AI 配音引擎）

```bash
python3 -m edge_tts --version 2>&1
```
- ✅ 存在 → 通过
- ❌ 不存在 → 自动安装：
  ```bash
  pip3 install edge-tts
  ```
  安装后重新验证

> **注意**：macOS 上 `which edge-tts` 可能找不到（二进制不在 PATH 中）。始终使用 `python3 -m edge_tts` 作为调用方式。

#### 汇总报告

检测全部完成后，向用户输出环境状态摘要：

```
环境检测完成：
  ✓ Node.js v22.x.x
  ✓ FFmpeg x.x
  ✓ HyperFrames x.x.x
  ✓ lark-cli (已授权)
  ✓ lark-doc skill (已安装)
  ✓ edge-tts (已安装)
环境就绪，可以开始生成视频。
```

如有 ❌ 项，列出具体问题和修复建议，**不要跳过直接进入视频制作**。

**setup-only 模式下**：输出报告后结束，不继续后续步骤。
**正常模式下**：输出报告后继续 Step 1。

### Step 1: 收集用户输入（保姆级引导）

**环境就绪后，agent 必须先输出一份「准备清单」，再逐一收集信息。** 不要直接甩问题给用户，先让他们知道需要什么。

#### 第一步：输出准备清单

向用户展示以下内容（可直接复制此段）：

```
📋 生成视频前，请准备以下 8 项信息：

1️⃣ 系统名称（中英文）
   你要培训的系统叫什么。例如：
   · 中文：供应商协同平台
   · 英文：Supplier Collaboration Platform

2️⃣ 本期主题（中英文）
   这期视频教什么操作。例如：
   · 中文：创建员工
   · 英文：Create Employees

3️⃣ 期数
   这是系列培训的第几期。例如：第1期、第2期

4️⃣ 飞书文档链接
   包含操作步骤和截图的飞书云文档 URL。
   格式类似：https://ztn.feishu.cn/wiki/xxxxx
   ⚠️ 文档需要有权限访问，且包含截图

5️⃣ 目标章节
   文档中要提取的具体章节编号。例如：1.2 创建员工
   一段文档可以做多期视频，每期选不同章节

6️⃣ 配音语言
   视频的旁白用中文还是英文配音。
   · 中文配音：朗读中文描述文字
   · 英文配音：朗读英文描述文字

7️⃣ 背景音乐
   是否在视频中加入轻松的背景音乐。
   · 需要 / 不需要（推荐）

8️⃣ 转场音效
   是否在场景切换时加入音效。
   · 需要（推荐） / 不需要
```

#### 第二步：收集信息

展示清单后，使用 AskUserQuestion 工具收集（可合并为 1-2 个问题）：

- **Q1**：系统名称（中英文）+ 本期主题（中英文）+ 期数
  - 提供预设选项（如有常用值），Other 让用户自由输入
- **Q2**：飞书文档链接 + 目标章节 + 配音语言（中文/英文）+ 背景音乐（需要/不需要）+ 转场音效（需要/不需要）
  - 配音语言预设：中文配音、英文配音
  - 背景音乐预设：需要、不需要（推荐）
  - 转场音效预设：需要（推荐）、不需要

| 参数 | 说明 | 示例 |
|------|------|------|
| 系统名称（中英文） | 系统全称，用于视频封面第一行 | 供应商协同平台 / Supplier Collaboration Platform |
| 本期主题（中英文） | 本期培训的核心操作，用于封面最大字 | 创建员工 / Create Employees |
| 期数 | 系列培训的第几期 | 第1期 |
| 飞书文档链接 | 含操作截图的飞书云文档 URL | https://ztn.feishu.cn/wiki/xxx |
| 目标章节 | 文档中要提取的章节（可做系列） | 1.2 创建员工 |
| 配音语言 | 中文或英文旁白（**选定后不可更改**） | 中文配音 |
| 背景音乐 | 是否加轻松背景音乐 | 不需要 |
| 转场音效 | 是否加场景切换音效 | 需要 |

**唯一例外**：如果用户在触发 skill 时的消息中已经明确给出了全部 8 个参数，可跳过 AskUserQuestion 直接进入 Step 2。

#### 信息校验

收集完成后，向用户确认：

```
收到！我整理一下：
  · 系统：供应商协同平台 / Supplier Collaboration Platform
  · 主题：创建员工 / Create Employees
  · 期数：第1期
  · 文档：https://ztn.feishu.cn/wiki/xxxxx
  · 章节：1.2
  · 配音：中文
  · 背景音乐：不需要
  · 转场音效：需要

确认无误，开始生成视频 🎬
```

如有信息不完整或格式可疑（如链接不像飞书 URL），立即追问，不要硬猜。

### Step 2: 读取飞书文档

1. 先读取 `lark-doc` 的 fetch 参考（`~/.qoderworkcn/skills/lark-doc/references/lark-doc-fetch.md`）
2. 用 `outline` 获取文档目录，定位目标章节的 block-id
3. 用 `section --start-block-id <id> --detail with-ids` 读取章节全文
4. 提取所有 `<image token="...">` 的 token 值和上下文描述文字

### Step 3: 下载截图到本地

**飞书图片 authcode URL 有效期仅几分钟，必须下载到本地使用。**

```bash
# 在项目目录下创建 images 目录
mkdir -p images/

# 使用 lark-cli 逐个下载（token 来自文档中的 <image token="xxx"> 标签）
lark-cli docs +media-download --token {TOKEN_1} --output images/
lark-cli docs +media-download --token {TOKEN_2} --output images/
# ... 每个 token 下载一次
```

下载后将文件重命名为语义化名称（`step1.png`、`step2.png` ...），并整理对应表：

| 步骤 | 图片文件 | 原始 token | 内容描述 |
|------|---------|-----------|---------|
| Step 1 | images/step1.png | NPmDbfu6UosmeyxFuuecXS8Lnnf | 派遣管理菜单 |
| Step 2 | images/step2.png | SZEubZuY4ozOl9xTMkrcUVQznRf | 我的待办 |

然后逐一用 Read 工具查看图片内容，识别：
- 图片对应的操作步骤
- 图片中的关键 UI 元素（按钮、菜单、输入框）
- 是否有红框/红箭头标注

### Step 4: 规划分镜脚本

根据截图数量和内容规划分镜。总时长 **≤ 60 秒**（上限，不要求必须 60s），结构固定：

| 场景 | 时长 | 内容 |
|------|------|------|
| 封面（Title Card） | **≥ 3s**（TTS 驱动） | 系统名称 + 主题 + 期数（视觉全显示，配音只读主题） |
| 操作步骤 × N | **按 TTS 时长动态计算** | 左文右图，中英双语 |
| 结尾（Closing） | **TTS 驱动** | 固定结束语 |

#### TTS 驱动时长计算

场景时长由配音音频时长决定，而非文字阅读公式：

```
scene_duration = ceil(TTS_duration + 0.2)

其中：
  TTS_duration = 该场景 TTS 音频的实际秒数（ffprobe 测量）
  0.2          = 缓冲时间，避免音频顶满场景
  ceil         = 向上取整到整数秒
```

**封面特殊规则**：
- 封面 TTS 只读主题（如"查看派遣需求"），时长很短
- 封面场景最少 3 秒，给视觉动画留足时间
- `title_duration = max(3, ceil(TTS_title + 0.2))`

**约束条件**：
- 总时长不超过 60 秒
- 如果总时长超过 60 秒：需要精简步骤内容（减少文字），重新生成 TTS，直到满足要求

#### 时长计算流程

1. 为所有场景准备配音文本（见 Step 8.1）
2. 生成所有 TTS 音频文件（见 Step 8.2）
3. 用 `ffprobe` 测量每段音频时长
4. 按公式计算各场景时长
5. 排布时间轴

**示例**（ep3：4 步，中文配音）：
```
TTS_title  = 1.99s → scene = max(3, ceil(2.19)) = 3s
TTS_step1  = 7.80s → scene = ceil(8.0)  = 10s   ← 向上取整到 10
TTS_step2  = 7.94s → scene = ceil(8.14) = 10s
TTS_step3  = 7.68s → scene = ceil(7.88) = 10s
TTS_step4  = 7.03s → scene = ceil(7.23) = 11s   ← 给结尾留余量
TTS_close  = 5.62s → scene = ceil(5.82) = 11s
Total = 3 + 10 + 10 + 10 + 11 + 11 = 55s ≤ 60 ✓
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
   - 左侧文字区：`flex: 0 0 380px; padding-right: 40px`
   - 右侧截图区：`flex: 1; height: 920px; border-radius: 16px; background: #111`
   - 截图 `<img src="images/stepN.png">` 使用本地下载的图片
   - 截图样式：`object-fit: contain; object-position: center`（**不裁切**，完整展示）
   - 步骤标签：`Step 0X` 蓝色胶囊
   - 中文标题 + 英文标题（小一号）+ 中文描述 + 英文描述

3. **中英双语格式**：
   - 标题：中文（56px 白色）+ 英文（28px 灰色 #64748b）
   - 描述：中文（22px 灰色 #94a3b8）+ 英文（18px 浅灰 #64748b）

4. **结尾固定文案**：
   - 主标题：本期结束
   - 副标题：This episode is now complete
   - 描述：有任何心愿选题、优化建议，欢迎反馈！
   - 英文：Got topic ideas or suggestions? We'd love your feedback!

5. **转场遮罩**：
   - 每个场景间插入 0.1s 的黑色闪光遮罩
   - `data-track-index="1"`，`z-index: 20`
   - 位于场景结束前 0.1s（如场景 0~3s，转场在 2.9s）

### Step 6: 初始化项目 & 编写代码

```bash
# 初始化 HyperFrames 项目
npx hyperframes init hras-training-ep<期数> --example blank --non-interactive

# 将下载的图片复制到项目目录
cp -r images/ hras-training-ep<期数>/images/
```

然后将生成的 HTML 写入 `index.html`。所有 `<img src="images/stepN.png">` 使用本地图片路径。

### Step 7: 验证 & 渲染

```bash
cd hras-training-ep<N>

# Lint 检查（修复所有 error，warning 中 gsap_exit_missing_hard_kill 必须修复）
npx hyperframes lint

# 渲染（无声视频）
npx hyperframes render --quality high --output renders/hras-ep<N>-silent.mp4
```

修复 lint 中的 `gsap_exit_missing_hard_kill` 警告：在每个场景退出动画之后添加 `tl.set` 硬杀帧。

### Step 8: 生成配音 & 合并音视频

使用 edge-tts（微软免费 TTS 引擎）为每个场景生成配音音频，然后与视频合并。

#### 8.1 确定配音文本

根据用户在 Step 1 选择的配音语言，为每个场景准备朗读文本：

| 场景 | 中文配音文本 | 英文配音文本 |
|------|------------|------------|
| 封面 | **{本期主题}**（只读主题，不读系统名/培训名/期数） | **{Topic in English}**（同样只读主题） |
| 步骤 N | {中文标题}。{中文描述} | {English Title}. {English description} |
| 结尾 | 本期结束。有任何心愿选题、优化建议，欢迎反馈。 | This episode is now complete. Got topic ideas or suggestions? We'd love your feedback! |

**⚠️ 所有场景必须使用同一语言配音，不得混用中英文。**

#### 8.2 批量生成 TTS 音频（一次性生成所有场景）

**所有 TTS 文件必须在同一个批次中生成，使用完全相同的声音和语速参数，确保音色一致。**

```bash
# 中文配音（voice: zh-CN-XiaoxiaoNeural, rate: +5%）
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{封面朗读文本}" --write-media audio/tts_title.mp3
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{步骤1朗读文本}" --write-media audio/tts_step1.mp3
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{步骤2朗读文本}" --write-media audio/tts_step2.mp3
# ... 依次生成所有场景
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{结尾朗读文本}" --write-media audio/tts_close.mp3

# 英文配音（voice: en-US-AriaNeural, rate: +5%）
python3 -m edge_tts --voice en-US-AriaNeural --rate=+5% \
  --text "{title narration}" --write-media audio/tts_title.mp3
# ... 同理
```

#### 8.3 测量 TTS 时长 & 计算场景时长

```bash
# 获取每段音频时长
ffprobe -v error -show_entries format=duration -of csv=p=0 audio/tts_title.mp3
ffprobe -v error -show_entries format=duration -of csv=p=0 audio/tts_step1.mp3
# ... 所有场景

# 按公式计算场景时长：scene_dur = ceil(TTS_dur + 0.2)
# 封面：title_dur = max(3, ceil(TTS_title + 0.2))
```

#### 8.4 构建完整配音轨（WAV 中间格式）

**必须使用 WAV 作为音频中间格式。** AAC 编码会产生不精确的时长（apad + AAC 导致时长偏差），WAV 可以保证精确的场景对齐。

```bash
# 将每段 TTS 填充到对应场景时长（WAV 格式）
ffmpeg -i audio/tts_title.mp3 -af "apad=whole_dur={title_dur}" \
  -ar 24000 -ac 1 audio/pad_title.wav

ffmpeg -i audio/tts_step1.mp3 -af "apad=whole_dur={step1_dur}" \
  -ar 24000 -ac 1 audio/pad_step1.wav

# ... 对所有场景重复

# 写入 concat 列表
cat > audio/concat_list.txt << 'EOF'
file 'pad_title.wav'
file 'pad_step1.wav'
file 'pad_step2.wav'
file 'pad_step3.wav'
file 'pad_step4.wav'
file 'pad_close.wav'
EOF

# 拼接为完整配音轨（WAV）
ffmpeg -f concat -safe 0 -i audio/concat_list.txt \
  -ar 24000 -ac 1 audio/voiceover.wav
```

#### 8.5 生成转场音效（如用户选择"需要"）

推荐使用 Mixkit 免费商用音效素材（现代风格，iOS/Slack 通知风），而非 FFmpeg 合成白噪声。

```bash
# 下载 Mixkit 音效（例：ID 2364，现代通知音）
curl -L -o audio/sfx_raw.mp3 \
  "https://assets.mixkit.co/active_storage/sfx/2364/2364-preview.mp3"

# 转换为 WAV，调整音量
ffmpeg -i audio/sfx_raw.mp3 -af "volume=1.1" \
  -ar 24000 -ac 1 audio/sfx_processed.wav
```

其他推荐 Mixkit 音效 ID（在 mixkit.co/free-sound-effects/notification/ 浏览）：
- 2364：现代通知音（推荐）
- 2358：清脆提示音
- 2361：柔和过渡音

#### 8.6 生成背景音乐（如用户选择"需要"）

使用 FFmpeg 合成温暖的 ambient pad：

```bash
TOTAL_DUR=$(ffprobe -v error -show_entries format=duration -of csv=p=0 renders/hras-ep<N>-silent.mp4)

ffmpeg -f lavfi -i "sine=frequency=261.63:duration=${TOTAL_DUR}" \
       -f lavfi -i "sine=frequency=329.63:duration=${TOTAL_DUR}" \
       -f lavfi -i "sine=frequency=392.00:duration=${TOTAL_DUR}" \
       -filter_complex "\
         [0]volume=0.08[a];[1]volume=0.06[b];[2]volume=0.05[c];\
         [a][b][c]amix=inputs=3:duration=longest,\
         chorus=0.7:0.9:55:0.4:0.25:2,\
         afade=t=in:st=0:d=2,\
         afade=t=out:st=$(echo "$TOTAL_DUR - 3" | bc):d=3\
       " -ar 24000 -ac 1 audio/bgm.wav
```

#### 8.7 混合音频 & 合并视频

**场景 A：仅配音**（无背景音乐、无转场音效）

```bash
ffmpeg -i renders/hras-ep<N>-silent.mp4 -i audio/voiceover.wav \
  -c:v copy -c:a aac -shortest \
  -map 0:v:0 -map 1:a:0 \
  renders/hras-ep<N>.mp4
```

**场景 B：配音 + 转场音效**

```bash
# 计算每个转场的时间点（ms）
# T_trans_i = 各场景结束时间 - 0.1s，转位到 ms

ffmpeg -i audio/voiceover.wav \
       -i audio/sfx_processed.wav -i audio/sfx_processed.wav ... \
  -filter_complex "\
       [0]apad[vo];\
       [1]adelay={T1_ms}|{T1_ms},volume=0.3[s1];\
       [2]adelay={T2_ms}|{T2_ms},volume=0.3[s2];\
       [vo][s1][s2]amix=inputs=3:duration=first[aout]" \
  -map "[aout]" -t ${TOTAL_DUR} audio/final_audio.wav

# 合并音视频
ffmpeg -i renders/hras-ep<N>-silent.mp4 -i audio/final_audio.wav \
  -c:v copy -c:a aac -shortest \
  -map 0:v:0 -map 1:a:0 \
  renders/hras-ep<N>.mp4
```

**场景 C：配音 + 背景音乐**

```bash
ffmpeg -i renders/hras-ep<N>-silent.mp4 \
       -i audio/voiceover.wav \
       -i audio/bgm.wav \
  -filter_complex "[1:a]volume=1.0[vo];[2:a]volume=0.15[bg];[vo][bg]amix=inputs=2:duration=shortest[aout]" \
  -map 0:v:0 -map "[aout]" \
  -c:v copy -c:a aac -shortest \
  renders/hras-ep<N>.mp4
```

**场景 D：配音 + 背景音乐 + 转场音效（全部启用）**

```bash
ffmpeg -i renders/hras-ep<N>-silent.mp4 \
       -i audio/voiceover.wav \
       -i audio/bgm.wav \
       -i audio/sfx_processed.wav -i audio/sfx_processed.wav ... \
  -filter_complex "\
       [1:a]volume=1.0[vo];\
       [2:a]volume=0.15[bg];\
       [3]adelay={T1_ms}|{T1_ms},volume=0.3[s1];\
       [4]adelay={T2_ms}|{T2_ms},volume=0.3[s2];\
       [vo][bg][s1][s2]amix=inputs=4:duration=first[aout]" \
  -map 0:v:0 -map "[aout]" \
  -c:v copy -c:a aac -shortest \
  renders/hras-ep<N>.mp4
```

**音量参考**：配音 1.0（基准）、背景音乐 0.15（衬托）、转场音效 0.3（点缀）。

**⚠️ FFmpeg amix 注意事项**：
- `duration=first`：以第一个输入（配音轨）的时长为准
- `duration=shortest`：以最短输入为准（SFX 短时会导致截断！）
- 混合配音+SFX 时必须用 `duration=first` 或 `duration=longest`
- 最终合并音视频时 `-shortest` 确保音视频对齐

### Step 9: 交付

将带配音的 MP4 复制到输出目录并提供文件链接。

---

## GSAP 动画规范

### 入场动画（每个场景）

**封面**（0 ~ title_dur）：
```javascript
tl.from("#title-sys",   { y: 30, opacity: 0, duration: 0.5, ease: "power3.out" }, 0.2);
tl.from("#title-topic", { y: 50, opacity: 0, duration: 0.6, ease: "power3.out" }, 0.4);
tl.from("#title-ep",    { y: 25, opacity: 0, duration: 0.4, ease: "power2.out" }, 0.7);
tl.from("#title-badge", { y: 20, opacity: 0, duration: 0.4, ease: "power2.out" }, 1.0);
```

**步骤场景**（scene_start ~ scene_end）：
```javascript
const T = scene_start;  // 场景起始时间
tl.from("#stepN-badge",    { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, T + 0.2);
tl.from("#stepN-title-cn", { y: 40, opacity: 0, duration: 0.7, ease: "power3.out" }, T + 0.4);
tl.from("#stepN-title-en", { y: 30, opacity: 0, duration: 0.6, ease: "power2.out" }, T + 0.6);
tl.from("#stepN-desc-cn",  { y: 25, opacity: 0, duration: 0.5, ease: "power2.out" }, T + 0.8);
tl.from("#stepN-desc-en",  { y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, T + 1.0);
tl.from("#stepN-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, T + 0.5);
```

**结尾**（close_start ~ close_end）：
```javascript
const CS = close_start;
tl.from("#close-icon",       { scale: 0.5, opacity: 0, duration: 0.7, ease: "back.out(1.7)" }, CS + 0.3);
tl.from("#close-title-cn",   { y: 40, opacity: 0, duration: 0.7, ease: "power3.out" }, CS + 0.6);
tl.from("#close-title-en",   { y: 25, opacity: 0, duration: 0.5, ease: "power2.out" }, CS + 0.9);
tl.from("#close-feedback-cn",{ y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, CS + 1.2);
tl.from("#close-feedback-en",{ y: 15, opacity: 0, duration: 0.4, ease: "power2.out" }, CS + 1.4);
tl.from("#close-line",       { scaleX: 0, opacity: 0, duration: 0.7, ease: "power2.out" }, CS + 1.6);
```

### 退场动画（每个场景，非结尾）

**封面退出**（在 title_dur - 0.5 处开始）：
```javascript
const TE = title_dur - 0.5;
tl.to("#title-sys",   { opacity: 0, y: -20, duration: 0.2, ease: "power2.in" }, TE);
tl.to("#title-topic", { opacity: 0, y: -30, duration: 0.2, ease: "power2.in" }, TE + 0.05);
tl.to("#title-ep",    { opacity: 0, y: -15, duration: 0.2, ease: "power2.in" }, TE + 0.1);
tl.to("#title-badge", { opacity: 0, y: -10, duration: 0.15, ease: "power2.in" }, TE + 0.15);
```

**步骤退出**（在 scene_end - 0.6 处开始）：
```javascript
const EX = scene_end - 0.6;
tl.to("#stepN-text > *",  { opacity: 0, y: -20, duration: 0.3, stagger: 0.04, ease: "power2.in" }, EX);
tl.to("#stepN-img-wrap",  { opacity: 0, x: 30, duration: 0.3, ease: "power2.in" }, EX + 0.1);
```

**结尾最终淡出**（仅结尾场景允许 fade out）：
```javascript
const FE = close_end - 1.5;
tl.to("#close-icon",       { opacity: 0, duration: 0.5, ease: "power2.in" }, FE);
tl.to("#close-title-cn",   { opacity: 0, y: -20, duration: 0.5, ease: "power2.in" }, FE + 0.1);
tl.to("#close-title-en",   { opacity: 0, y: -15, duration: 0.4, ease: "power2.in" }, FE + 0.2);
tl.to("#close-feedback-cn",{ opacity: 0, duration: 0.4, ease: "power2.in" }, FE + 0.3);
tl.to("#close-feedback-en",{ opacity: 0, duration: 0.3, ease: "power2.in" }, FE + 0.4);
tl.to("#close-line",       { scaleX: 0, opacity: 0, duration: 0.4, ease: "power2.in" }, FE + 0.5);
```

### 转场动画

- 0.1s 黑色闪光遮罩（快速淡入淡出各 0.05s）
- 位于每段结束前 0.1s

```javascript
// 转场 N（在 scene_end - 0.1 处）
const TR = scene_end - 0.1;
tl.from("#trans-N", { opacity: 0, duration: 0.05, ease: "none" }, TR);
tl.to("#trans-N",   { opacity: 0, duration: 0.05, ease: "none" }, TR + 0.05);
```

- z-index 必须高于所有场景（z-index: 20）

### GSAP 通用规则

- 所有 timeline 必须 `{ paused: true }` 并注册到 `window.__timelines["main"]`
- 不使用 `Math.random()`、`Date.now()` 等非确定性逻辑
- 不使用 `repeat: -1`，计算精确重复次数
- 同步构建 timeline，不用 async/await
- 不直接 animate video 元素，用 wrapper div
- 退场动画后添加 `tl.set` 硬杀帧（修复 lint 警告）
- 最后一帧可做 fade out，其他场景只用转场遮罩退出

---

## 设计风格

| 属性 | 值 |
|------|-----|
| 背景色 | #0a0a0a（近黑） |
| 主标题色 | #ffffff |
| 中文描述色 | #94a3b8 |
| 英文副标题色 | #64748b |
| 强调色 | #3b82f6（蓝） |
| 步骤标签背景 | rgba(59,130,246,0.15) |
| 截图区背景 | #111（图片未覆盖区域） |
| 字体 | Inter, Helvetica Neue, Arial, sans-serif |
| 分辨率 | 1920x1080 |
| 帧率 | 30fps（standard）/ 60fps（high） |

### 布局参数

| 元素 | 值 |
|------|-----|
| 外侧 padding | `0 80px` |
| 文字区宽度 | `flex: 0 0 380px` |
| 文字区右间距 | `padding-right: 40px` |
| 截图区宽度 | `flex: 1`（占满剩余空间） |
| 截图区高度 | `920px`（图片主导画面） |
| 截图圆角 | `16px` |
| 截图适配 | `object-fit: contain`（完整展示，不裁切） |

---

## 环境依赖说明（给复用者）

本 skill 依赖以下运行时环境，Step 0 会自动检测和安装：

| 依赖 | 最低版本 | 用途 | 安装方式 |
|------|---------|------|---------|
| Node.js | >= 22 | HyperFrames 运行时 | `brew install node` |
| FFmpeg | 任意 | 视频编码 + 音视频合并 | `brew install ffmpeg` |
| HyperFrames | latest (npx) | HTML→视频渲染引擎 | npx 自动下载 |
| Chrome | 捆绑版 | 无头浏览器逐帧截图 | `npx hyperframes browser` 自动下载 |
| edge-tts | 任意 | AI 配音引擎（免费 TTS） | `pip3 install edge-tts` |

**首次运行**时 `npx hyperframes` 会下载约 300MB（含 Chrome + 渲染依赖），后续运行会使用缓存。

**跨平台**：macOS 和 Linux 均支持。Windows 需要 WSL2 环境。

---

## 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 渲染后截图缺失/404 | 飞书图片 URL 过期（有效期仅几分钟） | 必须下载到本地（Step 3），使用本地路径 |
| 配音音色不一致 | 不同批次生成的 TTS 参数不同 | 所有 TTS 一次性批量生成（Step 8.2） |
| 音视频不对齐 | AAC 编码导致时长偏差 | 使用 WAV 中间格式，最终步骤才转 AAC |
| `edge-tts: not found` | macOS PATH 问题 | 使用 `python3 -m edge_tts` 调用 |
| npx ENOTEMPTY 错误 | npx 缓存损坏 | 删除 `~/.npm/_npx/<hash>` 后重试 |
