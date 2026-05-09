# 爵士贝斯练琴助手 v2 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在现有单文件index.html中新增节拍器、六维RPG评分系统、像素画风主题，所有数据存localStorage。

**Architecture:** 单文件HTML，内联CSS+JS。CSS按像素主题/节拍器/结算画面分块，JS按数据层/节拍器引擎/评分引擎/UI分块。与现有日历系统和练琴计划共存于侧边栏导航。

**Tech Stack:** 纯HTML/CSS/JS，Web Audio API (节拍器)，Google Fonts (Press Start 2P + Silkscreen)，localStorage (数据持久化)

---

## 文件变更

| 文件 | 操作 | 职责 |
|------|------|------|
| `index.html` | 大幅修改 | 全站HTML/CSS/JS，新增节拍器页+RPG系统+结算面板 |

---

## 数据模型（JS全局常量+localStorage）

```javascript
const STORAGE_RPG = 'bass_rpg_state';
const DIM_KEYS = ['tech','rhythm','ear','harmony','improv','repertoire'];
const DIM_LABELS = {
  tech:'技术基本功', rhythm:'节奏/时值感', ear:'听力',
  harmony:'和声/乐理', improv:'即兴/创造力', repertoire:'曲库/实战'
};
const DIM_EMOJI = { tech:'🔧', rhythm:'⏱️', ear:'👂', harmony:'📚', improv:'🎵', repertoire:'🎤' };

// 累加升级阈值
const LEVEL_THRESHOLDS = [0,300,700,1300,2100,3100,4600,6300,8300,99999];
const LEVEL_NAMES = [
  '初级I','初级II','初级III',
  '中级I','中级II','中级III',
  '高级I','高级II','高级III'
];

// 标签权重
const TAG_WEIGHTS = {
  arpeggio:    { tech:6, harmony:2 },
  walking:     { tech:2, rhythm:3, improv:2, repertoire:1 },
  ear:         { ear:6, repertoire:2 },
  metronome:   { rhythm:6 },
  improv:      { improv:5, tech:2 },
  theory:      { harmony:6 },
  song:        { repertoire:4, rhythm:2, improv:1 },
  technique:   { tech:5 }
};

const TAG_OPTIONS = [
  { id:'arpeggio',  label:'🎹 琶音/音阶' },
  { id:'walking',   label:'🚶 Walking Bass' },
  { id:'ear',       label:'🎧 扒谱/练耳' },
  { id:'metronome', label:'⏱️ 节拍器/节奏' },
  { id:'improv',    label:'🎵 即兴/Solo' },
  { id:'theory',    label:'📚 乐理/和声' },
  { id:'song',      label:'🎤 曲目跟弹' },
  { id:'technique', label:'🔧 手指/技巧' }
];

// 默认状态
function defaultRpgState() {
  return {
    scores: { tech:0, rhythm:0, ear:0, harmony:0, improv:0, repertoire:0 },
    lastPractice: { tech:null, rhythm:null, ear:null, harmony:null, improv:null, repertoire:null },
    history: []
  };
}
```

---

### Task 1: 像素RPG画风 — CSS主题改造

**文件:** 修改 `index.html` 的 `<style>` 块头部 + 各组件样式

- [ ] **Step 1: 在 `<head>` 中加入 Google Fonts**

在 `<meta charset="UTF-8">` 之后插入：
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Silkscreen:wght@400;700&display=swap" rel="stylesheet">
```

- [ ] **Step 2: 替换 `:root` 变量，加入像素主题色**

将现有 `:root` 块替换为：
```css
:root {
  --bg: #1a1a2e;
  --surface: #25253e;
  --surface2: #2e2e4a;
  --border: #4a4a6a;
  --text: #e8e0d0;
  --text2: #a09880;
  --accent: #f0b030;
  --accent2: #d4882a;
  --green: #58b868;
  --red: #e04848;
  --blue: #58a8d8;
  --pixel-green: #48c048;
  --pixel-dark: #16213e;
  --radius: 0px; /* 像素风 = 直角 */
  --shadow: 4px 4px 0px rgba(0,0,0,0.5);
  --font-pixel: 'Press Start 2P', monospace;
  --font-ui: 'Silkscreen', monospace;
}
```

- [ ] **Step 3: 替换全局字体和基础样式**

在 `* { margin:0... }` 之后，`body` 之前加入：
```css
body {
  font-family: var(--font-ui);
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  display: flex;
  image-rendering: pixelated;
  image-rendering: crisp-edges;
}
h1, h2, h3, h4 { font-family: var(--font-pixel); font-weight: 400; }
h2 { font-size: 1rem; }
h3 { font-size: 0.75rem; }
```

- [ ] **Step 4: 像素化所有交互元素**

在现有CSS之后追加像素风覆盖样式：
```css
/* Pixel borders */
.sidebar, .modal, .plan-section, .stat-card, .day-detail, .page {
  border: 3px solid var(--border);
  box-shadow: var(--shadow);
}

/* Pixel buttons */
.btn, .mood-btn, .cal-nav-btn {
  font-family: var(--font-ui);
  border: 3px solid;
  border-color: #888 #444 #222 #666;
  background: var(--surface2);
  color: var(--text);
  font-size: 0.8em;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  transition: none;
}
.btn:active, .mood-btn:active, .cal-nav-btn:active {
  border-color: #444 #888 #666 #222;
  transform: translate(1px, 1px);
}
.btn-primary {
  border-color: #f8c848 #885800 #583800 #d09828;
  background: var(--accent);
  color: #1a1a1a;
}

/* Pixel progress bars (经验条) */
.pixel-bar {
  height: 14px;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  position: relative;
  overflow: hidden;
}
.pixel-bar-fill {
  height: 100%;
  background: repeating-linear-gradient(
    90deg,
    var(--accent) 0px,
    var(--accent) 8px,
    #c08020 8px,
    #c08020 10px
  );
  transition: width 0.5s steps(20);
}

/* Pixel input */
input, textarea {
  font-family: var(--font-ui);
  border: 2px solid var(--border);
  background: var(--pixel-dark);
  color: var(--text);
  padding: 8px 10px;
  font-size: 0.8em;
}
input:focus, textarea:focus {
  border-color: var(--accent);
  outline: none;
  box-shadow: 0 0 0 2px rgba(240,176,48,0.3);
}

/* Nav items pixel */
.nav-item {
  font-family: var(--font-ui);
  font-size: 0.75em;
  border-left: 4px solid transparent;
}
.nav-item.active {
  border-left: 4px solid var(--accent);
}

/* Sidebar pixel */
.sidebar {
  border-right: 4px solid var(--border);
  background: var(--pixel-dark);
}

