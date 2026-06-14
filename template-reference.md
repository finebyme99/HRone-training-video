# HRAS 培训视频 HTML 模板参考

## 完整模板结构

以下是生成的 `index.html` 的完整模板（基于 v2.0 规范）。根据实际步骤数量复制/删除中间的场景块。

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=1920, height=1080" />
    <script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      html, body {
        margin: 0; width: 1920px; height: 1080px;
        overflow: hidden; background: #0a0a0a;
      }
      body {
        font-family: "Inter", "Helvetica Neue", "Arial", sans-serif;
      }
    </style>
  </head>
  <body>
    <div id="root"
         data-composition-id="main"
         data-start="0"
         data-duration="{{total_dur}}"
         data-width="1920"
         data-height="1080">

      <!-- ========== SCENE: Title Card (0 ~ title_dur) ========== -->
      <div id="scene-title" class="clip"
           data-start="0" data-duration="{{title_dur}}" data-track-index="0"
           style="position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;background:#0a0a0a;z-index:10;">

        <!-- 系统名称 / System Name -->
        <p id="title-sys" style="font-size:48px;font-weight:500;color:#94a3b8;letter-spacing:3px;text-align:center;">
          {{系统名称-中文}}
        </p>

        <!-- 本期主题 / Episode Topic (最大字体) -->
        <h1 id="title-topic" style="font-size:80px;font-weight:700;color:#fff;letter-spacing:-1px;text-align:center;line-height:1.15;margin-top:16px;">
          {{本期主题-中文}}
        </h1>

        <!-- 期数 / Episode Number -->
        <p id="title-ep" style="font-size:24px;font-weight:400;color:#60a5fa;margin-top:16px;letter-spacing:1px;">
          HRAS系统敏捷培训 · 第{{X}}期
        </p>

        <!-- 英文装饰 badge -->
        <div id="title-badge" style="margin-top:40px;padding:10px 28px;border:1px solid rgba(59,130,246,0.3);border-radius:40px;background:rgba(59,130,246,0.08);">
          <span style="font-size:18px;color:#60a5fa;font-weight:500;">{{Topic in English}}</span>
        </div>
      </div>

      <!-- ========== SCENE: Step N (复制此块，替换 N) ========== -->
      <div id="scene-step{{N}}" class="clip"
           data-start="{{start}}" data-duration="{{dur}}" data-track-index="0"
           style="position:absolute;inset:0;display:flex;align-items:center;background:#0a0a0a;z-index:{{z}};">
        <div style="display:flex;width:100%;height:100%;align-items:center;padding:0 80px;">

          <!-- 左侧文字 (始终在左) -->
          <div id="step{{N}}-text" style="flex:0 0 380px;padding-right:40px;">
            <div id="step{{N}}-badge" style="display:inline-block;padding:6px 16px;background:rgba(59,130,246,0.15);border-radius:20px;margin-bottom:24px;">
              <span style="font-size:18px;color:#60a5fa;font-weight:600;">Step {{0N}}</span>
            </div>
            <h2 id="step{{N}}-title-cn" style="font-size:56px;font-weight:700;color:#fff;line-height:1.2;letter-spacing:-0.5px;">
              {{中文标题}}
            </h2>
            <p id="step{{N}}-title-en" style="font-size:28px;font-weight:400;color:#64748b;margin-top:8px;">
              {{English Title}}
            </p>
            <p id="step{{N}}-desc-cn" style="font-size:22px;color:#94a3b8;margin-top:20px;line-height:1.7;">
              {{中文描述第1行}}<br/>{{中文描述第2行}}
            </p>
            <p id="step{{N}}-desc-en" style="font-size:18px;color:#64748b;margin-top:12px;line-height:1.6;">
              {{English description line 1}}<br/>{{English description line 2}}
            </p>
          </div>

          <!-- 右侧截图 (始终在右) -->
          <div id="step{{N}}-img-wrap" style="flex:1;height:920px;overflow:hidden;border-radius:16px;position:relative;background:#111;">
            <img id="step{{N}}-img" src="images/step{{N}}.png"
                 style="width:100%;height:100%;object-fit:contain;object-position:center;" />
          </div>

        </div>
      </div>

      <!-- ========== SCENE: Closing (close_start ~ close_end) ========== -->
      <div id="scene-close" class="clip"
           data-start="{{close_start}}" data-duration="{{close_dur}}" data-track-index="0"
           style="position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;background:#0a0a0a;z-index:{{z}};">

        <div id="close-icon" style="width:80px;height:80px;border-radius:50%;background:linear-gradient(135deg,#3b82f6,#2563eb);display:flex;align-items:center;justify-content:center;margin-bottom:40px;">
          <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="20 6 9 17 4 12"></polyline>
          </svg>
        </div>

        <h2 id="close-title-cn" style="font-size:56px;font-weight:700;color:#fff;text-align:center;">
          本期结束
        </h2>
        <p id="close-title-en" style="font-size:26px;color:#64748b;margin-top:12px;text-align:center;">
          This episode is now complete
        </p>

        <p id="close-feedback-cn" style="font-size:28px;color:#94a3b8;margin-top:36px;text-align:center;">
          有任何心愿选题、优化建议，欢迎反馈！
        </p>
        <p id="close-feedback-en" style="font-size:20px;color:#64748b;margin-top:10px;text-align:center;">
          Got topic ideas or suggestions? We'd love your feedback!
        </p>

        <div id="close-line" style="width:120px;height:3px;background:linear-gradient(90deg,transparent,#3b82f6,transparent);border-radius:2px;margin-top:48px;"></div>
      </div>

      <!-- ========== TRANSITION OVERLAYS (0.1s flash) ========== -->
      <!-- 每个场景间插入一个 0.1s 的黑色闪光遮罩 -->
      <!-- z-index: 20 高于所有场景 -->
      <div id="trans-1" class="clip" data-start="{{t1}}" data-duration="0.1" data-track-index="1"
           style="position:absolute;inset:0;background:#0a0a0a;z-index:20;"></div>
      <!-- 复制此块，替换 N 和 {{tN}} -->

    </div>

    <script>
      window.__timelines = window.__timelines || {};
      const tl = gsap.timeline({ paused: true });

      // ============================================
      // === Title Card (0 ~ title_dur) ===
      // ============================================
      // Entrance
      tl.from("#title-sys",   { y: 30, opacity: 0, duration: 0.5, ease: "power3.out" }, 0.2);
      tl.from("#title-topic", { y: 50, opacity: 0, duration: 0.6, ease: "power3.out" }, 0.4);
      tl.from("#title-ep",    { y: 25, opacity: 0, duration: 0.4, ease: "power2.out" }, 0.7);
      tl.from("#title-badge", { y: 20, opacity: 0, duration: 0.4, ease: "power2.out" }, 1.0);
      // Exit (at title_dur - 0.5)
      tl.to("#title-sys",   { opacity: 0, y: -20, duration: 0.2, ease: "power2.in" }, {{title_exit}});
      tl.to("#title-topic", { opacity: 0, y: -30, duration: 0.2, ease: "power2.in" }, {{title_exit + 0.05}});
      tl.to("#title-ep",    { opacity: 0, y: -15, duration: 0.2, ease: "power2.in" }, {{title_exit + 0.1}});
      tl.to("#title-badge", { opacity: 0, y: -10, duration: 0.15, ease: "power2.in" }, {{title_exit + 0.15}});

      // ============================================
      // === Transition N (0.1s flash) ===
      // ============================================
      // tl.from("#trans-N", { opacity: 0, duration: 0.05, ease: "none" }, {{trans_start}});
      // tl.to("#trans-N",   { opacity: 0, duration: 0.05, ease: "none" }, {{trans_start + 0.05}});

      // ============================================
      // === Step N (step_start ~ step_end) ===
      // ============================================
      // const T = step_start;
      // Entrance
      // tl.from("#stepN-badge",    { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, T + 0.2);
      // tl.from("#stepN-title-cn", { y: 40,  opacity: 0, duration: 0.7, ease: "power3.out" }, T + 0.4);
      // tl.from("#stepN-title-en", { y: 30,  opacity: 0, duration: 0.6, ease: "power2.out" }, T + 0.6);
      // tl.from("#stepN-desc-cn",  { y: 25,  opacity: 0, duration: 0.5, ease: "power2.out" }, T + 0.8);
      // tl.from("#stepN-desc-en",  { y: 20,  opacity: 0, duration: 0.5, ease: "power2.out" }, T + 1.0);
      // tl.from("#stepN-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, T + 0.5);
      // Exit (at step_end - 0.6)
      // const EX = step_end - 0.6;
      // tl.to("#stepN-text > *", { opacity: 0, y: -20, duration: 0.3, stagger: 0.04, ease: "power2.in" }, EX);
      // tl.to("#stepN-img-wrap", { opacity: 0, x: 30, duration: 0.3, ease: "power2.in" }, EX + 0.1);

      // ============================================
      // === Closing (close_start ~ close_end) ===
      // ============================================
      // const CS = close_start;
      // Entrance
      // tl.from("#close-icon",       { scale: 0.5, opacity: 0, duration: 0.7, ease: "back.out(1.7)" }, CS + 0.3);
      // tl.from("#close-title-cn",   { y: 40, opacity: 0, duration: 0.7, ease: "power3.out" }, CS + 0.6);
      // tl.from("#close-title-en",   { y: 25, opacity: 0, duration: 0.5, ease: "power2.out" }, CS + 0.9);
      // tl.from("#close-feedback-cn",{ y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, CS + 1.2);
      // tl.from("#close-feedback-en",{ y: 15, opacity: 0, duration: 0.4, ease: "power2.out" }, CS + 1.4);
      // tl.from("#close-line",       { scaleX: 0, opacity: 0, duration: 0.7, ease: "power2.out" }, CS + 1.6);
      // Final fade out (at close_end - 1.5)
      // const FE = close_end - 1.5;
      // tl.to("#close-icon",       { opacity: 0, duration: 0.5, ease: "power2.in" }, FE);
      // tl.to("#close-title-cn",   { opacity: 0, y: -20, duration: 0.5, ease: "power2.in" }, FE + 0.1);
      // tl.to("#close-title-en",   { opacity: 0, y: -15, duration: 0.4, ease: "power2.in" }, FE + 0.2);
      // tl.to("#close-feedback-cn",{ opacity: 0, duration: 0.4, ease: "power2.in" }, FE + 0.3);
      // tl.to("#close-feedback-en",{ opacity: 0, duration: 0.3, ease: "power2.in" }, FE + 0.4);
      // tl.to("#close-line",       { scaleX: 0, opacity: 0, duration: 0.4, ease: "power2.in" }, FE + 0.5);

      window.__timelines["main"] = tl;
    </script>
  </body>
