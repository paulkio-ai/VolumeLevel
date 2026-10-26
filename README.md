# VolumeLevel
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Voice Level Coach</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,600;12..96,800&family=Atkinson+Hyperlegible:wght@400;700&family=Atkinson+Hyperlegible+Mono:wght@400;600&display=swap">
<style>
/* Layout: one projector-friendly stage — status word on top, segmented meter across, session tally below; settings fold out underneath. */
:root {
  --bg: #eef2f6;
  --surface: #ffffff;
  --ink: #132030;
  --muted: #576577;
  --line: #d3dbe4;
  --track: #e2e8ef;
  --soft: #2f74d6;
  --right: #17905a;
  --loud: #d93d27;
  --idle: #7a8796;
  --focus: #2f74d6;
  --display: "Bricolage Grotesque", "Segoe UI", system-ui, sans-serif;
  --body: "Atkinson Hyperlegible", "Segoe UI", system-ui, sans-serif;
  --mono: "Atkinson Hyperlegible Mono", ui-monospace, "SF Mono", Consolas, monospace;
  color-scheme: light;
}
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --bg: #0e141b; --surface: #161f2a; --ink: #edf2f8; --muted: #95a3b4; --line: #283545;
    --track: #212c39; --soft: #5c9cf2; --right: #33c07e; --loud: #ff6b55; --idle: #7f8c9c; --focus: #7fb2ff;
    color-scheme: dark;
  }
}
:root[data-theme="dark"] {
  --bg: #0e141b; --surface: #161f2a; --ink: #edf2f8; --muted: #95a3b4; --line: #283545;
  --track: #212c39; --soft: #5c9cf2; --right: #33c07e; --loud: #ff6b55; --idle: #7f8c9c; --focus: #7fb2ff;
  color-scheme: dark;
}
* { box-sizing: border-box; }
html, body { margin: 0; }
body {
  background: var(--bg); color: var(--ink);
  font: 16px/1.5 var(--body);
  padding-block: 20px 40px;
  padding-left: max(16px, env(safe-area-inset-left, 0px));
  padding-right: max(16px, env(safe-area-inset-right, 0px));
}
.wrap { max-width: 1100px; margin: 0 auto; display: grid; gap: 20px; }
header { display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between; gap: 12px; }
h1 { font: 800 1.5rem/1.1 var(--display); margin: 0; letter-spacing: -0.01em; }
h1 small { display: block; font: 400 0.9rem/1.4 var(--body); color: var(--muted); letter-spacing: 0; margin-top: 4px; }
.actions { display: flex; flex-wrap: wrap; gap: 8px; }
button {
  font: 700 0.95rem/1 var(--body); color: var(--ink); background: var(--surface);
  border: 1.5px solid var(--line); border-radius: 10px; padding: 11px 16px; cursor: pointer;
}
button:hover { border-color: var(--muted); }
button:focus-visible, input:focus-visible { outline: 3px solid var(--focus); outline-offset: 2px; }
button.primary { background: var(--ink); color: var(--bg); border-color: var(--ink); }
button[aria-pressed="true"] { background: var(--ink); color: var(--bg); border-color: var(--ink); }