/* Calendar day pixel hover */
.cal-day { border: 2px solid transparent; }
.cal-day:hover { border-color: var(--accent); background: var(--surface2); }
.cal-day.today { border-color: var(--accent); background: rgba(240,176,48,0.15); }
.cal-day.checked { background: var(--surface2); }
```

- [ ] **Step 5: 打开浏览器验证像素主题**

在浏览器打开 `index.html`，确认所有页面（日历/计划/统计）显示像素风格字体和边框，按钮有3D边框效果。

---

### Task 2: 节拍器 — HTML结构 + CSS

**文件:** 修改 `index.html`

- [ ] **Step 1: 在侧边栏导航中加入节拍器入口**

在"练琴统计"的 `.nav-item` 之后，streak-box 之前插入：
```html
<div class="nav-item" data-page="metronome">
  <span class="nav-icon">⏱️</span> 节拍器
</div>
```

- [ ] **Step 2: 在 MAIN 区域加入节拍器页面**

在 `page-stats` 的 `</div>` 之后、`</div>` (main闭合) 之前插入：
```html
<!-- METRONOME PAGE -->
<div class="page" id="page-metronome">
  <h2>⏱️ 节拍器</h2>

  <div class="metro-container">
    <div class="metro-pulse" id="metroPulse">
      <div class="metro-pulse-inner" id="metroPulseInner"></div>
    </div>

    <div class="metro-beats" id="metroBeats">
      <span class="beat-light" data-beat="0"></span>
      <span class="beat-light" data-beat="1"></span>
      <span class="beat-light" data-beat="2"></span>
      <span class="beat-light" data-beat="3"></span>
    </div>

    <div class="metro-bpm-display">
      <span class="metro-bpm-num" id="metroBpmNum">120</span>
      <span class="metro-bpm-label">BPM</span>
    </div>

    <div class="metro-slider-row">
      <button class="metro-bpm-btn" id="btnBpmMinus">-</button>
      <input type="range" class="metro-slider" id="metroBpmSlider" min="20" max="250" value="120">
      <button class="metro-bpm-btn" id="btnBpmPlus">+</button>
    </div>

    <div class="metro-subdiv">
      <span class="metro-label">细分</span>
      <div class="metro-btn-group" id="subdivGroup">
        <button class="metro-opt active" data-subdiv="1">四分 ♩</button>
        <button class="metro-opt" data-subdiv="2">八分 ♪</button>
        <button class="metro-opt" data-subdiv="4">十六分 ♬</button>
        <button class="metro-opt" data-subdiv="3">三连音 🎵</button>
      </div>
    </div>

    <div class="metro-accent">
      <span class="metro-label">重音</span>
      <div class="metro-btn-group" id="accentGroup">
        <button class="metro-opt active" data-accent="1">第1拍</button>
        <button class="metro-opt" data-accent="24">第2 & 4拍</button>
      </div>
    </div>

    <div class="metro-toggles">
      <label class="metro-check"><input type="checkbox" id="chkSound" checked> 🔊 声音</label>
      <label class="metro-check"><input type="checkbox" id="chkVisual" checked> 👁️ 视觉闪烁</label>
    </div>

    <div class="metro-controls">
      <button class="btn btn-primary metro-start" id="btnMetroStart">▶ START</button>
      <button class="btn btn-ghost metro-tap" id="btnTapTempo">👆 TAP</button>
    </div>
  </div>
</div>
```

- [ ] **Step 3: 在 CSS 中加入节拍器样式**

在 `</style>` 之前追加：
```css
/* ===== Metronome ===== */
.metro-container {
  max-width: 440px;
  margin: 0 auto;
  text-align: center;
  background: var(--surface);
  border: 3px solid var(--border);
  box-shadow: var(--shadow);
  padding: 24px 20px;
}
.metro-pulse {
  width: 80px; height: 80px;
  border-radius: 50%;
  margin: 0 auto 16px;
  background: var(--pixel-dark);
  border: 4px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.05s, border-color 0.05s;
}
.metro-pulse-inner {
  width: 20px; height: 20px;
  border-radius: 50%;
  background: var(--text2);
  transition: background 0.05s, transform 0.05s;
}
.metro-pulse.active { border-color: var(--accent); transform: scale(1.15); }
.metro-pulse.active .metro-pulse-inner { background: var(--accent); transform: scale(1.6); }
.metro-pulse.accent { border-color: var(--red); }
.metro-pulse.accent .metro-pulse-inner { background: var(--red); }

.metro-beats {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 16px;
}
.beat-light {
  width: 28px; height: 28px;
  border-radius: 4px;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  transition: background 0.05s;
}
.beat-light.on { background: var(--accent); border-color: var(--accent); }
.beat-light.accent { background: var(--red); border-color: var(--red); }

.metro-bpm-display { margin-bottom: 12px; }
.metro-bpm-num {
  font-family: var(--font-pixel);
  font-size: 2.2em;
  color: var(--accent);
  line-height: 1.2;
}
.metro-bpm-label {
  display: block;
  font-size: 0.7em;
  color: var(--text2);
  font-family: var(--font-ui);
  letter-spacing: 0.1em;
}
.metro-slider-row {
  display: flex;
  align-items: center;
  gap: 10px;
  justify-content: center;
  margin-bottom: 18px;
}
.metro-slider {
  width: 200px;
  -webkit-appearance: none;
  height: 8px;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  outline: none;
}
.metro-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px; height: 20px;
  background: var(--accent);
  border: 2px solid #fff;
  cursor: pointer;
}
.metro-bpm-btn {
  width: 36px; height: 36px;
  font-family: var(--font-pixel);
  font-size: 1em;
  background: var(--surface2);
  border: 3px solid;
  border-color: #888 #444 #222 #666;
  color: var(--text);
  cursor: pointer;
}
.metro-bpm-btn:active {
  border-color: #444 #888 #666 #222;
  transform: translate(1px, 1px);
}
.metro-subdiv, .metro-accent { margin-bottom: 12px; }
.metro-label {
  display: block;
  font-size: 0.7em;
  color: var(--text2);
  margin-bottom: 6px;
  font-family: var(--font-ui);
  text-transform: uppercase;
  letter-spacing: 0.08em;
}
.metro-btn-group { display: flex; gap: 6px; justify-content: center; flex-wrap: wrap; }
.metro-opt {
  padding: 6px 14px;
  font-family: var(--font-ui);
  font-size: 0.75em;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  color: var(--text2);
  cursor: pointer;
}
.metro-opt.active {
  background: var(--accent);
  color: #1a1a1a;
  border-color: var(--accent);
  font-weight: 700;
}
.metro-toggles {
  display: flex;
  gap: 20px;
  justify-content: center;
  margin-bottom: 18px;
}
.metro-check {
  font-family: var(--font-ui);
  font-size: 0.75em;
  display: flex;
  align-items: center;
  gap: 6px;
  cursor: pointer;
  color: var(--text2);
}
.metro-check input[type="checkbox"] {
  width: 16px; height: 16px;
  accent-color: var(--accent);
  cursor: pointer;
}
.metro-controls {
  display: flex;
  gap: 12px;
  justify-content: center;
}
.metro-start { min-width: 160px; font-size: 1em; padding: 14px 24px; }
.metro-tap { min-width: 80px; }
```

---

### Task 3: 节拍器 — JS 引擎

**文件:** 修改 `index.html` 的 `<script>` 块，在 INIT 之前插入节拍器JS

- [ ] **Step 1: 在 INIT 区域之前插入节拍器状态变量和引擎**

```javascript
// ============================================================
// METRONOME ENGINE
// ============================================================
let metroState = {
  running: false,
  bpm: 120,
  subdiv: 1,     // 1=四分 2=八分 4=十六分 3=三连音
  accent: '1',   // '1' = 第1拍重音, '24' = 第2&4拍重音
  soundOn: true,
  visualOn: true,
  beatIndex: 0,  // 0-based, counts within current measure
  audioCtx: null,
  intervalId: null,
  tapTimes: []
};

