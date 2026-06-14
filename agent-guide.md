# HRAS 培训视频生成器 — Agent 操作指南

> **本文档面向 AI Agent（Claude Code、Codex、Qoder、Workbuddy、Cursor 等）。**
> 你收到这个 GitHub 链接后，请通读本文档，然后按步骤执行，全程自动完成环境检测、用户输入收集和视频生成。

GitHub 仓库：https://github.com/finebyme99/HRone-training-video

---

## 仓库文件说明

本仓库包含生成视频所需的全部指令文件：

| 文件 | 用途 |
|------|------|
| `SKILL.md` | 完整工作流指令（Step 0-9），含环境检测、用户引导、HTML 生成规则、GSAP 动画规范、音频管线 |
| `template-reference.md` | HTML 模板代码、TTS 驱动时长公式、时间轴排布、布局参数 |
| `README.md` | 项目介绍、依赖列表、故障排除 |

**首先**，请读取仓库中的 `SKILL.md` 和 `template-reference.md`，获取完整的工作流细节。本文档是精简版操作指引，汇总了关键步骤。

---

## 核心规则（必读）

### 配音语言一致性

**用户在第一步选择的配音语言是最终决定，后续不得更改。** 中文配音始终中文，英文配音始终英文。不要因为场景内容或 agent 判断而切换语言。

### 封面配音

封面场景配音**只朗读本期主题**（如"查看派遣需求"），不读系统名称、培训名称、期数。视觉元素全部保留。

### TTS 批量生成

所有场景的 TTS 文件**必须在同一批次中生成**，使用相同的声音和语速参数，确保音色一致。

### 截图必须下载到本地

飞书图片 URL 有效期仅几分钟，**必须下载到本地使用**，不能直接使用飞书 URL。

### 音频使用 WAV 中间格式

所有音频处理使用 WAV 格式，最终合并时才转 AAC。AAC 编码会产生不精确的时长。

---

## 第一步：环境检测与自动安装

按顺序检测 6 项依赖，能自动装的直接装，不要只提示用户。

### 1.1 Node.js >= 22

```bash
node --version
```

缺失或版本不够 → 自动安装：

```bash
# macOS
brew install node

# Linux
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt-get install -y nodejs

# Windows (winget)
winget install OpenJS.NodeJS.LTS
```

如果 `brew` 也不存在 → 告知用户先装 Homebrew：
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

### 1.2 FFmpeg

```bash
ffmpeg -version 2>&1 | head -1
```

缺失 → 自动安装：

```bash
# macOS
brew install ffmpeg

# Linux
sudo apt-get install ffmpeg

# Windows (winget)
winget install Gyan.FFmpeg
```

### 1.3 HyperFrames 渲染引擎

```bash
npx --yes hyperframes@latest doctor 2>&1
```

该命令自动下载 HyperFrames + 捆绑 Chrome（~300MB 首次）。确认输出无 critical error。

> 如遇 `ENOTEMPTY` 报错：删除 npx 缓存目录 `~/.npm/_npx/<hash>` 后重试。

### 1.4 lark-cli（飞书文档读取工具）

```bash
which lark-cli 2>&1
```

存在 → 验证授权：

```bash
lark-cli auth status 2>&1
```

未授权 → 引导用户在对话中发送任意飞书文档链接，触发 OAuth 授权流程。

不存在 → lark-cli 通常随 QoderWork 的飞书集成自动安装。如果用户的 agent 不是 QoderWork，需要单独安装 lark-cli（通过 npm 或同事分发）。

### 1.5 lark-doc skill

```bash
ls ~/.qoderworkcn/skills/lark-doc/SKILL.md 2>&1
```

缺失 → 告知用户需要安装 `lark-doc` skill。在 QoderWork 中可从 Skill Market 搜索安装；其他 agent 需要获取 skill 文件夹放到对应路径。

### 1.6 edge-tts（AI 配音引擎）

```bash
python3 -m edge_tts --version 2>&1
```