/* Stage */
.stage {
  --zone: var(--idle);
  background: color-mix(in srgb, var(--zone) 12%, var(--surface));
  border: 2px solid color-mix(in srgb, var(--zone) 45%, var(--line));
  border-radius: 20px; padding: clamp(20px, 4vw, 40px);
  display: grid; gap: clamp(18px, 3vw, 30px);
  transition: background-color .25s, border-color .25s;
}
.stage[data-zone="soft"] { --zone: var(--soft); }
.stage[data-zone="right"] { --zone: var(--right); }
.stage[data-zone="loud"] { --zone: var(--loud); }
.status { display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between; gap: 16px 28px; }
.status-main { display: flex; align-items: center; gap: clamp(14px, 2.5vw, 28px); min-width: 0; }
.glyph {
  flex: none; width: clamp(64px, 10vw, 112px); aspect-ratio: 1; max-width: 100%;
  border-radius: 50%; background: var(--zone); color: var(--surface);
  display: grid; place-items: center; transition: background-color .25s;
}
.glyph svg { width: 56%; height: 56%; }
.word { font: 800 clamp(2.6rem, 8.5vw, 6rem)/0.95 var(--display); letter-spacing: -0.02em; color: var(--zone); text-wrap: balance; transition: color .25s; }
.hint { font-size: clamp(1rem, 2vw, 1.35rem); color: var(--ink); margin-top: 8px; }
.readout { text-align: right; font-family: var(--mono); font-variant-numeric: tabular-nums; }
.readout b { display: block; font-size: clamp(2.2rem, 6vw, 3.8rem); line-height: 1; font-weight: 600; }
.readout span { color: var(--muted); font-size: 0.85rem; letter-spacing: 0.06em; text-transform: uppercase; }

/* Meter */
.meter { display: grid; gap: 8px; }
.bar { position: relative; display: grid; grid-template-columns: repeat(60, 1fr); gap: 3px; height: clamp(56px, 9vw, 92px); }
.seg { border-radius: 3px; background: color-mix(in srgb, var(--c) 14%, var(--track)); transition: background-color 60ms; }
.seg.on { background: var(--c); }
.seg[data-z="soft"] { --c: var(--soft); }
.seg[data-z="right"] { --c: var(--right); }
.seg[data-z="loud"] { --c: var(--loud); }
.peak { position: absolute; top: -6px; bottom: -6px; width: 4px; margin-left: -2px; border-radius: 2px; background: var(--ink); opacity: 0; transition: left 80ms linear, opacity .3s; }
.peak.show { opacity: .85; }
.scale { position: relative; height: 22px; font: 0.78rem/1 var(--mono); color: var(--muted); font-variant-numeric: tabular-nums; }
.scale span { position: absolute; top: 4px; transform: translateX(-50%); }
.scale span:first-child { transform: none; }
.scale span:last-child { transform: translateX(-100%); }
.bands { display: flex; flex-wrap: wrap; gap: 8px 20px; font-size: 0.92rem; color: var(--muted); }
.bands i { display: inline-block; width: 12px; height: 12px; border-radius: 3px; margin-right: 6px; vertical-align: -1px; }

/* Tally */
.tally { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; }
.chip { background: var(--surface); border: 1.5px solid var(--line); border-radius: 14px; padding: 14px 16px; display: grid; gap: 6px; }
.chip .lab { font-size: 0.78rem; font-weight: 700; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); display: flex; align-items: center; gap: 8px; }
.chip .lab i { width: 10px; height: 10px; border-radius: 50%; }
.chip b { font: 600 1.7rem/1 var(--mono); font-variant-numeric: tabular-nums; }
.chip .track { height: 6px; border-radius: 3px; background: var(--track); overflow: hidden; }
.chip .track div { height: 100%; width: 0; border-radius: 3px; transition: width .3s; }
.tally-head { display: flex; flex-wrap: wrap; justify-content: space-between; align-items: baseline; gap: 8px; }
.tally-head h2, .settings h2 { font: 700 1.05rem/1.2 var(--display); margin: 0; }
.tally-head p { margin: 0; color: var(--muted); font-size: 0.92rem; font-variant-numeric: tabular-nums; }

