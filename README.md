# HRAS 系统敏捷培训视频生成器

给 AI Agent 一个飞书文档链接 → 自动生成 60 秒内的中英双语培训短视频（MP4）。

基于 [HyperFrames](https://github.com/heygen-com/hyperframes)（HTML + GSAP → MP4）+ edge-tts（免费 AI 配音），截图从飞书文档下载到本地渲染，风格参考飞书/Apple 官方产品介绍。

---


## 输出成果案例
点击进飞书查看往期视频：https://ztn.feishu.cn/file/DEJcbADW0oshzexb6Kec5Vq0nLe


## 使用方法

把这个仓库链接发给你的 AI Agent：

```
https://github.com/finebyme99/HRone-training-video
```

然后说：**帮我生成 HRAS 培训视频**

Agent 会自动完成所有操作——环境安装、信息收集、读取飞书文档、渲染 MP4。你只需要回答几个问题，等 3-5 分钟拿视频。

### 你需要准备的信息

| # | 信息 | 示例 |
|---|------|------|
| 1 | 系统名称（中英文） | 供应商协同平台 / Supplier Collaboration Platform |
| 2 | 本期主题（中英文） | 创建员工 / Create Employees |
| 3 | 期数 | 第1期 |
| 4 | 飞书文档链接 | `https://ztn.feishu.cn/wiki/xxxxx` |
| 5 | 目标章节 | 1.2 创建员工 |
| 6 | 配音语言 | 中文 / 英文 |
| 7 | 背景音乐 | 需要 / 不需要（推荐） |
| 8 | 转场音效 | 需要（推荐） / 不需要 |

**小贴士**：

- 飞书文档里需要包含操作步骤的截图（越多越好，5-15 张为佳）
- 一份文档可以做多期视频——每期选不同章节即可
- 文档需要你的飞书账号有访问权限
- 配音语言选定后不可更改，请提前确认

---

## 手动安装（适合非 QoderWork 用户）

### 1. 安装 Skill 文件

```bash
# macOS / Linux
mkdir -p ~/.qoderworkcn/skills/hras-training-video
cp ~/Downloads/SKILL.md ~/.qoderworkcn/skills/hras-training-video/
cp ~/Downloads/template-reference.md ~/.qoderworkcn/skills/hras-training-video/

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video"
Copy-Item "$env:USERPROFILE\Downloads\SKILL.md" "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video\"
Copy-Item "$env:USERPROFILE\Downloads\template-reference.md" "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video\"
```

### 2. 环境检测

告诉 Agent：**安装 HRAS 视频环境**

Agent 会自动检测并安装 6 项依赖（Node.js、FFmpeg、HyperFrames、lark-cli、lark-doc skill、edge-tts）。

### 3. 生成视频

告诉 Agent：**帮我生成 HRAS 培训视频**

---

## 视频结构

每个视频固定三段式布局：

1. **封面**（≥3s）—— 系统名称 + 主题 + 期数 + 英文 badge（配音只读主题）
2. **操作步骤** × N（TTS 驱动时长）—— 左：中英双语文字（380px），右：系统截图（920px，完整展示不裁切），深色背景
3. **结尾**（TTS 驱动时长）—— "本期结束" + 反馈提示

场景间转场：0.1s 黑色闪光。

### 输出规格

- 格式：MP4（H.264）
- 分辨率：1920×1080
- 时长：≤ 60 秒（TTS 驱动，动态计算）
- 音频：AI 配音（中文或英文，edge-tts）+ 可选背景音乐 + 可选转场音效
- 截图：下载到本地，`object-fit: contain` 完整展示

---

## 文件结构

```
HRone-training-video/
├── README.md                  ← 项目介绍（你正在看的）
├── agent-guide.md             ← Agent 操作手册
├── SKILL.md                   ← 完整工作流 + GSAP 动画规范
├── template-reference.md      ← HTML 模板 + 时长公式
└── LICENSE                    ← MIT
```

### 各平台 Skill 安装位置

| 平台 | Skill 文件位置 | Agent 加载方式 |
|------|--------------|--------------|
| QoderWork | `~/.qoderworkcn/skills/hras-training-video/` | 自动加载 |
| Cursor | 项目根目录或 `.cursor/` | 在聊天中 `@` 引用 |
| Claude Code | 项目根目录或 `~/.claude/` | 在对话中引用 |
| 其他 Agent | 任意可访问路径 | 将 SKILL.md 内容作为上下文传入 |

---

## 环境依赖

| 依赖 | 最低版本 | 用途 | 自动安装 |
|------|---------|------|---------|
| Node.js | ≥ 22 | HyperFrames 运行时 | `brew install node` |
| FFmpeg | 任意 | 视频编码 + 音频混合 | `brew install ffmpeg` |
| HyperFrames | latest | HTML→MP4 渲染引擎 | `npx` 自动下载 |
| Chrome | 捆绑版 | 无头浏览器逐帧截图 | HyperFrames 自动下载 |
| lark-cli | 任意 | 飞书文档 API + 图片下载 | QoderWork 飞书集成 |
| lark-doc skill | 任意 | 飞书文档读取 | Skill Market 安装 |
| edge-tts | 任意 | AI 配音（免费 TTS） | `pip3 install edge-tts` |

---

## Windows 用户

渲染管线需要 WSL2 环境：

```powershell
# 管理员 PowerShell
wsl --install
```

重启后在 WSL2 终端（Ubuntu）中执行所有命令。

---

## 故障排除

| 问题 | 解决 |
|------|------|
| 渲染后截图空白 | 飞书 URL 过期 → 必须下载到本地（`lark-cli docs +media-download`） |
| 不同场景配音音色不一致 | TTS 分批生成了 → 一次性全部重新生成 |
| 音视频不对齐 | AAC 时长不精确 → 使用 WAV 中间格式 |
| `edge-tts: not found` | macOS PATH 问题 → 用 `python3 -m edge_tts` |
| 渲染崩溃 | 关闭其他应用释放内存后重试 |
| `lark-cli: not found` | 发送任意飞书文档链接触发安装 |
| `HyperFrames init` 报 ENOTEMPTY | 删除 `~/.npm/_npx/<缓存目录>` 后重试 |
| 首次渲染很慢 | 正常——HyperFrames 需下载 ~300MB Chrome 捆绑包 |

---

## For AI Agents

If you're an AI agent (Claude Code, Codex, Qoder, Workbuddy, Cursor, etc.) and received this repo link:

1. Read [`agent-guide.md`](./agent-guide.md) — your complete step-by-step operation manual
2. Read [`SKILL.md`](./SKILL.md) — detailed workflow rules and GSAP animation specs
3. Read [`template-reference.md`](./template-reference.md) — HTML template and duration formulas

Then follow agent-guide.md from Step 1 to Step 9. Everything you need is in this repo.

### Critical Rules

- **Voiceover language**: Always follow the user's original selection. Never switch languages mid-video.
- **Title voiceover**: Only read the episode topic, NOT system name/training info/episode number.
- **TTS batch**: Generate ALL TTS files in one batch for consistent voice quality.
- **Local images**: Download Feishu screenshots locally (URLs expire in minutes).
- **WAV intermediate**: Use WAV for all audio processing; only convert to AAC at final merge.

### Agent Auto-Setup

Execute these checks in order. Auto-install where possible:

```bash
# 1. Node.js ≥ 22
node --version
# Missing? brew install node / apt install nodejs

# 2. FFmpeg
ffmpeg -version 2>&1 | head -1
# Missing? brew install ffmpeg / apt install ffmpeg

# 3. HyperFrames rendering engine
npx --yes hyperframes@latest doctor 2>&1
# Auto-downloads ~300MB Chrome bundle on first run

# 4. lark-cli (Feishu doc reader)
which lark-cli 2>&1 && lark-cli auth status 2>&1
# Missing: bundled with QoderWork Lark integration
# Unauthorized: guide user to send a Feishu doc link

# 5. lark-doc skill
ls ~/.qoderworkcn/skills/lark-doc/SKILL.md 2>&1
# Missing: install from QoderWork Skill Market

# 6. edge-tts (AI voiceover)
python3 -m edge_tts --version 2>&1
# Missing: pip3 install edge-tts
# Note: on macOS use `python3 -m edge_tts` (PATH issues)
```

---

## License

MIT