function getMetroIntervalMs() {
  // 一拍 = 60000/bpm, 再除以细分数
  const beatMs = 60000 / metroState.bpm;
  return beatMs / metroState.subdiv;
}

function getTotalBeats() {
  // 4/4拍: 4 beats * subdiv
  return 4 * metroState.subdiv;
}

function isAccentBeat(idx) {
  const total = getTotalBeats();
  const beatInMeasure = idx % total;
  if (metroState.accent === '1') {
    // 重音在第1拍（subdiv=1时beat 0；subdiv>1时每个大拍的第1个细分）
    return beatInMeasure % metroState.subdiv === 0 && Math.floor(beatInMeasure / metroState.subdiv) === 0;
  } else if (metroState.accent === '24') {
    // 重音在第2和第4拍的第1个细分
    const bigBeat = Math.floor(beatInMeasure / metroState.subdiv);
    const isFirstOfBigBeat = beatInMeasure % metroState.subdiv === 0;
    return isFirstOfBigBeat && (bigBeat === 1 || bigBeat === 3);
  }
  return false;
}

function playClick(isAccent) {
  if (!metroState.soundOn || !metroState.audioCtx) return;
  const ctx = metroState.audioCtx;
  const now = ctx.currentTime;
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.type = 'square';
  osc.frequency.value = isAccent ? 1000 : 800;
  gain.gain.setValueAtTime(0.3, now);
  gain.gain.exponentialRampToValueAtTime(0.001, now + 0.05);
  osc.start(now);
  osc.stop(now + 0.05);
}

function flashBeat(isAccent) {
  if (!metroState.visualOn) return;
  const pulse = document.getElementById('metroPulse');
  const inner = document.getElementById('metroPulseInner');
  const lights = document.querySelectorAll('.beat-light');
  const total = getTotalBeats();
  const idx = metroState.beatIndex % total;
  const bigBeat = Math.floor(idx / metroState.subdiv);

  // pulse circle
  pulse.classList.add('active');
  if (isAccent) pulse.classList.add('accent');
  else pulse.classList.remove('accent');

  // beat lights
  lights.forEach((l, i) => {
    l.classList.remove('on', 'accent');
    if (i === bigBeat) {
      l.classList.add('on');
      if (isAccent) l.classList.add('accent');
    }
  });

  setTimeout(() => {
    pulse.classList.remove('active', 'accent');
  }, 80);
}

function metroTick() {
  const isAccent = isAccentBeat(metroState.beatIndex);
  playClick(isAccent);
  flashBeat(isAccent);
  metroState.beatIndex++;
}

function startMetronome() {
  if (metroState.running) return;
  if (!metroState.audioCtx) {
    metroState.audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  if (metroState.audioCtx.state === 'suspended') {
    metroState.audioCtx.resume();
  }
  metroState.running = true;
  metroState.beatIndex = 0;
  document.getElementById('btnMetroStart').textContent = '■ STOP';
  metroTick();
  metroState.intervalId = setInterval(metroTick, getMetroIntervalMs());
}

function stopMetronome() {
  metroState.running = false;
  if (metroState.intervalId) {
    clearInterval(metroState.intervalId);
    metroState.intervalId = null;
  }
  document.getElementById('btnMetroStart').textContent = '▶ START';
  // Reset visual
  document.getElementById('metroPulse').classList.remove('active', 'accent');
  document.querySelectorAll('.beat-light').forEach(l => l.classList.remove('on', 'accent'));
}

function updateBpm(bpm) {
  metroState.bpm = Math.max(20, Math.min(250, bpm));
  document.getElementById('metroBpmNum').textContent = metroState.bpm;
  document.getElementById('metroBpmSlider').value = metroState.bpm;
  if (metroState.running) {
    clearInterval(metroState.intervalId);
    metroState.intervalId = setInterval(metroTick, getMetroIntervalMs());
  }
}

function handleTapTempo() {
  const now = Date.now();
  const times = metroState.tapTimes;
  times.push(now);
  // Keep last 8 taps
  while (times.length > 8) times.shift();
  if (times.length >= 3) {
    let sum = 0;
    for (let i = 1; i < times.length; i++) {
      sum += times[i] - times[i-1];
    }
    const avgMs = sum / (times.length - 1);
    const bpm = Math.round(60000 / avgMs);
    updateBpm(Math.max(20, Math.min(250, bpm)));
  }
  // Clear taps if idle for 2s
  clearTimeout(metroState._tapTimeout);
  metroState._tapTimeout = setTimeout(() => { metroState.tapTimes = []; }, 2000);
}
```

- [ ] **Step 2: 在 INIT 函数中加入节拍器事件绑定**

在现有 `init()` 函数中追加：
```javascript
// --- Metronome ---
document.getElementById('btnMetroStart').addEventListener('click', () => {
  if (metroState.running) stopMetronome();
  else startMetronome();
});

document.getElementById('btnTapTempo').addEventListener('click', handleTapTempo);

document.getElementById('metroBpmSlider').addEventListener('input', (e) => {
  updateBpm(parseInt(e.target.value));
});

document.getElementById('btnBpmMinus').addEventListener('click', () => {
  updateBpm(metroState.bpm - 1);
});

document.getElementById('btnBpmPlus').addEventListener('click', () => {
  updateBpm(metroState.bpm + 1);
});

document.querySelectorAll('#subdivGroup .metro-opt').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('#subdivGroup .metro-opt').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    metroState.subdiv = parseInt(btn.dataset.subdiv);
    if (metroState.running) {
      clearInterval(metroState.intervalId);
      metroState.intervalId = setInterval(metroTick, getMetroIntervalMs());
    }
  });
});

document.querySelectorAll('#accentGroup .metro-opt').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('#accentGroup .metro-opt').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    metroState.accent = btn.dataset.accent;
  });
});

