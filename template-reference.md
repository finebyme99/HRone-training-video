# HRAS 培训视频 HTML 模板参考

## 完整模板结构

以下是生成的 `index.html` 的完整模板。根据实际步骤数量复制/删除中间的场景块。

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
         data-duration="50"
         data-width="1920"
         data-height="1080">

      <!-- ========== SCENE: Title Card ========== -->
      <div id="scene-title" class="clip"
           data-start="0" data-duration="4.5" data-track-index="0"
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
        <div style="display:flex;width:100%;height:100%;align-items:center;padding:0 120px;">

          <!-- 左侧文字 (始终在左) -->
          <div id="step{{N}}-text" style="flex:0 0 520px;padding-right:60px;">
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
          <div id="step{{N}}-img-wrap" style="flex:1;height:720px;overflow:hidden;border-radius:16px;position:relative;">
            <img id="step{{N}}-img" src="{{filename}}.png"
                 style="width:100%;height:100%;object-fit:cover;object-position:center;" />
            <div style="position:absolute;inset:0;background:linear-gradient(90deg,#0a0a0a 0%,transparent 15%);"></div>

            <!-- 如果有红框/红箭头标注 → 添加关键帧标签 -->
            <!-- 取消下面的注释来启用 Key Step 标签 -->
            <!--
            <div id="step{{N}}-keytag" style="position:absolute;top:24px;right:24px;padding:8px 20px;background:rgba(239,68,68,0.9);border-radius:8px;">
              <span style="font-size:16px;color:#fff;font-weight:600;">Key Step</span>
            </div>
            -->
          </div>

        </div>
      </div>

      <!-- ========== SCENE: Closing ========== -->
      <div id="scene-close" class="clip"
           data-start="{{close_start}}" data-duration="5.5" data-track-index="0"
           style="position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;background:#0a0a0a;z-index:5;">

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

      <!-- ========== TRANSITION OVERLAYS ========== -->
      <!-- 每个场景间插入一个转场遮罩，duration=0.8s，z-index=20 -->
      <!-- <div id="trans-N" class="clip" data-start="{{t}}" data-duration="0.8" data-track-index="1"
           style="position:absolute;inset:0;background:#0a0a0a;z-index:20;"></div> -->

    </div>

    <script>
      window.__timelines = window.__timelines || {};
      const tl = gsap.timeline({ paused: true });

      // === Title Card ===
      tl.from("#title-sys", { y: 30, opacity: 0, duration: 0.6, ease: "power3.out" }, 0.3);
      tl.from("#title-topic", { y: 50, opacity: 0, duration: 0.8, ease: "power3.out" }, 0.6);
      tl.from("#title-ep", { y: 25, opacity: 0, duration: 0.5, ease: "power2.out" }, 1.0);
      tl.from("#title-badge", { y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, 1.3);
      // Exit
      tl.to("#title-sys", { opacity: 0, y: -20, duration: 0.4, ease: "power2.in" }, 3.6);
      tl.to("#title-topic", { opacity: 0, y: -30, duration: 0.5, ease: "power2.in" }, 3.7);
      tl.to("#title-ep", { opacity: 0, y: -15, duration: 0.4, ease: "power2.in" }, 3.8);
      tl.to("#title-badge", { opacity: 0, y: -10, duration: 0.3, ease: "power2.in" }, 3.9);

      // === Step scenes (repeat for each step) ===
      // Transition
      // tl.from("#trans-N", { opacity: 0, duration: 0.4, ease: "power1.inOut" }, T);
      // tl.to("#trans-N", { opacity: 0, duration: 0.4, ease: "power1.inOut" }, T+0.4);
      //
      // Entrance
      // tl.from("#stepN-badge", { x: -30, opacity: 0, duration: 0.5, ease: "power3.out" }, T+0.6);
      // tl.from("#stepN-title-cn", { y: 40, opacity: 0, duration: 0.7, ease: "power3.out" }, T+0.8);
      // tl.from("#stepN-title-en", { y: 30, opacity: 0, duration: 0.6, ease: "power2.out" }, T+1.0);
      // tl.from("#stepN-desc-cn", { y: 25, opacity: 0, duration: 0.5, ease: "power2.out" }, T+1.2);
      // tl.from("#stepN-desc-en", { y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, T+1.4);
      // tl.from("#stepN-img-wrap", { x: 60, opacity: 0, scale: 0.95, duration: 0.8, ease: "expo.out" }, T+0.9);
      //
      // Key-frame highlight (ONLY if screenshot has red box/arrow)
      // tl.to("#stepN-img", { scale: 1.03, duration: 0.4, ease: "power2.inOut", yoyo: true, repeat: 1 }, T+2.0);
      // tl.from("#stepN-keytag", { x: 40, opacity: 0, duration: 0.5, ease: "power3.out" }, T+2.0);
      //
      // Exit
      // tl.to("#stepN-text > *", { opacity: 0, y: -20, duration: 0.4, stagger: 0.05, ease: "power2.in" }, END-1.0);
      // tl.to("#stepN-img-wrap", { opacity: 0, x: 30, duration: 0.4, ease: "power2.in" }, END-0.9);
      // tl.set("#stepN-img-wrap", { opacity: 0 }, END-0.5);  // hard kill

      // === Closing ===
      // tl.from("#close-icon", { scale: 0.5, opacity: 0, duration: 0.7, ease: "back.out(1.7)" }, CS+0.4);
      // tl.from("#close-title-cn", { y: 40, opacity: 0, duration: 0.8, ease: "power3.out" }, CS+0.7);
      // tl.from("#close-title-en", { y: 25, opacity: 0, duration: 0.6, ease: "power2.out" }, CS+1.0);
      // tl.from("#close-feedback-cn", { y: 20, opacity: 0, duration: 0.5, ease: "power2.out" }, CS+1.3);
      // tl.from("#close-feedback-en", { y: 15, opacity: 0, duration: 0.5, ease: "power2.out" }, CS+1.5);
      // tl.from("#close-line", { scaleX: 0, opacity: 0, duration: 0.8, ease: "power2.out" }, CS+1.7);
      // Final fade out (allowed on last scene)
      // tl.to("#close-icon", { opacity: 0, duration: 0.6, ease: "power2.in" }, END-1.0);
      // tl.to("#close-title-cn", { opacity: 0, y: -20, duration: 0.6, ease: "power2.in" }, END-0.9);
      // tl.to("#close-title-en", { opacity: 0, y: -15, duration: 0.5, ease: "power2.in" }, END-0.8);
      // tl.to("#close-feedback-cn", { opacity: 0, duration: 0.5, ease: "power2.in" }, END-0.7);
      // tl.to("#close-feedback-en", { opacity: 0, duration: 0.4, ease: "power2.in" }, END-0.6);
      // tl.to("#close-line", { scaleX: 0, opacity: 0, duration: 0.5, ease: "power2.in" }, END-0.5);

      window.__timelines["main"] = tl;
    </script>
  </body>
</html>
```

## 时长计算公式（动态阅读时长）

每个步骤的展示时长根据文字量自动计算，50s 只是上限，不要求必须填满。

```
total_duration ≤ 50s（上限）

title_dur = 4.5    (固定)
close_dur = 5.5    (固定)
transition_overlap = 0.8  // 转场嵌入步骤时长内

// 每步的阅读时长计算：
raw_N = cn_chars_N × 0.3 + en_words_N × 0.25 + 3.0
step_dur_N = clamp(raw_N, min=5, max=10)

// 50s 封顶检查：
total = 4.5 + Σ(step_dur_N) + 5.5
if total > 50:
    available = 50 - 4.5 - 5.5 = 40s
    scale = available / Σ(step_dur_N)
    step_dur_N = max(step_dur_N × scale, 5)  // 压缩但不低于5s
```

**参数说明**：
- `cn_chars_N`：第 N 步所有中文文字（标题+描述）的字符数
- `en_words_N`：第 N 步所有英文文字（标题+描述）的单词数
- `0.3`：中文字符阅读速度（约 3.3 字/秒）
- `0.25`：英文单词阅读速度（约 4 词/秒）
- `3.0`：动画入场 + 视觉消化基础时间

**示例 A**（5 步，每步约 30 字中文 + 15 词英文）：
```
raw = 30×0.3 + 15×0.25 + 3 = 15.75 → clamp → 10s
total = 4.5 + 5×10 + 5.5 = 60s > 50 → 压缩 → 每步 8s → 总 50s
```

**示例 B**（3 步，每步约 15 字中文 + 8 词英文）：
```
raw = 15×0.3 + 8×0.25 + 3 = 9.5s → clamp → 9.5s
total = 4.5 + 3×9.5 + 5.5 = 38.5s ≤ 50 → 保持 → 总 38.5s ✓
```

## 时间轴排布公式（适配可变步骤时长）

```
T_title     = 0       ~ 4.5
T_trans_1   = 4.0     ~ 4.8

// 每步起始 = 前一步结束时间（累积）
T_step_1    = 4.5     ~ 4.5 + step_dur_1
T_trans_2   = T_step_1_end - 0.5
T_step_2    = T_step_1_end  ~ T_step_1_end + step_dur_2
...
T_step_N    = cumsum_N-1    ~ cumsum_N-1 + step_dur_N

// 累积和：cumsum_k = 4.5 + Σ(step_dur_1..k)

T_trans_last = cumsum_N - 0.5
T_close     = cumsum_N  ~ cumsum_N + 5.5
```

**关键**：每步的 `data-start` 和 `data-duration` 根据该步计算出的时长动态填入，不再等分。

## 关键帧高亮动画（红框/红箭头截图）

当截图包含红框或红箭头标注时，表示该处有系统关键操作位置。
需要在截图入场后添加脉冲动画和标签：

```javascript
// 截图入场后 1.5s，执行脉冲缩放
tl.to("#stepN-img", {
  scale: 1.03,
  duration: 0.4,
  ease: "power2.inOut",
  yoyo: true,
  repeat: 1   // 不是 -1！ 计算: 一次脉冲 = in + out = 0.8s
}, sceneStart + 2.0);

// 同时滑入 Key Step 标签
tl.from("#stepN-keytag", {
  x: 40,
  opacity: 0,
  duration: 0.5,
  ease: "power3.out"
}, sceneStart + 2.0);
```

HTML 中需要在截图 wrapper 内添加标签元素：
```html
<div id="stepN-keytag" style="position:absolute;top:24px;right:24px;padding:8px 20px;background:rgba(239,68,68,0.9);border-radius:8px;z-index:5;">
  <span style="font-size:16px;color:#fff;font-weight:600;">Key Step</span>
</div>
```

## 判断截图是否有红框/红箭头

在 Step 3 中通过 Read 工具查看每张截图时，关注以下特征：
- 红色矩形框（通常高亮按钮或菜单项）
- 红色箭头（指向特定操作位置）
- alt 文本中提到 "红色箭头"、"红色框"、"red arrow"、"red box"

如果有任何上述特征，该场景启用关键帧高亮动画。
