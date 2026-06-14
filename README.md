# HRone Training Video Generator

Generate bilingual (Chinese + English) system training videos (≤60s, MP4) from Feishu/Lark documents with screenshots. Built on [HyperFrames](https://github.com/heygen-com/hyperframes) (HTML + GSAP → MP4), with AI voiceover (Chinese/English), optional background music and transition sound effects. Screenshots downloaded locally from Feishu docs for reliable rendering.

**One-liner**: Give an AI agent a Feishu doc link → get a polished training video.

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

---

## For Humans

Want to generate HRAS training videos? Just send this link to your AI agent:

```
https://github.com/finebyme99/HRone-training-video
```

Then say: **帮我生成 HRAS 培训视频**

The agent will handle everything — environment setup, collecting your inputs, reading the Feishu doc, and rendering the MP4. You just answer a few questions and wait 3-5 minutes.

### What you'll need to provide

| # | What | Example |
|---|------|---------|
| 1 | System name (CN + EN) | 供应商协同平台 / Supplier Collaboration Platform |
| 2 | Topic (CN + EN) | 创建员工 / Create Employees |
| 3 | Episode number | 第1期 |
| 4 | Feishu doc link | `https://ztn.feishu.cn/wiki/xxxxx` |
| 5 | Target section | 1.2 创建员工 |
| 6 | Voiceover language | Chinese or English narration |
| 7 | Background music | Yes / No (recommended) |
| 8 | Transition SFX | Yes (recommended) / No |

The Feishu doc should contain screenshots of the operation steps (the more the better, 5-15 is ideal).

---

## Quick Start (Manual Setup)

### 1. Install the skill files

```bash
# macOS / Linux
mkdir -p ~/.qoderworkcn/skills/hras-training-video

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.qoderworkcn\skills\hras-training-video"
```

Download the files from this repo and place them in the directory above:
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

The agent will auto-check and install all 6 dependencies (Node.js, FFmpeg, HyperFrames, lark-cli, lark-doc skill, edge-tts). See [Agent Auto-Setup](#agent-auto-setup) below for details.

### 3. Generate a video

Tell your AI agent:

> **帮我生成 HRAS 培训视频**

The agent will first show you a preparation checklist, then ask for 8 pieces of info.

**Tips**:

- The Feishu doc must contain screenshots of the operation steps (the more the better, 5-15 is ideal)
- One doc can produce multiple episodes — just pick a different section each time
- The doc must be accessible to your Feishu account
- Videos include AI voiceover — choose Chinese or English narration when prompted
- Background music and SFX are optional — no BGM by default, SFX recommended

After you provide these, the agent will confirm the details and start generating. Wait 3-5 minutes for the MP4.

---

## Agent Auto-Setup

This section is written for AI agents to follow programmatically.

### Prerequisites check & auto-install

#### Check 1: Node.js ≥ 22

```bash
node --version
```

If missing or version < 22: `brew install node` (macOS) / `apt install nodejs` (Linux)

#### Check 2: FFmpeg

```bash
ffmpeg -version 2>&1 | head -1
```

If missing: `brew install ffmpeg` (macOS) / `apt install ffmpeg` (Linux)

#### Check 3: HyperFrames rendering engine

```bash
npx --yes hyperframes@latest doctor 2>&1
```

Auto-downloads HyperFrames + bundled Chrome (~300MB first time). Confirm no critical errors.

#### Check 4: lark-cli (Feishu doc reader)

```bash
which lark-cli 2>&1
lark-cli auth status 2>&1
```

If missing: bundled with QoderWork's Lark integration. If unauthorized: guide user to send a Feishu doc link.

#### Check 5: lark-doc skill

```bash
ls ~/.qoderworkcn/skills/lark-doc/SKILL.md 2>&1
```

If missing: install from QoderWork Skill Market.

#### Check 6: edge-tts (AI voiceover)

```bash
python3 -m edge_tts --version 2>&1
```

If missing: `pip3 install edge-tts`

> **Note**: On macOS, use `python3 -m edge_tts` instead of `edge-tts` (PATH issues).

---

## File Structure

```
HRone-training-video/
├── README.md                  ← You are here (project overview)
├── agent-guide.md             ← Agent operation manual (read this first!)
├── SKILL.md                   ← Detailed workflow + GSAP animation rules
├── template-reference.md      ← HTML composition template + formulas
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
| FFmpeg | any | Video encoding + audio mixing | Yes (`brew` / `apt`) |
| HyperFrames | latest | HTML→MP4 renderer | Yes (`npx` auto-download) |
| Chrome | bundled | Headless frame capture | Yes (HyperFrames auto-download) |
| lark-cli | any | Feishu doc API + image download | Via QoderWork Lark integration |
| lark-doc skill | any | Feishu doc reading | Via Skill Market |
| edge-tts | any | AI voiceover (free TTS) | Yes (`pip3`) |

---

## Output Specs

- Format: MP4 (H.264)
- Resolution: 1920x1080
- Duration: ≤ 60 seconds (TTS-driven, dynamic)
- Structure: Title card (≥3s) → N steps (left text + right screenshot) → Closing
- Audio: AI voiceover (Chinese or English, via edge-tts) + optional BGM + optional SFX
- Screenshots: downloaded locally, displayed with `object-fit: contain` (full view, no cropping)

---

## Video Structure

Each video follows a fixed layout:

1. **Title Card** (≥3s, TTS-driven) — System name + topic + episode + English badge (voiceover reads only the topic)
2. **Steps** (TTS-driven duration each) — Left: bilingual text (380px), Right: screenshot (920px, contain), dark background
3. **Closing** (TTS-driven) — "本期结束" + feedback prompt

Transitions between scenes: 0.1s black flash overlay.

---

## Windows Users

Windows requires WSL2 for the rendering pipeline:

```powershell
# Install WSL2 (run PowerShell as Administrator)
wsl --install
```

After reboot, open the WSL2 terminal (Ubuntu) and run all commands from there.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Screenshots blank in render | Feishu URLs expired — must download locally via `lark-cli docs +media-download` |
| Voice sounds different between scenes | TTS generated in separate batches — regenerate ALL in one batch |
| Audio/video out of sync | AAC duration imprecise — use WAV intermediate format |
| `edge-tts: not found` | macOS PATH issue — use `python3 -m edge_tts` |
| Render crashes | Close other apps to free RAM, retry |
| `lark-cli: not found` | Send any Feishu doc link to trigger installation |
| `HyperFrames init` ENOTEMPTY | Delete `~/.npm/_npx/<cache-dir>` and retry |
| Slow first render | Normal — ~300MB Chrome bundle download |

---

## License

MIT