document.getElementById('chkSound').addEventListener('change', (e) => {
  metroState.soundOn = e.target.checked;
});

document.getElementById('chkVisual').addEventListener('change', (e) => {
  metroState.visualOn = e.target.checked;
});

// 键盘空格控制节拍器
document.addEventListener('keydown', (e) => {
  if (e.code === 'Space' && e.target === document.body) {
    e.preventDefault();
    if (metroState.running) stopMetronome();
    else startMetronome();
  }
});
```

- [ ] **Step 3: 在导航切换逻辑中加入节拍器页面处理**

在现有 `if (page === 'stats') renderStats();` 附近追加：
```javascript
if (page === 'metronome') { /* no init needed */ }
```

---

### Task 4: RPG评分引擎 + 增强打卡弹窗

**文件:** 修改 `index.html` 的 `<script>` 块

- [ ] **Step 1: 在数据层之后插入 RPG 数据函数**

```javascript
// ============================================================
// RPG DATA LAYER
// ============================================================
function loadRpgState() {
  try {
    const raw = localStorage.getItem(STORAGE_RPG);
    if (!raw) return defaultRpgState();
    const saved = JSON.parse(raw);
    // Merge with defaults in case of schema changes
    const def = defaultRpgState();
    return {
      scores: { ...def.scores, ...(saved.scores || {}) },
      lastPractice: { ...def.lastPractice, ...(saved.lastPractice || {}) },
      history: saved.history || []
    };
  } catch { return defaultRpgState(); }
}

function saveRpgState(state) {
  localStorage.setItem(STORAGE_RPG, JSON.stringify(state));
}

function getLevel(xp) {
  for (let i = LEVEL_THRESHOLDS.length - 1; i >= 0; i--) {
    if (xp >= LEVEL_THRESHOLDS[i]) return i;
  }
  return 0;
}

function getLevelName(xp) {
  return LEVEL_NAMES[getLevel(xp)];
}

function getXpToNext(xp) {
  const level = getLevel(xp);
  const nextIdx = level + 1;
  if (nextIdx >= LEVEL_THRESHOLDS.length) return Infinity;
  return LEVEL_THRESHOLDS[nextIdx] - xp;
}

function getLevelFloor(xp) {
  return LEVEL_THRESHOLDS[getLevel(xp)];
}

function calcXpGain(baseXP, minutes, currentLevel, selfRating) {
  const timeMult = Math.max(0.1, minutes / 30);
  // 高级段位衰减 (level >= 6 = 高级I或以上)
  const decayMult = currentLevel >= 6 ? 0.6 : 1.0;
  const ratingMult = Math.max(0.2, (selfRating || 5) / 5);
  return Math.round(baseXP * timeMult * decayMult * ratingMult);
}
```

- [ ] **Step 2: 在现有打卡弹窗HTML中增加标签和自评区域**

在现有 modal 的 `inputContent` textarea 之后，`mood-selector` 之前插入：
```html
<div class="form-group">
  <label>🏷️ 练琴标签（可多选）</label>
  <div class="tag-selector" id="tagSelector">
  </div>
</div>
<div class="form-group">
  <label>📊 六维自评（1-10分，根据参照表客观评价）</label>
  <div class="self-rating-grid" id="selfRatingGrid">
  </div>
</div>
<div class="form-group" style="text-align:right;">
  <button type="button" class="btn btn-ghost btn-sm" id="btnRefTable">📋 查看评分参照表</button>
</div>
```

- [ ] **Step 3: 在 CSS 中加入标签和自评样式**

```css
/* Tag selector */
.tag-selector { display: flex; flex-wrap: wrap; gap: 6px; }
.tag-chip {
  padding: 5px 10px;
  font-family: var(--font-ui);
  font-size: 0.7em;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  color: var(--text2);
  cursor: pointer;
  user-select: none;
}
.tag-chip.selected {
  background: var(--accent);
  color: #1a1a1a;
  border-color: var(--accent);
  font-weight: 700;
}

/* Self rating grid */
.self-rating-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}
.rating-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.75em;
  font-family: var(--font-ui);
}
.rating-row .dim-label { min-width: 70px; color: var(--text2); font-size: 0.7em; }
.rating-row input[type="range"] {
  flex: 1;
  height: 6px;
  -webkit-appearance: none;
  background: var(--pixel-dark);
  border: 1px solid var(--border);
}
.rating-row input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 16px; height: 16px;
  background: var(--accent);
  border: 1px solid #fff;
  cursor: pointer;
}
.rating-row .rating-val {
  width: 24px;
  text-align: center;
  font-family: var(--font-pixel);
  font-size: 0.65em;
  color: var(--accent);
}

.btn-sm { padding: 4px 10px; font-size: 0.65em; }
```

- [ ] **Step 4: 在 modal 的 JS 逻辑中加入标签/自评渲染和保存逻辑**

在现有 modal 相关JS中，`openModal` 函数内增加渲染标签和自评网格的调用：
```javascript
function renderTagSelector(existingTags) {
  const container = document.getElementById('tagSelector');
  container.innerHTML = TAG_OPTIONS.map(t => {
    const sel = existingTags && existingTags.includes(t.id) ? ' selected' : '';
    return `<span class="tag-chip${sel}" data-tag="${t.id}">${t.label}</span>`;
  }).join('');
  container.querySelectorAll('.tag-chip').forEach(chip => {
    chip.addEventListener('click', () => chip.classList.toggle('selected'));
  });
}