缺失 → 自动安装：`pip3 install edge-tts`

> **注意**：macOS 上 `which edge-tts` 可能找不到（二进制不在 PATH 中）。始终使用 `python3 -m edge_tts` 调用。

### 汇总报告

检测完毕后向用户输出：

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

如有未通过项，列出具体问题和修复建议。**不要跳过直接进入视频制作。**

---

## 第二步：引导用户输入

环境就绪后，**先展示准备清单**，再收集信息。不要直接甩问题。

### 2.1 展示准备清单

```
📋 生成视频前，请准备以下 8 项信息：

1️⃣ 系统名称（中英文）
   例如：供应商协同平台 / Supplier Collaboration Platform

2️⃣ 本期主题（中英文）
   例如：创建员工 / Create Employees

3️⃣ 期数
   例如：第1期、第2期

4️⃣ 飞书文档链接
   含操作截图的飞书云文档 URL，格式如 https://ztn.feishu.cn/wiki/xxxxx
   ⚠️ 文档需有权限访问，且包含截图

5️⃣ 目标章节
   文档中要提取的章节编号，如 1.2 创建员工
   一份文档可做每期视频，每期选不同章节

6️⃣ 配音语言（中文/英文）
   中文配音或英文旁白（选定后不可更改）

7️⃣ 背景音乐
   是否在视频中加入轻松的背景音乐
   · 需要 / 不需要（推荐）

8️⃣ 转场音效
   是否在场景切换时加入音效
   · 需要（推荐） / 不需要
```

### 2.2 收集信息

展示清单后，逐一询问用户。如果用户的初始消息中已给出全部 8 项，可跳过询问直接确认。

### 2.3 确认信息

```
收到！我整理一下：
  · 系统：XX / XX
  · 主题：XX / XX
  · 期数：第X期
  · 文档：https://ztn.feishu.cn/wiki/xxxxx
  · 章节：X.X
  · 配音：中文/英文
  · 背景音乐：需要/不需要
  · 转场音效：需要/不需要
确认无误，开始生成视频 🎬
```

信息不完整或格式可疑（如链接不像飞书 URL）→ 追问，不要硬猜。

---

## 第三步：读取飞书文档

1. 先读取 `lark-doc` 的 fetch 参考（`~/.qoderworkcn/skills/lark-doc/references/lark-doc-fetch.md`），了解 lark-cli 的文档读取命令
2. 用 `outline` 获取文档目录，定位目标章节的 block-id
3. 用 `section --start-block-id <id> --detail with-ids` 读取章节全文
4. 提取所有 `<image token="...">` 的 token 值和上下文描述文字

---

## 第四步：下载截图到本地

**飞书图片 URL 有效期仅几分钟，必须下载！**

```bash
mkdir -p images/

# 逐个下载（token 来自文档中的 <image token="xxx"> 标签）
lark-cli docs +media-download --token {TOKEN} --output images/
# 重复每个 token...
```

下载后重命名为语义化名称（`step1.png`、`step2.png` ...），整理对应表：

| 步骤 | 图片文件 | 原始 token | 内容描述 |
|------|---------|-----------|---------|
| Step 1 | images/step1.png | xxx | 派遣管理菜单 |

然后用 Read 工具逐张查看图片内容，识别操作步骤和关键 UI 元素。

---

## 第五步：规划分镜

总时长 **≤ 60 秒**（上限），结构固定三段：

| 场景 | 时长 | 内容 |
|------|------|------|
| 封面 | TTS 驱动（≥3s） | 系统名称 + 主题 + 期数（配音只读主题） |
| 操作步骤 × N | TTS 驱动 | 左文右图，中英双语 |
| 结尾 | TTS 驱动 | 固定结束语 |

### 时长计算

```
scene_dur = ceil(TTS_duration + 0.2)
title_dur = max(3, ceil(TTS_title + 0.2))
total = title_dur + Σ(step_dur) + close_dur ≤ 60
```

- 先生成所有 TTS 音频 → 用 ffprobe 测量时长 → 计算场景时长
- 如果 total > 60：精简步骤文字，重新生成 TTS

