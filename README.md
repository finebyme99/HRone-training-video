# HRone Training Video Generator

Generate bilingual (Chinese + English) system training videos (≤50s, MP4) from Feishu/Lark documents with screenshots. Built on [HyperFrames](https://github.com/heygen-com/hyperframes) (HTML + GSAP → MP4).

**One-liner**: Give an AI agent a Feishu doc link → get a polished training video.

---

## Quick Start (3 Steps)

### 1. Install the skill files

```bash
# macOS / Linux
mkdir -p ~/.qoderworkcn/skills/hras-training-video

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video"
```

Download the two files from this repo and place them in the directory above:
- [`SKILL.md`](./SKILL.md) — main workflow + rules
- [`template-reference.md`](./template-reference.md) — HTML template reference

```bash
# macOS / Linux (assuming files in ~/Downloads)
cp ~/Downloads/SKILL.md ~/.qoderworkcn/skills/hras-training-video/
cp ~/Downloads/template-reference.md ~/.qoderworkcn/skills/hras-training-video/

# Windows (PowerShell)
Copy-Item "$env:USERPROFILE\Downloads\SKILL.md" "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video\"
Copy-Item "$env:USERPROFILE\Downloads\template-reference.md" "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video\"
```

### 2. Run environment setup

Tell your AI agent:

> **安装 HRAS 视频环境**

The agent will auto-check and install all 5 dependencies (Node.js, FFmpeg, HyperFrames, lark-cli, lark-doc skill). See [Agent Auto-Setup](#agent-auto-setup) below for details.

### 3. Generate a video

Tell your AI agent:

> **帮我生成 HRAS 培训视频**

The agent will ask for: system name, topic, episode number, Feishu doc link, and target section. Then wait 3-5 minutes for the MP4.

---

## Agent Auto-Setup

This section is written for AI agents (QoderWork, Cursor, Claude, Copilot, etc.) to follow programmatically.

### Prerequisites check & auto-install

Execute these checks in order. For any that fail, auto-install where possible:

#### Check 1: Node.js ≥ 22

```bash
node --version
```

If missing or version < 22:

```bash
# macOS
brew install node

# Linux
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt-get install -y nodejs

# Windows (winget)
winget install OpenJS.NodeJS.LTS
```

#### Check 2: FFmpeg

```bash
ffmpeg -version 2>&1 | head -1
```

If missing:

```bash
# macOS
brew install ffmpeg

# Linux
sudo apt-get install ffmpeg

# Windows (winget)
winget install Gyan.FFmpeg
```

#### Check 3: HyperFrames rendering engine

```bash
npx --yes hyperframes@latest doctor 2>&1
```

This auto-downloads HyperFrames + bundled Chrome (~300MB first time). Confirm no critical errors in output.

#### Check 4: lark-cli (Feishu doc reader)

```bash
which lark-cli 2>&1
```

If missing: lark-cli is bundled with QoderWork's Lark integration. User needs to trigger auth by sending any Feishu doc link to their agent.

```bash
lark-cli auth status 2>&1
```

If not authorized: guide user to send a Feishu doc link to trigger the OAuth flow.

#### Check 5: lark-doc skill

```bash
ls ~/.qoderworkcn/skills/lark-doc/SKILL.md 2>&1
```

If missing: install `lark-doc` skill from QoderWork Skill Market, or download from a colleague.

### Summary report

After all checks, output:

```
Environment check complete:
  ✓ Node.js v22.x.x
  ✓ FFmpeg x.x
  ✓ HyperFrames x.x.x
  ✓ lark-cli (authorized)
  ✓ lark-doc skill (installed)
Ready to generate videos.
```

---

## File Structure

```
HRone-training-video/
├── README.md                  ← You are here
├── SKILL.md                   ← Agent workflow instructions
├── template-reference.md      ← HTML composition template
└── LICENSE                    ← MIT
```

### How to use these files

| Platform | Where to put files | How agent loads them |
|----------|-------------------|---------------------|
| QoderWork | `~/.qoderworkcn/skills/hras-training-video/` | Auto-loaded as skill |
| Cursor | Project root or `.cursor/` directory | Reference in chat or `@` the file |
| Claude Code | Project root or `~/.claude/` | Reference in conversation |
| Other agents | Any accessible path | Feed `SKILL.md` content as context |

---

## Dependencies

| Dependency | Min Version | Purpose | Auto-install? |
|-----------|-------------|---------|--------------|
| Node.js | ≥ 22 | HyperFrames runtime | Yes (`brew` / `apt`) |
| FFmpeg | any | Video encoding | Yes (`brew` / `apt`) |
| HyperFrames | latest | HTML→MP4 renderer | Yes (`npx` auto-download) |
| Chrome | bundled | Headless frame capture | Yes (HyperFrames auto-download) |
| lark-cli | any | Feishu doc API | Via QoderWork Lark integration |
| lark-doc skill | any | Feishu doc reading | Via Skill Market |

---

## Usage

### Trigger words

Say any of these to your AI agent:

- `生成 HRAS 培训视频` / `Generate HRAS training video`
- `制作系统培训视频` / `Make system training video`
- `安装 HRAS 视频环境` / `Setup HRAS video environment` (setup-only mode)

### Input parameters

The agent will collect these via conversation:

| Parameter | Example |
|-----------|---------|
| System name (CN + EN) | 供应商协同平台 / Supplier Collaboration Platform |
| Topic (CN + EN) | 创建员工 / Create Employees |
| Episode number | 第1期 |
| Feishu doc link | `https://ztn.feishu.cn/wiki/xxx` |
| Target section | 1.2 创建员工 |

### Output

- Format: MP4 (H.264)
- Resolution: 1920×1080
- Duration: ≤ 50 seconds (dynamic based on text length)
- Structure: Title card → N steps (left text + right screenshot) → Closing

---

## Video Structure

Each video follows a fixed layout:

1. **Title Card** (4.5s) — System name + topic + episode + English badge
2. **Steps** (5-10s each) — Left: bilingual text, Right: screenshot, dark background
3. **Closing** (5.5s) — "本期结束" + feedback prompt

Screenshots with red box/arrow annotations automatically trigger a "Key Step" pulse highlight animation.

---

## Windows Users

Windows requires WSL2 for the rendering pipeline:

```powershell
# Install WSL2 (run PowerShell as Administrator)
wsl --install
```

After reboot, open the WSL2 terminal (Ubuntu) and run all commands from there. Skill files can be placed in the Windows-side `~/.qoderworkcn/skills/` directory; QoderWork will find them.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Render crashes mid-way | Close other apps to free RAM, retry |
| `lark-cli: not found` | Send any Feishu doc link to your agent to trigger installation |
| `lark-cli auth` shows unauthorized | Send a Feishu doc link, browser will pop up for OAuth |
| `HyperFrames init` fails with "directory not empty" | `rm -rf ~/.npm/_npx/<cache-dir>` to clear npx cache |
| Slow first render | Normal — HyperFrames downloads ~300MB Chrome bundle on first run |

---

## License

MIT