function renderSelfRatingGrid(existingRatings) {
  const container = document.getElementById('selfRatingGrid');
  container.innerHTML = DIM_KEYS.map(key => {
    const val = (existingRatings && existingRatings[key]) || 5;
    return `<div class="rating-row">
      <span class="dim-label">${DIM_EMOJI[key]} ${DIM_LABELS[key]}</span>
      <input type="range" min="1" max="10" value="${val}" data-dim="${key}">
      <span class="rating-val">${val}</span>
    </div>`;
  }).join('');
  container.querySelectorAll('input[type="range"]').forEach(slider => {
    slider.addEventListener('input', (e) => {
      const val = e.target.value;
      e.target.nextElementSibling.textContent = val;
    });
  });
}
```

在 `openModal` 函数内追加（在设置mood之后）：
```javascript
const existingTags = existing ? existing.tags : [];
const existingRatings = existing ? existing.ratings : null;
renderTagSelector(existingTags);
renderSelfRatingGrid(existingRatings);
```

- [ ] **Step 5: 修改保存逻辑，应用衰减并计算XP**

修改 `btnSave` 的 click handler（扩展现有逻辑）：
```javascript
document.getElementById('btnSave').addEventListener('click', () => {
  // ... 现有验证逻辑 ...
  
  // 收集标签
  const tags = [];
  document.querySelectorAll('#tagSelector .tag-chip.selected').forEach(c => {
    tags.push(c.dataset.tag);
  });
  
  // 收集自评
  const ratings = {};
  document.querySelectorAll('#selfRatingGrid input[type="range"]').forEach(s => {
    ratings[s.dataset.dim] = parseInt(s.value);
  });
  
  // 加载RPG状态并应用衰减
  let rpg = loadRpgState();
  rpg = applyDecay(rpg);
  
  // 计算本次XP
  const xpGained = {};
  for (const tag of tags) {
    const weights = TAG_WEIGHTS[tag];
    if (!weights) continue;
    for (const [dim, baseXP] of Object.entries(weights)) {
      const currentLevel = getLevel(rpg.scores[dim]);
      const gain = calcXpGain(baseXP, time, currentLevel, ratings[dim]);
      xpGained[dim] = (xpGained[dim] || 0) + gain;
      rpg.scores[dim] += gain;
      rpg.lastPractice[dim] = dateStr;
    }
  }
  
  // 如果没选标签，自评分数也给予少量基础经验
  if (tags.length === 0) {
    for (const dim of DIM_KEYS) {
      if (ratings[dim] && ratings[dim] >= 6) {
        const gain = calcXpGain(2, time, getLevel(rpg.scores[dim]), ratings[dim]);
        xpGained[dim] = (xpGained[dim] || 0) + gain;
        rpg.scores[dim] += gain;
        rpg.lastPractice[dim] = dateStr;
      }
    }
  }
  
  // 记录历史
  rpg.history.push({
    date: dateStr,
    tags,
    ratings,
    xpGained,
    time,
    mood
  });
  // 只保留最近365条记录
  if (rpg.history.length > 365) rpg.history = rpg.history.slice(-365);
  
  saveRpgState(rpg);
  
  // 保存原有打卡数据
  setCheckin(dateStr, { time, content, mood, tags, ratings, xpGained, updatedAt: new Date().toISOString() });
  
  // ... 现有后续逻辑 ...
  selectedDate = dateStr;
  closeModal();
  renderCalendar();
  
  // 显示结算画面
  showSettlement(dateStr, xpGained, rpg, tags, ratings);
});
```

- [ ] **Step 6: 实现衰减函数**

```javascript
function applyDecay(rpg) {
  const now = new Date();
  const todayStr = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')}`;
  
  for (const dim of DIM_KEYS) {
    const lastDate = rpg.lastPractice[dim];
    if (!lastDate) continue;
    // 跳过今天已练习的维度
    if (lastDate === todayStr) continue;
    
    const last = new Date(lastDate);
    const daysSince = (now - last) / (1000 * 60 * 60 * 24);
    if (daysSince > 5) {
      const decayDays = Math.floor(daysSince - 5);
      const floor = getLevelFloor(rpg.scores[dim]);
      for (let i = 0; i < decayDays; i++) {
        rpg.scores[dim] = Math.max(floor, Math.round(rpg.scores[dim] * 0.98));
      }
    }
  }
  return rpg;
}
```

---

### Task 5: 结算画面 + 参照表

**文件:** 修改 `index.html`

- [ ] **Step 1: 在 main 区域末尾加入结算面板 HTML**

```html
<!-- SETTLEMENT MODAL -->
<div class="modal-overlay" id="settlementOverlay">
  <div class="modal settlement-modal" id="settlementModal">
    <!-- Screen 1: Summary -->
    <div class="settlement-screen" id="settlementSummary">
      <div class="settlement-title">★ 练 琴 完 成 ! ★</div>
      <div class="settlement-xp-list" id="settlementXpList"></div>
      <div class="settlement-total">总获得 <span id="settlementTotalXp">0</span> XP</div>
      <div class="settlement-hint">▼ 下滑查看详情</div>
    </div>

    <!-- Screen 2: RPG Animation -->
    <div class="settlement-screen" id="settlementRpg">
      <div class="rpg-character" id="rpgCharacter">🎸</div>
      <div class="rpg-road" id="rpgRoad"></div>
      <div class="rpg-levelup" id="rpgLevelUp" style="display:none;">
        <span class="levelup-text">⚡ LEVEL UP! ⚡</span>
      </div>
      <div class="settlement-bars" id="settlementBars"></div>
      <div class="settlement-hint">▼ 查看建议</div>
    </div>

    <!-- Screen 3: Advice -->
    <div class="settlement-screen" id="settlementAdvice">
      <div class="advice-box" id="adviceBox"></div>
      <div class="btn-row" style="margin-top:14px; justify-content:center;">
        <button class="btn btn-ghost btn-sm" id="btnRefTable2">📋 查看评分参照表</button>
        <button class="btn btn-primary" id="btnCloseSettlement">👍 知道了</button>
      </div>
    </div>
  </div>
</div>

<!-- REFERENCE TABLE MODAL -->
<div class="modal-overlay" id="refTableOverlay">
  <div class="modal" style="max-width:640px; max-height:80vh; overflow-y:auto;">
    <h3 style="margin-bottom:16px;">📋 六维评分参照表</h3>
    <div id="refTableContent"></div>
    <div class="btn-row" style="justify-content:center;margin-top:16px;">
      <button class="btn btn-primary" id="btnCloseRefTable">关闭</button>
    </div>
  </div>
</div>
```

- [ ] **Step 2: 在 CSS 中加入结算画面样式**

```css
/* ===== Settlement Screen ===== */
.settlement-modal { width: 480px; max-width: 92vw; }
.settlement-title {
  font-family: var(--font-pixel);
  font-size: 0.85em;
  color: var(--accent);
  text-align: center;
  margin-bottom: 16px;
  text-shadow: 2px 2px 0px rgba(0,0,0,0.5);
}
.settlement-xp-list {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 6px;
  margin-bottom: 14px;
}
.xp-row {
  font-family: var(--font-ui);
  font-size: 0.7em;
  padding: 4px 8px;
  background: var(--pixel-dark);
  border: 1px solid var(--border);
  display: flex;
  justify-content: space-between;
}
.xp-row .dim-name { color: var(--text2); }
.xp-row .dim-xp { color: var(--accent); font-weight: 700; }
.xp-row.zero { opacity: 0.4; }
.settlement-total {
  text-align: center;
  font-family: var(--font-pixel);
  font-size: 0.7em;
  color: var(--green);
  margin-bottom: 10px;
}
.settlement-hint {
  text-align: center;
  font-size: 0.6em;
  color: var(--text2);
  margin-top: 10px;
  animation: blink 1.5s steps(2) infinite;
}
@keyframes blink { 50% { opacity: 0.3; } }