</html>
```

---

## 时长计算公式（TTS 驱动）

场景时长由 TTS 配音音频时长决定，而非文字阅读公式。

```
total_duration ≤ 60s（上限）

// 每个场景的时长 = TTS 音频时长 + 缓冲，向上取整
scene_dur = ceil(TTS_duration + 0.2)

// 封面最少 3 秒（给视觉动画留时间）
title_dur = max(3, ceil(TTS_title + 0.2))

// 结尾和其他步骤按 TTS 自然计算
close_dur = ceil(TTS_close + 0.2)
step_dur_N = ceil(TTS_step_N + 0.2)
```

**参数说明**：
- `TTS_duration`：该场景 edge-tts 生成的 mp3 音频实际时长（用 ffprobe 测量）
- `0.2`：缓冲时间，避免音频顶满场景边界
- `ceil`：向上取整到整数秒

**60s 封顶检查**：
```
total = title_dur + Σ(step_dur_N) + close_dur
if total > 60:
    // 精简步骤文字 → 重新生成 TTS → 重新计算
    // 不做等比压缩（会破坏 TTS 对齐）
```

**示例**（4 步，中文配音）：
```
TTS_title  = 1.99s → title = max(3, ceil(2.19)) = 3s
TTS_step1  = 7.80s → step1 = ceil(8.0)  = 10s
TTS_step2  = 7.94s → step2 = ceil(8.14) = 10s
TTS_step3  = 7.68s → step3 = ceil(7.88) = 10s
TTS_step4  = 7.03s → step4 = ceil(7.23) = 11s
TTS_close  = 5.62s → close = ceil(5.82) = 11s
Total = 3 + 10 + 10 + 10 + 11 + 11 = 55s ≤ 60 ✓
```

---

## 时间轴排布公式

```
T_title     = 0                  ~ title_dur
T_trans_1   = title_dur - 0.1   ~ title_dur