---

## 第六步：生成 HTML

参照仓库中的 `template-reference.md` 生成 HyperFrames 项目。

```bash
# 初始化项目
npx hyperframes init hras-training-ep<期数> --example blank --non-interactive

# 复制图片到项目目录
cp -r images/ hras-training-ep<期数>/images/
```

将生成的 HTML 写入 `index.html`。

### 关键规则

- **图片使用本地路径**：`<img src="images/step1.png">`，不使用飞书 URL
- **封面三行居中**：系统名称 48px 灰、主题 80px 白（最大字）、期数 24px 蓝 + 英文 badge
- **步骤统一左文右图**：左侧 380px 文字区，右侧截图区 flex:1、920px 高、圆角 16px
- **截图完整展示**：`object-fit: contain`（不裁切），背景 `#111`
- **中英双语**：标题中文 56px 白 + 英文 28px 灰，描述中文 22px + 英文 18px
- **结尾固定**："本期结束 / This episode is now complete" + 反馈提示
- **转场 0.1s**：黑色闪光遮罩，每场景结束前 0.1s

### GSAP 动画规范

- Timeline：`{ paused: true }`，注册到 `window.__timelines["main"]`
- 入场：文字 `y/opacity` power3.out，截图 `x/opacity/scale` expo.out
- 退场：文字 `opacity/y` stagger 0.04，截图 `opacity/x`
- 退场后必须 `tl.set` 硬杀帧（修复 `gsap_exit_missing_hard_kill`）
- 转场：0.1s 黑色闪光（淡入 0.05s + 淡出 0.05s），z-index: 20
- **禁止**：`repeat: -1`、`Math.random()`、`Date.now()`

---

## 第七步：验证与渲染

```bash
cd hras-training-ep<N>

# Lint（修复所有 error；gsap_exit_missing_hard_kill 警告必须修复）
npx hyperframes lint

# 渲染无声视频
npx hyperframes render --quality high --output renders/hras-ep<N>-silent.mp4
```

---

## 第八步：生成配音 & 音频 & 合并

### 8.1 配音文本

| 场景 | 中文配音文本 | 英文配音文本 |
|------|------------|------------|
| 封面 | **{本期主题}**（只读主题！） | **{Topic}**（只读主题！） |
| 步骤 N | {中文标题}。{中文描述} | {English Title}. {English description} |
| 结尾 | 本期结束。有任何心愿选题、优化建议，欢迎反馈。 | This episode is now complete. Got topic ideas or suggestions? We'd love your feedback! |

**所有场景必须使用同一语言配音。**

### 8.2 批量生成 TTS（一次性全部生成）

```bash
# 中文（voice: zh-CN-XiaoxiaoNeural, rate: +5%）
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{封面文本}" --write-media audio/tts_title.mp3
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{步骤1文本}" --write-media audio/tts_step1.mp3
# ... 所有场景一次生成
python3 -m edge_tts --voice zh-CN-XiaoxiaoNeural --rate=+5% \
  --text "{结尾文本}" --write-media audio/tts_close.mp3
```

### 8.3 测量 & 计算时长

```bash
ffprobe -v error -show_entries format=duration -of csv=p=0 audio/tts_title.mp3
# → 1.99s → scene_dur = max(3, ceil(2.19)) = 3s
ffprobe -v error -show_entries format=duration -of csv=p=0 audio/tts_step1.mp3
# → 7.80s → scene_dur = ceil(8.0) = 10s
```

### 8.4 构建配音轨（WAV 中间格式）

```bash
# 填充每段到精确场景时长
ffmpeg -i audio/tts_title.mp3 -af "apad=whole_dur=3" -ar 24000 -ac 1 audio/pad_title.wav
ffmpeg -i audio/tts_step1.mp3 -af "apad=whole_dur=10" -ar 24000 -ac 1 audio/pad_step1.wav
# ... 所有场景

# 拼接
cat > audio/concat_list.txt << 'EOF'
file 'pad_title.wav'
file 'pad_step1.wav'
...
file 'pad_close.wav'
EOF
ffmpeg -f concat -safe 0 -i audio/concat_list.txt -ar 24000 -ac 1 audio/voiceover.wav
```