/* RPG Road */
.rpg-character {
  font-size: 2em;
  text-align: center;
  animation: walk 0.4s steps(4) infinite;
}
@keyframes walk {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-4px); }
}
.rpg-road {
  height: 4px;
  background: var(--border);
  margin: 8px 0 16px;
  position: relative;
}
.rpg-road::after {
  content: '';
  position: absolute;
  left: 0; top: 0; height: 100%;
  width: var(--road-progress, 0%);
  background: var(--accent);
  transition: width 1s steps(30);
}
.rpg-levelup {
  text-align: center;
  padding: 10px;
  margin-bottom: 10px;
  animation: levelupPulse 0.4s steps(4) infinite;
}
.levelup-text {
  font-family: var(--font-pixel);
  font-size: 0.8em;
  color: #ffe040;
}
@keyframes levelupPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.08); }
}

/* Settlement bars */
.settlement-bars { margin-bottom: 12px; }
.settlement-bar-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 6px;
  font-size: 0.7em;
  font-family: var(--font-ui);
}
.settlement-bar-row .bar-label {
  width: 65px;
  text-align: right;
  color: var(--text2);
  font-size: 0.65em;
}
.settlement-bar-row .bar-wrap {
  flex: 1;
  height: 12px;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  position: relative;
}
.settlement-bar-row .bar-fill {
  height: 100%;
  background: repeating-linear-gradient(90deg, var(--green) 0px, var(--green) 6px, #389838 6px, #389838 8px);
  transition: width 0.6s steps(20);
}
.settlement-bar-row .bar-lvl {
  width: 55px;
  font-size: 0.6em;
  color: var(--accent);
}

/* Advice */
.advice-box {
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  padding: 16px;
  font-family: var(--font-ui);
  font-size: 0.8em;
  line-height: 1.7;
  color: var(--text);
}
.advice-box .advice-highlight {
  color: var(--accent);
  font-weight: 700;
}
```

- [ ] **Step 3: 在 JS 中实现 showSettlement 函数**

```javascript
function showSettlement(dateStr, xpGained, rpg, tags, ratings) {
  const overlay = document.getElementById('settlementOverlay');
  const totalXP = Object.values(xpGained).reduce((a, b) => a + b, 0);
  
  // Screen 1: Summary
  document.getElementById('settlementXpList').innerHTML = DIM_KEYS.map(key => {
    const gained = xpGained[key] || 0;
    const cls = gained === 0 ? ' zero' : '';
    return `<div class="xp-row${cls}">
      <span class="dim-name">${DIM_EMOJI[key]} ${DIM_LABELS[key]}</span>
      <span class="dim-xp">${gained > 0 ? '+' : ''}${gained} XP</span>
    </div>`;
  }).join('');
  document.getElementById('settlementTotalXp').textContent = totalXP;
  
  // Screen 2: RPG bars
  let hasLevelUp = false;
  document.getElementById('settlementBars').innerHTML = DIM_KEYS.map(key => {
    const xp = rpg.scores[key];
    const level = getLevel(xp);
    const prevLevel = getLevel((xp - (xpGained[key] || 0)));
    const levelName = LEVEL_NAMES[level];
    const nextThresh = LEVEL_THRESHOLDS[level + 1] || LEVEL_THRESHOLDS[LEVEL_THRESHOLDS.length - 1];
    const prevThresh = LEVEL_THRESHOLDS[level];
    const pct = nextThresh === Infinity ? 100 : Math.min(100, Math.round((xp - prevThresh) / (nextThresh - prevThresh) * 100));
    
    if (level > prevLevel && prevLevel < LEVEL_NAMES.length - 1) hasLevelUp = true;
    
    return `<div class="settlement-bar-row">
      <span class="bar-label">${DIM_EMOJI[key]} ${DIM_LABELS[key].slice(0, 4)}</span>
      <div class="bar-wrap"><div class="bar-fill" style="width:${pct}%"></div></div>
      <span class="bar-lvl">${levelName}</span>
    </div>`;
  }).join('');
  
  if (hasLevelUp) {
    document.getElementById('rpgLevelUp').style.display = 'block';
    // 播放8-bit音效
    playLevelUpSound();
  } else {
    document.getElementById('rpgLevelUp').style.display = 'none';
  }
  
  // Road progress (取六维平均进度作为整体进度)
  const avgProgress = DIM_KEYS.reduce((sum, key) => {
    const xp = rpg.scores[key];
    return sum + getLevel(xp) / (LEVEL_NAMES.length - 1);
  }, 0) / DIM_KEYS.length;
  document.getElementById('rpgRoad').style.setProperty('--road-progress', Math.round(avgProgress * 100) + '%');
  
  // Screen 3: Advice
  document.getElementById('adviceBox').innerHTML = generateAdvice(rpg, tags, ratings);
  
  overlay.classList.add('show');
  // Reset scroll to top
  document.getElementById('settlementModal').scrollTop = 0;
}

function generateAdvice(rpg, tags, ratings) {
  const lines = [];
  
  // 检查衰减警告
  const now = new Date();
  for (const dim of DIM_KEYS) {
    const lastDate = rpg.lastPractice[dim];
    if (!lastDate) continue;
    const daysSince = (now - new Date(lastDate)) / (1000 * 60 * 60 * 24);
    if (daysSince >= 4 && daysSince <= 5) {
      lines.push(`⚠️ <span class="advice-highlight">${DIM_LABELS[dim]}</span> 已${Math.floor(daysSince)}天未练，即将衰减！`);
    } else if (daysSince > 5) {
      lines.push(`🔻 <span class="advice-highlight">${DIM_LABELS[dim]}</span> 正在衰减中，快安排练习！`);
    }
  }
  
  // 检查快升级的维度
  for (const dim of DIM_KEYS) {
    const xp = rpg.scores[dim];
    const toNext = getXpToNext(xp);
    if (toNext !== Infinity && toNext <= 200) {
      const nextName = LEVEL_NAMES[getLevel(xp) + 1];
      lines.push(`⭐ <span class="advice-highlight">${DIM_LABELS[dim]}</span> 距 ${nextName} 还差 ${toNext} XP！集中突破！`);
    }
  }
  
  // 如果没有警告，给鼓励
  if (lines.length === 0) {
    lines.push('👍 各方面均衡发展，继续保持节奏！');
  }
  
  // 基于今天练习内容的建议
  if (tags.length === 0) {
    lines.push('💡 下次试试选择练琴标签，可以获得更精准的经验加成哦。');
  }
  
  return lines.join('<br>');
}

function playLevelUpSound() {
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const notes = [523, 659, 784, 1047]; // C5 E5 G5 C6
    notes.forEach((freq, i) => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.type = 'square';
      osc.frequency.value = freq;
      const t = ctx.currentTime + i * 0.12;
      gain.gain.setValueAtTime(0.15, t);
      gain.gain.exponentialRampToValueAtTime(0.001, t + 0.1);
      osc.start(t);
      osc.stop(t + 0.1);
    });
  } catch(e) { /* 静默失败 */ }
}
```

- [ ] **Step 4: 实现参照表内容**

```javascript
const REFERENCE_TABLE = {
  tech: {
    title: '技术基本功',
    levels: {
      初级: {
        standard: '能弹完C大调音阶（两个八度），速度80bpm八分音符',
        anchor: '手指能准确落在品上，换把无明显断音',
        benchmark: 'Paul Chambers - So What 中的 Walking 段落（慢速版）'
      },
      中级: {
        standard: '所有大调音阶、自然小调音阶、五声音阶，速度120bpm十六分音符',
        anchor: '各调音阶能在E/A弦两个把位上流畅切换，不卡顿',
        benchmark: 'Jaco Pastorius - Come On, Come Over 前奏（原速80%）'
      },
      高级: {
        standard: '音阶模进（三度、四度、六度），各调式音阶，速度140bpm+',
        anchor: '能在任意把位即兴跑模进，不依赖肌肉记忆',
        benchmark: 'Marcus Miller - Power 中的贝斯Solo段落'
      }
    }
  },
  rhythm: {
    title: '节奏/时值感',
    levels: {
      初级: {
        standard: '开着节拍器，弹四分音符能稳住',
        anchor: '节拍器60bpm，录30秒四分音符，回听偏差不超过1/32拍',
        benchmark: 'Michael Jackson - Billie Jean 贝斯线（几乎全是八分音符）'
      },
      中级: {
        standard: '开着节拍器，弹附点、切分、休止符都能卡准',
        anchor: '节拍器80bpm，弹附点节奏连续8小节，录音回听无抢拍拖拍',
        benchmark: 'Rocco Prestia (Tower of Power) - What Is Hip（注意切分和休止符）'
      },
      高级: {
        standard: '能主动创造"lay back"或"push"的感觉，且保持稳定',
        anchor: '能在同一BPM下，同一段乐句故意弹出三种时值感觉（正拍/lay back/push）',
        benchmark: 'Ray Brown 的任何Oscar Peterson三重奏录音'
      }
    }
  },
  ear: {
    title: '听力（耳力）',
    levels: {
      初级: {
        standard: '能听出大调和小调的区别',
        anchor: '随机放10首流行歌，能正确说出每首是大调还是小调（准确率>80%）',
        benchmark: 'The Beatles - Let It Be（大调） vs While My Guitar Gently Weeps（小调）'
      },
      中级: {
        standard: '能听出II-V-I进行，能在贝斯上摸出简单旋律（如儿歌）',
        anchor: '放一首没听过的简单流行歌，10分钟内摸出贝斯线的根音进行',
        benchmark: 'Autumn Leaves 多个版本的贝斯线比较（Cannonball / Bill Evans）'
      },
      高级: {
        standard: '能听出和弦的延伸音（9、11、13），能扒下复杂的贝斯线',
        anchor: '能扒出Jaco或Marcus Miller的Solo段落，标注每个音的和弦功能',
        benchmark: 'Jaco Pastorius - Portrait of Tracy（音程跨越极大）'
      }
    }
  },
  harmony: {
    title: '和声/乐理',
    levels: {
      初级: {
        standard: '知道大/小三和弦的组成音，能弹出根三五四音',
        anchor: '看向任意一个和弦名，3秒内弹出其根音、三音、五音、八度音',
        benchmark: '任何流行歌的和弦进行（如 I-V-vi-IV）'
      },
      中级: {
        standard: '知道7和弦（maj7、7、m7、m7b5）的组成音，能在一个调内快速找到',
        anchor: '随机给一个Dm7b5，立刻弹出四个组成音D-F-Ab-C',
        benchmark: 'All of Me 或 Blue Bossa 的和弦分析'
      },
      高级: {
        standard: '理解延伸音、替代和弦、调式音阶，能分析和弦功能',
        anchor: '能在爵士Lead Sheet上标记每个和弦的可用音阶和替代可能性',
        benchmark: 'John Coltrane - Giant Steps 的和声分析'
      }
    }
  },
  improv: {
    title: '即兴/创造力',
    levels: {
      初级: {
        standard: '只会弹根音或照谱弹',
        anchor: '能在Blues 12小节上稳定走根音，偶尔在第四拍加半音趋近',
        benchmark: '最简单的Blues Walking Bass（参考Paul Chambers早期录音）'
      },
      中级: {
        standard: '能基于和弦音做简单的Fill，加一点花',
        anchor: '在一个12小节Blues里，能即兴2个Chorus不重复，使用和弦音+经过音',
        benchmark: 'Ray Brown - That\'s All 中的Solo段落'
      },
      高级: {
        standard: '能围绕旋律做即兴变奏，有自己的句子',
        anchor: '同一首曲子连续即兴4遍，每遍都有不同的句子和动机发展',
        benchmark: 'Niels-Henning Ørsted Pedersen 的任何Solo录音'
      }
    }
  },
  repertoire: {
    title: '曲库/实战',
    levels: {
      初级: {
        standard: '能背下5首流行歌的贝斯线',
        anchor: '不看谱子弹完5首歌，录音与录音室版本对比无明显错误',
        benchmark: '陶喆 - 小镇姑娘 / Billie Jean / 简单流行歌'
      },
      中级: {
        standard: '能背下15首不同风格的歌（流行、摇滚、Funk）',
        anchor: '能在一个jam session里不看谱走完5首不同风格的曲子',
        benchmark: '能跟完一场小型jam session（比如酒吧开放麦）'
      },
      高级: {
        standard: '能不用谱子上台跟乐队走完一场演出（40分钟+）',
        anchor: '陌生乐队给和弦走向就能跟，不需要提前排练完整曲目',
        benchmark: '职业贝斯手的基本要求：靠Lead Sheet或耳朵就能演出'
      }
    }
  }
};