T_step_1    = title_dur          ~ title_dur + step_dur_1
T_trans_2   = step_1_end - 0.1  ~ step_1_end

T_step_2    = step_1_end         ~ step_1_end + step_dur_2
...
T_step_N    = cumsum_N-1         ~ cumsum_N-1 + step_dur_N

T_trans_last = cumsum_N - 0.1   ~ cumsum_N
T_close     = cumsum_N           ~ cumsum_N + close_dur

// 累积和：cumsum_k = title_dur + Σ(step_dur_1..k)
```

**关键**：每个场景的 `data-start` 和 `data-duration` 根据 TTS 计算出的时长动态填入。

---

## 布局参数对比（v1.4 → v2.0）

| 参数 | v1.4 (旧) | v2.0 (新) | 说明 |
|------|-----------|-----------|------|
| 文字区宽度 | `flex:0 0 520px` | `flex:0 0 380px` | 收窄文字区，给截图更多空间 |
| 文字区内距 | `padding-right:60px` | `padding-right:40px` | 减少间距 |
| 截图区高度 | `720px` | `920px` | 图片主导画面 |
| 截图适配 | `object-fit:cover` | `object-fit:contain` | 完整展示不裁切 |
| 截图背景 | 无 | `background:#111` | 未覆盖区域显示深灰 |
| 外侧 padding | `0 120px` | `0 80px` | 减少外侧留白 |
| 封面时长 | 固定 4.5s | TTS 驱动（≥3s） | 由配音决定 |
| 结尾时长 | 固定 5.5s | TTS 驱动 | 由配音决定 |
| 步骤时长 | 文字量公式 [5,10]s | TTS 驱动 | 由配音决定 |
| 转场时长 | 0.8s | 0.1s | 快速闪光 |
| 总时长上限 | 40s | 60s | 允许更多内容 |
| 图片来源 | 飞书 URL（会过期） | 本地 images/ | 必须下载 |