/* Settings */
.settings { background: var(--surface); border: 1.5px solid var(--line); border-radius: 16px; padding: 20px; display: grid; gap: 18px; }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(230px, 1fr)); gap: 18px 28px; }
.field { display: grid; gap: 6px; min-width: 0; }
.field label { font-weight: 700; display: flex; justify-content: space-between; gap: 8px; }
.field label output { font-family: var(--mono); font-weight: 400; font-variant-numeric: tabular-nums; }
.field p { margin: 0; font-size: 0.88rem; color: var(--muted); }
input[type="range"] { width: 100%; accent-color: var(--ink); }
.calib { display: flex; flex-wrap: wrap; align-items: center; gap: 12px; padding-top: 14px; border-top: 1px solid var(--line); }
.calib p { margin: 0; flex: 1 1 320px; color: var(--muted); font-size: 0.92rem; max-width: 65ch; }
.msg { margin: 0; padding: 12px 16px; border-radius: 12px; background: color-mix(in srgb, var(--loud) 12%, var(--surface)); border: 1.5px solid color-mix(in srgb, var(--loud) 40%, var(--line)); }
.note { color: var(--muted); font-size: 0.85rem; max-width: 75ch; margin: 0; }
@media (prefers-reduced-motion: reduce) { * { transition: none !important; } }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>Voice Level Coach<small>Practice presenting loud enough for the whole room</small></h1>
    <div class="actions">
      <button id="startBtn" class="primary" type="button">Start listening</button>
      <button id="demoBtn" type="button" aria-pressed="false">Demo</button>
      <button id="resetBtn" type="button">Reset tally</button>
      <button id="fsBtn" type="button">Full screen</button>
    </div>
  </header>

  <p class="msg" id="msg" hidden></p>

  <section class="stage" id="stage" data-zone="idle" aria-live="polite">
    <div class="status">
      <div class="status-main">
        <div class="glyph" id="glyph" aria-hidden="true"></div>
        <div>
          <div class="word" id="word">Ready</div>
          <div class="hint" id="hint">Press Start listening, then begin your presentation.</div>
        </div>
      </div>
      <div class="readout" aria-hidden="true"><b id="db">--</b><span>approx. dB</span></div>
    </div>

    <div class="meter">
      <div class="bar" id="bar"><div class="peak" id="peak"></div></div>
      <div class="scale" id="scale"></div>
      <div class="bands" id="bands"></div>
    </div>
  </section>

  <section class="tally-wrap" style="display:grid;gap:12px">
    <div class="tally-head">
      <h2>This practice run</h2>
      <p id="talkTime">Speaking time 0:00</p>
    </div>
    <div class="tally">
      <div class="chip"><div class="lab"><i style="background:var(--soft)"></i>Too soft</div><b id="pSoft">0%</b><div class="track"><div id="tSoft" style="background:var(--soft)"></div></div></div>
      <div class="chip"><div class="lab"><i style="background:var(--right)"></i>Just right</div><b id="pRight">0%</b><div class="track"><div id="tRight" style="background:var(--right)"></div></div></div>
      <div class="chip"><div class="lab"><i style="background:var(--loud)"></i>Too loud</div><b id="pLoud">0%</b><div class="track"><div id="tLoud" style="background:var(--loud)"></div></div></div>
    </div>
  </section>

  <section class="settings" aria-labelledby="setH">
    <h2 id="setH">Settings</h2>
    <div class="grid">
      <div class="field">
        <label for="soft">"Too soft" below <output id="softOut"></output></label>
        <input type="range" id="soft" min="40" max="80" step="1">
        <p>Raise this for a big room or a noisy class.</p>
      </div>
      <div class="field">
        <label for="loud">"Too loud" above <output id="loudOut"></output></label>
        <input type="range" id="loud" min="55" max="95" step="1">
        <p>Speech louder than this starts to sound like shouting.</p>
      </div>
      <div class="field">
        <label for="hold">Reaction time <output id="holdOut"></output></label>
        <input type="range" id="hold" min="0.4" max="3" step="0.1">
        <p>Longer ignores the short pauses between words.</p>
      </div>
      <div class="field">
        <label for="offset">Mic sensitivity <output id="offsetOut"></output></label>
        <input type="range" id="offset" min="60" max="120" step="1">
        <p>Adjusts how this microphone's signal maps to dB. Calibrate sets it for you.</p>
      </div>
    </div>
    <div class="calib">
      <button id="calBtn" type="button">Calibrate (3 seconds)</button>
      <p id="calText">Stand where the presenter will stand and speak at a good presentation volume. The app sets "Just right" around that level.</p>
    </div>
  </section>

  <p class="note">Readings are approximate. Browsers can't measure true sound pressure, and every microphone differs, so calibrate once per device and room. Default range for a classroom presentation voice: about 55–75 dB. Settings are saved on this computer.</p>