function renderRefTable() {
  const container = document.getElementById('refTableContent');
  let html = '';
  for (const [dimKey, dimData] of Object.entries(REFERENCE_TABLE)) {
    html += `<h3 style="margin-top:16px;color:var(--accent);">${DIM_EMOJI[dimKey]} ${dimData.title}</h3>`;
    for (const [level, data] of Object.entries(dimData.levels)) {
      html += `<div class="plan-section" style="margin-bottom:6px;">
        <div class="plan-section-header" style="padding:10px 14px;">
          <span style="font-size:0.8em;">【${level}】${data.standard.slice(0, 40)}...</span>
          <span class="arrow">▶</span>
        </div>
        <div class="plan-section-body">
          <p><strong>可验证标准：</strong>${data.anchor}</p>
          <p><strong>标杆参照：</strong>${data.benchmark}</p>
        </div>
      </div>`;
    }
  }
  container.innerHTML = html;
  // 绑定折叠事件
  container.querySelectorAll('.plan-section-header').forEach(header => {
    header.addEventListener('click', () => header.parentElement.classList.toggle('open'));
  });
}
```

- [ ] **Step 5: 绑定结算画面和参照表的事件**

在 INIT 中追加：
```javascript
// Settlement close
document.getElementById('btnCloseSettlement').addEventListener('click', () => {
  document.getElementById('settlementOverlay').classList.remove('show');
});
document.getElementById('settlementOverlay').addEventListener('click', (e) => {
  if (e.target === e.currentTarget) document.getElementById('settlementOverlay').classList.remove('show');
});