---

## 封面配音特殊规则

封面场景的 **4 个视觉元素全部保留**（系统名称、主题、期数、英文 badge），但 **配音只朗读本期主题**。

例如：
- 视觉显示：供应商协同平台 / 查看派遣需求 / HRAS系统敏捷培训·第3期 / View Dispatch Requirements
- 配音朗读：「查看派遣需求。」（完）

这样封面可以快速过渡到正文，不浪费时间在系统介绍上。

---

## GSAP 时间轴示例（ep3 参考值）

以下是 4 步、总 55s 视频的完整 GSAP 时间轴，可作为模板参考：

```javascript
// === Title Card (0 ~ 3s) ===
tl.from("#title-sys",   { y: 30, opacity: 0, duration: 0.5, ease: "power3.out" }, 0.2);
tl.from("#title-topic", { y: 50, opacity: 0, duration: 0.6, ease: "power3.out" }, 0.4);
tl.from("#title-ep",    { y: 25, opacity: 0, duration: 0.4, ease: "power2.out" }, 0.7);
tl.from("#title-badge", { y: 20, opacity: 0, duration: 0.4, ease: "power2.out" }, 1.0);
// Exit at 2.5s
tl.to("#title-sys",   { opacity: 0, y: -20, duration: 0.2, ease: "power2.in" }, 2.5);
tl.to("#title-topic", { opacity: 0, y: -30, duration: 0.2, ease: "power2.in" }, 2.55);
tl.to("#title-ep",    { opacity: 0, y: -15, duration: 0.2, ease: "power2.in" }, 2.6);
tl.to("#title-badge", { opacity: 0, y: -10, duration: 0.15, ease: "power2.in" }, 2.65);

// === Transition 1 at 2.9s (0.1s) ===
tl.from("#trans-1", { opacity: 0, duration: 0.05, ease: "none" }, 2.9);
tl.to("#trans-1",   { opacity: 0, duration: 0.05, ease: "none" }, 2.95);

// === Step 1 (3 ~ 13s) ===
tl.from("#step1-badge",    { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, 3.2);
tl.from("#step1-title-cn", { y: 40,  opacity: 0, duration: 0.7, ease: "power3.out" }, 3.4);
tl.from("#step1-title-en", { y: 30,  opacity: 0, duration: 0.6, ease: "power2.out" }, 3.6);
tl.from("#step1-desc-cn",  { y: 25,  opacity: 0, duration: 0.5, ease: "power2.out" }, 3.8);
tl.from("#step1-desc-en",  { y: 20,  opacity: 0, duration: 0.5, ease: "power2.out" }, 4.0);
tl.from("#step1-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, 3.5);
// Exit at 12.4s
tl.to("#step1-text > *", { opacity: 0, y: -20, duration: 0.3, stagger: 0.04, ease: "power2.in" }, 12.4);
tl.to("#step1-img-wrap", { opacity: 0, x: 30, duration: 0.3, ease: "power2.in" }, 12.5);

// === Transition 2 at 12.9s ===
tl.from("#trans-2", { opacity: 0, duration: 0.05, ease: "none" }, 12.9);
tl.to("#trans-2",   { opacity: 0, duration: 0.05, ease: "none" }, 12.95);

// === Step 2 (13 ~ 23s) ===
tl.from("#step2-badge",    { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, 13.2);
tl.from("#step2-title-cn", { y: 40,  opacity: 0, duration: 0.7, ease: "power3.out" }, 13.4);
tl.from("#step2-title-en", { y: 30,  opacity: 0, duration: 0.6, ease: "power2.out" }, 13.6);
tl.from("#step2-desc-cn",  { y: 25,  opacity: 0, duration: 0.5, ease: "power2.out" }, 13.8);
tl.from("#step2-desc-en",  { y: 20,  opacity: 0, duration: 0.5, ease: "power2.out" }, 14.0);
tl.from("#step2-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, 13.5);
// Exit at 22.4s
tl.to("#step2-text > *", { opacity: 0, y: -20, duration: 0.3, stagger: 0.04, ease: "power2.in" }, 22.4);
tl.to("#step2-img-wrap", { opacity: 0, x: 30, duration: 0.3, ease: "power2.in" }, 22.5);

// === Transition 3 at 22.9s ===
tl.from("#trans-3", { opacity: 0, duration: 0.05, ease: "none" }, 22.9);
tl.to("#trans-3",   { opacity: 0, duration: 0.05, ease: "none" }, 22.95);

// === Step 3 (23 ~ 33s) ===
tl.from("#step3-badge",    { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, 23.2);
tl.from("#step3-title-cn", { y: 40,  opacity: 0, duration: 0.7, ease: "power3.out" }, 23.4);
tl.from("#step3-title-en", { y: 30,  opacity: 0, duration: 0.6, ease: "power2.out" }, 23.6);
tl.from("#step3-desc-cn",  { y: 25,  opacity: 0, duration: 0.5, ease: "power2.out" }, 23.8);
tl.from("#step3-desc-en",  { y: 20,  opacity: 0, duration: 0.5, ease: "power2.out" }, 24.0);
tl.from("#step3-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, 23.5);
// Exit at 32.4s
tl.to("#step3-text > *", { opacity: 0, y: -20, duration: 0.3, stagger: 0.04, ease: "power2.in" }, 32.4);
tl.to("#step3-img-wrap", { opacity: 0, x: 30, duration: 0.3, ease: "power2.in" }, 32.5);

// === Transition 4 at 32.9s ===
tl.from("#trans-4", { opacity: 0, duration: 0.05, ease: "none" }, 32.9);
tl.to("#trans-4",   { opacity: 0, duration: 0.05, ease: "none" }, 32.95);

// === Step 4 (33 ~ 44s) ===
tl.from("#step4-badge",    { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, 33.2);
tl.from("#step4-title-cn", { y: 40,  opacity: 0, duration: 0.7, ease: "power3.out" }, 33.4);
tl.from("#step4-title-en", { y: 30,  opacity: 0, duration: 0.6, ease: "power2.out" }, 33.6);
tl.from("#step4-desc-cn",  { y: 25,  opacity: 0, duration: 0.5, ease: "power2.out" }, 33.8);
tl.from("#step4-desc-en",  { y: 20,  opacity: 0, duration: 0.5, ease: "power2.out" }, 34.0);
tl.from("#step4-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, 33.5);
// Exit at 43.4s
tl.to("#step4-text > *", { opacity: 0, y: -20, duration: 0.3, stagger: 0.04, ease: "power2.in" }, 43.4);
tl.to("#step4-img-wrap", { opacity: 0, x: 30, duration: 0.3, ease: "power2.in" }, 43.5);

// === Transition 5 at 43.9s ===
tl.from("#trans-5", { opacity: 0, duration: 0.05, ease: "none" }, 43.9);
tl.to("#trans-5",   { opacity: 0, duration: 0.05, ease: "none" }, 43.95);

// === Closing (44 ~ 55s) ===
tl.from("#close-icon",       { scale: 0.5, opacity: 0, duration: 0.7, ease: "back.out(1.7)" }, 44.3);
tl.from("#close-title-cn",   { y: 40, opacity: 0, duration: 0.7, ease: "power3.out" }, 44.6);
tl.from("#close-title-en",   { y: 25, opacity: 0, duration: 0.5, ease: "power2.out" }, 44.9);
tl.from("#close-feedback-cn",{ y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, 45.2);
tl.from("#close-feedback-en",{ y: 15, opacity: 0, duration: 0.4, ease: "power2.out" }, 45.4);
tl.from("#close-line",       { scaleX: 0, opacity: 0, duration: 0.7, ease: "power2.out" }, 45.6);
// Final fade out at 53.5s
tl.to("#close-icon",       { opacity: 0, duration: 0.5, ease: "power2.in" }, 53.5);
tl.to("#close-title-cn",   { opacity: 0, y: -20, duration: 0.5, ease: "power2.in" }, 53.6);
tl.to("#close-title-en",   { opacity: 0, y: -15, duration: 0.4, ease: "power2.in" }, 53.7);
tl.to("#close-feedback-cn",{ opacity: 0, duration: 0.4, ease: "power2.in" }, 53.8);
tl.to("#close-feedback-en",{ opacity: 0, duration: 0.3, ease: "power2.in" }, 53.9);
tl.to("#close-line",       { scaleX: 0, opacity: 0, duration: 0.4, ease: "power2.in" }, 54.0);
```

---

## 音频管线要点

1. **WAV 中间格式**：所有音频处理使用 WAV，最终合并时才转 AAC。AAC 编码会产生不精确的时长。
2. **TTS 批量生成**：所有场景的 TTS 一次性生成，确保音色一致。
3. **WAV 填充**：用 `apad=whole_dur={scene_dur}` 将每段 TTS 填充到精确的场景时长。
4. **拼接**：用 FFmpeg concat 将所有 padded WAV 拼成完整配音轨。
5. **SFX 定位**：用 `adelay={ms}|{ms}` 将音效定位到转场时间点。
6. **amix duration**：混合配音+SFX 时用 `duration=first`（以配音轨为准），不要用 `duration=shortest`。