</div>

<script>
(() => {
  const MIN = 30, MAX = 100, SEGS = 60;
  const ICONS = {
    idle: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="3" width="6" height="11" rx="3"/><path d="M5 11a7 7 0 0 0 14 0M12 18v3"/></svg>',
    soft: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.8" stroke-linecap="round" stroke-linejoin="round"><path d="M12 19V5M5 12l7-7 7 7"/></svg>',
    right: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.8" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12.5l4.5 4.5L19 7"/></svg>',
    loud: '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.8" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14M5 12l7 7 7-7"/></svg>'
  };
  const TEXT = {
    idle: ['Ready', 'Press Start listening, then begin your presentation.'],
    wait: ['Listening…', 'Start speaking when you are ready.'],
    soft: ['Too soft', 'Speak up so the back row can hear you.'],
    right: ['Just right', 'Great volume. Keep it here.'],
    loud: ['Too loud', 'Bring it down a little.']
  };

  const $ = id => document.getElementById(id);
  const defaults = { soft: 55, loud: 75, hold: 1.0, offset: 90 };
  let cfg = { ...defaults };
  try { Object.assign(cfg, JSON.parse(localStorage.getItem('vlc-settings') || '{}')); } catch (e) {}
  const save = () => { try { localStorage.setItem('vlc-settings', JSON.stringify(cfg)); } catch (e) {} };

  // Build meter segments, scale, legend
  const bar = $('bar'), peakEl = $('peak');
  const segs = [];
  for (let i = 0; i < SEGS; i++) { const s = document.createElement('div'); s.className = 'seg'; bar.insertBefore(s, peakEl); segs.push(s); }
  const segDb = i => MIN + (i + 0.5) * (MAX - MIN) / SEGS;
  const pct = db => ((Math.min(MAX, Math.max(MIN, db)) - MIN) / (MAX - MIN)) * 100;
  const scale = $('scale');
  for (let d = MIN; d <= MAX; d += 10) { const s = document.createElement('span'); s.style.left = pct(d) + '%'; s.textContent = d; scale.appendChild(s); }

  function paintZones() {
    segs.forEach((s, i) => { const d = segDb(i); s.dataset.z = d < cfg.soft ? 'soft' : d > cfg.loud ? 'loud' : 'right'; });
    $('bands').innerHTML =
      `<span><i style="background:var(--soft)"></i>Too soft: under ${cfg.soft} dB</span>` +
      `<span><i style="background:var(--right)"></i>Just right: ${cfg.soft}–${cfg.loud} dB</span>` +
      `<span><i style="background:var(--loud)"></i>Too loud: over ${cfg.loud} dB</span>`;
  }

  // Settings controls
  const fields = {
    soft: v => v + ' dB', loud: v => v + ' dB', hold: v => (+v).toFixed(1) + ' s',
    offset: v => (v >= 90 ? '+' : '') + (v - 90)
  };
  function syncInputs() {
    for (const k in fields) { $(k).value = cfg[k]; $(k + 'Out').textContent = fields[k](cfg[k]); }
    paintZones();
  }
  for (const k in fields) {
    $(k).addEventListener('input', e => {
      let v = +e.target.value;
      if (k === 'soft' && v > cfg.loud - 5) { cfg.loud = Math.min(95, v + 5); }
      if (k === 'loud' && v < cfg.soft + 5) { cfg.soft = Math.max(40, v - 5); }
      cfg[k] = v; syncInputs(); save();
    });
  }
  syncInputs();

  // Audio
  let ctx = null, analyser = null, stream = null, buf = null, running = false, demo = false;
  let smooth = MIN, history = [], peak = MIN, peakAt = 0;
  let zone = 'idle', cand = 'idle', candSince = 0, last = 0;
  const time = { soft: 0, right: 0, loud: 0 };
  let calib = null;

  function showMsg(t) { const m = $('msg'); m.textContent = t; m.hidden = !t; }

  async function start() {
    showMsg('');
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      showMsg('This browser can\'t use the microphone here. Open this file directly in Chrome, Edge, or Firefox (not inside another app\'s preview).');
      return;
    }
    try {
      stream = await navigator.mediaDevices.getUserMedia({ audio: { echoCancellation: false, noiseSuppression: false, autoGainControl: false } });
    } catch (e) {
      showMsg(e && e.name === 'NotAllowedError'
        ? 'Microphone access was blocked. Click the microphone or lock icon in the address bar, allow the microphone, then press Start listening again.'
        : 'No microphone was found. Plug one in or check your sound settings, then try again.');
      return;
    }
    ctx = new (window.AudioContext || window.webkitAudioContext)();
    const src = ctx.createMediaStreamSource(stream);
    analyser = ctx.createAnalyser(); analyser.fftSize = 2048;
    buf = new Float32Array(analyser.fftSize);
    src.connect(analyser);
    setDemo(false);
    running = true; $('startBtn').textContent = 'Stop'; setZone('wait');
    requestWake();
  }
  function stop() {
    running = false;
    if (stream) stream.getTracks().forEach(t => t.stop());
    if (ctx) ctx.close();
    stream = ctx = analyser = null;
    $('startBtn').textContent = 'Start listening';
    if (!demo) { setZone('idle'); $('db').textContent = '--'; smooth = MIN; render(MIN); }
  }

  function setDemo(on) {
    demo = on; $('demoBtn').setAttribute('aria-pressed', on);
    if (on) { if (running) stop(); setZone('wait'); }
    else if (!running) { setZone('idle'); $('db').textContent = '--'; smooth = MIN; render(MIN); }
  }

  let demoT = 0;
  function demoLevel(dt) {
    demoT += dt;
    // Wanders through soft → right → loud with syllable bursts and pauses
    const base = 63 + 14 * Math.sin(demoT * 0.35) + 4 * Math.sin(demoT * 1.3);
    const syll = Math.max(0, Math.sin(demoT * 9) + 0.25 * Math.sin(demoT * 23));
    const pause = Math.sin(demoT * 0.9) > 0.85 ? 0 : 1;
    return pause ? base - 10 + syll * 10 + Math.random() * 2 : 38 + Math.random() * 3;
  }

  function rawLevel() {
    analyser.getFloatTimeDomainData(buf);
    let sum = 0; for (let i = 0; i < buf.length; i++) sum += buf[i] * buf[i];
    const rms = Math.sqrt(sum / buf.length);
    return 20 * Math.log10(Math.max(rms, 1e-7)); // dBFS
  }

  function setZone(z) {
    zone = z;
    const key = z === 'wait' ? 'wait' : z;
    $('stage').dataset.zone = (z === 'wait') ? 'idle' : z;
    $('word').textContent = TEXT[key][0];
    $('hint').textContent = TEXT[key][1];
    $('glyph').innerHTML = ICONS[z === 'wait' ? 'idle' : z];
  }

  function render(level) {
    const p = pct(level);
    segs.forEach((s, i) => s.classList.toggle('on', segDb(i) <= level && level > MIN + 0.5));
    peakEl.style.left = pct(peak) + '%';
    peakEl.classList.toggle('show', (running || demo) && peak > MIN + 2);
  }

  function fmt(s) { s = Math.round(s); return Math.floor(s / 60) + ':' + String(s % 60).padStart(2, '0'); }
  function renderTally() {
    const total = time.soft + time.right + time.loud;
    for (const [k, n] of [['soft', 'Soft'], ['right', 'Right'], ['loud', 'Loud']]) {
      const v = total ? Math.round(time[k] / total * 100) : 0;
      $('p' + n).textContent = v + '%'; $('t' + n).style.width = v + '%';
    }
    $('talkTime').textContent = 'Speaking time ' + fmt(total);
  }

  function loop(now) {
    requestAnimationFrame(loop);
    const dt = last ? Math.min(0.1, (now - last) / 1000) : 0; last = now;
    if (!running && !demo) return;

    let raw;
    if (demo) raw = demoLevel(dt);
    else {
      const dbfs = rawLevel();
      if (calib) calib.samples.push(dbfs);
      raw = dbfs + cfg.offset;
    }
    raw = Math.min(MAX, Math.max(MIN, raw));
    // Fast attack, slower release
    const a = raw > smooth ? 0.45 : 0.12;
    smooth += (raw - smooth) * a;

    // Speaking level = loudest moment in the reaction window (ignores gaps between words)
    const t = now / 1000;
    history.push([t, smooth]);
    while (history.length && history[0][0] < t - cfg.hold) history.shift();
    const speech = history.reduce((m, h) => Math.max(m, h[1]), MIN);

    if (smooth >= peak || t - peakAt > 1.5) { peak = smooth; peakAt = t; }

    const floor = cfg.soft - 12;
    const next = speech < floor ? 'wait' : speech < cfg.soft ? 'soft' : speech > cfg.loud ? 'loud' : 'right';
    if (next !== cand) { cand = next; candSince = t; }
    const need = cand === 'wait' ? 1.2 : 0.35;
    if (cand !== zone && t - candSince >= need) setZone(cand);
    if (time[zone] !== undefined) time[zone] += dt;

    $('db').textContent = Math.round(speech < floor ? smooth : speech);
    render(smooth);
    renderTally();
  }
  requestAnimationFrame(loop);

  // Calibration: sample 3 s, use the 80th percentile of speaking frames as "middle of just right"
  $('calBtn').addEventListener('click', () => {
    if (!running) { $('calText').textContent = 'Press Start listening first, then Calibrate.'; return; }
    calib = { samples: [] };
    $('calBtn').disabled = true;
    let n = 3;
    $('calText').textContent = 'Speak now at a good presentation volume… 3';
    const iv = setInterval(() => {
      n--;
      if (n > 0) { $('calText').textContent = 'Speak now at a good presentation volume… ' + n; return; }
      clearInterval(iv);
      const s = calib.samples.sort((x, y) => x - y); calib = null;
      $('calBtn').disabled = false;
      if (s.length < 20) { $('calText').textContent = 'Not enough sound was picked up. Try again a little closer to the computer.'; return; }
      const p80 = s[Math.floor(s.length * 0.8)];
      const target = (cfg.soft + cfg.loud) / 2;
      cfg.offset = Math.round(Math.min(120, Math.max(60, target - p80)));
      syncInputs(); save();
      $('calText').textContent = 'Calibrated. That voice now reads about ' + Math.round(target) + ' dB, in the middle of "Just right."';
    }, 1000);
  });

  $('startBtn').addEventListener('click', () => running ? stop() : start());
  $('demoBtn').addEventListener('click', () => setDemo(!demo));
  $('resetBtn').addEventListener('click', () => { time.soft = time.right = time.loud = 0; renderTally(); });
  $('fsBtn').addEventListener('click', async () => {
    try { document.fullscreenElement ? await document.exitFullscreen() : await document.documentElement.requestFullscreen(); } catch (e) {}
  });

  let wake = null;
  async function requestWake() { try { wake = await navigator.wakeLock.request('screen'); } catch (e) {} }
  document.addEventListener('visibilitychange', () => { if (document.visibilityState === 'visible' && running) requestWake(); });

  setZone('idle'); render(MIN); renderTally();
})();
</script>
</body>
</html>