### 8.5 转场音效（如用户选择"需要"）

推荐 Mixkit 免费音效（现代风格）：

```bash
# 下载
curl -L -o audio/sfx_raw.mp3 "https://assets.mixkit.co/active_storage/sfx/2364/2364-preview.mp3"
# 转 WAV
ffmpeg -i audio/sfx_raw.mp3 -af "volume=1.1" -ar 24000 -ac 1 audio/sfx_processed.wav
```

### 8.6 混合 & 合并

```bash
# 仅配音
ffmpeg -i renders/hras-ep<N>-silent.mp4 -i audio/voiceover.wav \
  -c:v copy -c:a aac -shortest -map 0:v:0 -map 1:a:0 renders/hras-ep<N>.mp4

# 配音 + SFX（用 adelay 定位音效到转场时间点）
ffmpeg -i audio/voiceover.wav -i audio/sfx_processed.wav -i audio/sfx_processed.wav ... \
  -filter_complex "\
    [0]apad[vo];\
    [1]adelay=2900|2900,volume=0.3[s1];\
    [2]adelay=12900|12900,volume=0.3[s2];\
    [vo][s1][s2]amix=inputs=3:duration=first[aout]" \
  -map "[aout]" -t ${TOTAL_DUR} audio/final_audio.wav

ffmpeg -i renders/hras-ep<N>-silent.mp4 -i audio/final_audio.wav \
  -c:v copy -c:a aac -shortest -map 0:v:0 -map 1:a:0 renders/hras-ep<N>.mp4
```

**⚠️ amix 注意**：配音+SFX 用 `duration=first`（以配音为准），不要用 `duration=shortest`。

---

## 第九步：交付

将 MP4 文件路径提供给用户。

---

## 故障排除

| 问题 | 解决 |
|------|------|
| 渲染后截图空白/404 | 飞书 URL 过期了 → 必须下载到本地（第四步） |
| 配音音色不一致 | TTS 分多批生成了 → 一次性全部重新生成 |
| 音视频不对齐 | AAC 时长偏差 → 使用 WAV 中间格式 |
| `edge-tts: not found` | macOS PATH 问题 → 用 `python3 -m edge_tts` |
| 渲染崩溃 | 关闭其他应用释放内存后重试 |
| `lark-cli: not found` | 发送任意飞书文档链接触发安装 |
| `lark-cli auth` 未授权 | 发送飞书链接，浏览器弹窗点授权 |
| `HyperFrames init` 报 ENOTEMPTY | 删除 `~/.npm/_npx/<缓存目录>` 后重试 |
| 首次渲染很慢 | 正常 — HyperFrames 下载 ~300MB Chrome 捆绑包 |

---

## Windows 用户

渲染管线需要 WSL2：

```powershell
# 管理员 PowerShell
wsl --install
```

重启后在 WSL2 终端（Ubuntu）中执行所有命令。Skill 文件可放在 Windows 侧 `~/.qoderworkcn/skills/` 目录。

---

## 输出规格

- 格式：MP4（H.264）
- 分辨率：1920x1080
- 时长：≤ 60 秒（TTS 驱动动态计算）
- 音频：AI 配音（中文或英文，edge-tts）+ 可选背景音乐 + 可选转场音效
- 结构：封面 → N 个步骤（左文右图）→ 结尾

---

## 给人类用户

如果你想让 AI agent 帮你生成培训视频，只需要把这个仓库链接发给它：

```
https://github.com/finebyme99/HRone-training-video
```

然后告诉它：

> 帮我生成 HRAS 培训视频

Agent 会读取本仓库的指令文件，自动完成所有操作。你只需回答它的几个问题（系统名称、主题等），然后等 3-5 分钟拿 MP4。