// Reference table
document.getElementById('btnRefTable').addEventListener('click', () => {
  renderRefTable();
  document.getElementById('refTableOverlay').classList.add('show');
});
document.getElementById('btnRefTable2').addEventListener('click', () => {
  renderRefTable();
  document.getElementById('refTableOverlay').classList.add('show');
});
document.getElementById('btnCloseRefTable').addEventListener('click', () => {
  document.getElementById('refTableOverlay').classList.remove('show');
});
document.getElementById('refTableOverlay').addEventListener('click', (e) => {
  if (e.target === e.currentTarget) document.getElementById('refTableOverlay').classList.remove('show');
});
```

---

### Task 6: 统计页面升级 — RPG角色面板

**文件:** 修改 `index.html`

- [ ] **Step 1: 在 stats 页面头部加入 RPG 角色面板**

在 `page-stats` 的 `h2` 之后、`stat-grid` 之前插入：
```html
<div class="rpg-panel" id="rpgPanel">
  <h3 style="margin-top:0;">🎮 贝斯手属性面板</h3>
  <div class="rpg-bars" id="rpgBars"></div>
  <div style="text-align:right;margin-top:6px;">
    <button class="btn btn-ghost btn-sm" id="btnRefTable3">📋 参照表</button>
  </div>
</div>
```

- [ ] **Step 2: 在 CSS 中加入 RPG 面板样式**

```css
/* RPG Panel */
.rpg-panel {
  background: var(--surface);
  border: 3px solid var(--border);
  box-shadow: var(--shadow);
  padding: 20px;
  margin-bottom: 18px;
}
.rpg-bars { display: flex; flex-direction: column; gap: 8px; }
.rpg-bar-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: var(--font-ui);
  font-size: 0.72em;
}
.rpg-bar-row .rpg-dim-label {
  width: 85px;
  text-align: right;
  color: var(--text);
  font-size: 0.9em;
}
.rpg-bar-row .rpg-bar-wrap {
  flex: 1;
  height: 16px;
  background: var(--pixel-dark);
  border: 2px solid var(--border);
  position: relative;
}
.rpg-bar-row .rpg-bar-fill {
  height: 100%;
  transition: width 0.5s steps(20);
}
.rpg-bar-row .rpg-bar-fill.tech { background: repeating-linear-gradient(90deg, #e04848 0px, #e04848 6px, #b83838 6px, #b83838 8px); }
.rpg-bar-row .rpg-bar-fill.rhythm { background: repeating-linear-gradient(90deg, #58a8d8 0px, #58a8d8 6px, #3888b8 6px, #3888b8 8px); }
.rpg-bar-row .rpg-bar-fill.ear { background: repeating-linear-gradient(90deg, #58b868 0px, #58b868 6px, #389848 6px, #389848 8px); }
.rpg-bar-row .rpg-bar-fill.harmony { background: repeating-linear-gradient(90deg, #c058d8 0px, #c058d8 6px, #9038a8 6px, #9038a8 8px); }
.rpg-bar-row .rpg-bar-fill.improv { background: repeating-linear-gradient(90deg, #f0b030 0px, #f0b030 6px, #c08020 6px, #c08020 8px); }
.rpg-bar-row .rpg-bar-fill.repertoire { background: repeating-linear-gradient(90deg, #e08050 0px, #e08050 6px, #b06038 6px, #b06038 8px); }
.rpg-bar-row .rpg-lvl {
  width: 50px;
  font-size: 0.8em;
  color: var(--accent);
  font-family: var(--font-pixel);
}
.rpg-bar-row .rpg-xp {
  width: 50px;
  font-size: 0.65em;
  color: var(--text2);
  text-align: right;
}
```

- [ ] **Step 3: 在 renderStats 中追加 RPG 面板渲染**

在 `renderStats()` 函数末尾追加：
```javascript
renderRpgPanel();
```

并实现 `renderRpgPanel`：
```javascript
function renderRpgPanel() {
  let rpg = loadRpgState();
  rpg = applyDecay(rpg);
  saveRpgState(rpg);
  
  const container = document.getElementById('rpgBars');
  container.innerHTML = DIM_KEYS.map(key => {
    const xp = rpg.scores[key];
    const level = getLevel(xp);
    const levelName = LEVEL_NAMES[level];
    const nextThresh = LEVEL_THRESHOLDS[level + 1] || LEVEL_THRESHOLDS[LEVEL_THRESHOLDS.length - 1];
    const prevThresh = LEVEL_THRESHOLDS[level];
    const pct = nextThresh === Infinity ? 100 : Math.round((xp - prevThresh) / (nextThresh - prevThresh) * 100);
    const toNext = getXpToNext(xp);
    
    return `<div class="rpg-bar-row">
      <span class="rpg-dim-label">${DIM_EMOJI[key]} ${DIM_LABELS[key]}</span>
      <div class="rpg-bar-wrap">
        <div class="rpg-bar-fill ${key}" style="width:${pct}%"></div>
      </div>
      <span class="rpg-lvl">${levelName}</span>
      <span class="rpg-xp">${toNext === Infinity ? 'MAX' : toNext + '→'}</span>
    </div>`;
  }).join('');
}
```

---

### Task 7: 最终联调

**文件:** 修改 `index.html`

- [ ] **Step 1: 确保导航切换时刷新 RPG 面板**

在导航切换逻辑中追加：
```javascript
if (page === 'stats') { renderRpgPanel(); renderStats(); }
if (page === 'metronome') { /* stop metronome on nav away handled by page hide */ }
```

- [ ] **Step 2: 检查页面离开时节拍器停止**

在导航切换逻辑最前面加入：
```javascript
if (metroState.running) stopMetronome();
```

- [ ] **Step 3: 键盘快捷键空格只在非输入状态下触发节拍器**

现有代码已处理（`e.target === document.body`）。

- [ ] **Step 4: 打开 index.html 进行完整手工测试**

测试清单：
1. 像素主题：所有页面显示像素字体和边框
2. 节拍器：START/STOP、BPM滑块、细分切换、重音切换、声音/视觉开关、Tap Tempo
3. 打卡：选择标签、拖拽自评滑条、保存后弹出结算画面
4. 结算：三屏滚动查看、升级动画、建议文案
5. 参照表：从结算页和统计页均可打开
6. 统计页：RPG属性面板显示六维经验和等级
7. 衰减：修改localStorage模拟超过5天未练，确认经验掉2%
8. 空格键：在非输入元素上按空格启动/停止节拍器
9. 关闭结算后日历刷新
10. 连续打卡后streak正常更新
