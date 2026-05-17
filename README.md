<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Incu-Analyzer Dashboard</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mqtt/5.3.0/mqtt.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600;700&family=Syne:wght@400;700;800&display=swap');

  :root {
    --bg:       #0a0e14;
    --surface:  #111720;
    --card:     #161d28;
    --border:   #1e2a3a;
    --accent:   #00d4aa;
    --accent2:  #0096ff;
    --warn:     #ffb340;
    --danger:   #ff4f4f;
    --text:     #e2eaf5;
    --muted:    #5a7090;
    --mono:     'JetBrains Mono', monospace;
    --sans:     'Syne', sans-serif;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--sans);
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* ── Grid background ── */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(0,212,170,.04) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,212,170,.04) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  /* ── Header ── */
  header {
    position: relative;
    z-index: 10;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 18px 32px;
    border-bottom: 1px solid var(--border);
    background: rgba(10,14,20,.85);
    backdrop-filter: blur(12px);
  }

  .logo {
    display: flex; align-items: center; gap: 12px;
  }
  .logo-icon {
    width: 36px; height: 36px;
    background: var(--accent);
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 18px;
  }
  .logo-text { font-size: 20px; font-weight: 800; letter-spacing: -.5px; }
  .logo-sub  { font-family: var(--mono); font-size: 10px; color: var(--muted); margin-top: 1px; }

  .header-right { display: flex; align-items: center; gap: 20px; }

  #status-dot {
    width: 9px; height: 9px;
    border-radius: 50%;
    background: var(--muted);
    transition: background .3s;
  }
  #status-dot.connected   { background: var(--accent); box-shadow: 0 0 8px var(--accent); }
  #status-dot.connecting  { background: var(--warn);   animation: pulse 1s infinite; }
  #status-dot.error       { background: var(--danger); }

  #status-label {
    font-family: var(--mono);
    font-size: 11px;
    color: var(--muted);
  }

  #last-update {
    font-family: var(--mono);
    font-size: 11px;
    color: var(--muted);
  }

  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.4} }

  /* ── Main layout ── */
  main {
    position: relative;
    z-index: 1;
    max-width: 1200px;
    margin: 0 auto;
    padding: 28px 24px 48px;
  }

  /* ── Section title ── */
  .section-title {
    font-family: var(--mono);
    font-size: 10px;
    letter-spacing: 2px;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 14px;
    padding-left: 2px;
  }

  /* ── Temperature grid (T1–T4 spatial) ── */
  .temp-spatial {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1fr 1fr;
    gap: 10px;
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 10px;
    aspect-ratio: 1.6;
    margin-bottom: 24px;
  }

  .incubator-label {
    font-family: var(--mono);
    font-size: 10px;
    color: var(--muted);
    text-align: center;
    margin-bottom: 6px;
    letter-spacing: 1px;
  }

  /* ── Metric card ── */
  .card {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 18px 20px;
    display: flex;
    flex-direction: column;
    gap: 6px;
    transition: border-color .2s;
    position: relative;
    overflow: hidden;
  }
  .card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: var(--accent);
    opacity: 0;
    transition: opacity .3s;
  }
  .card.fresh::before { opacity: 1; }

  .card-label {
    font-family: var(--mono);
    font-size: 10px;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--muted);
  }
  .card-value {
    font-family: var(--mono);
    font-size: 26px;
    font-weight: 700;
    color: var(--text);
    line-height: 1;
    transition: color .3s;
  }
  .card-unit {
    font-family: var(--mono);
    font-size: 12px;
    color: var(--muted);
  }
  .card-desc {
    font-size: 11px;
    color: var(--muted);
    margin-top: 4px;
  }

  /* Temperature card color by value */
  .card-value.temp-cold { color: #64b5f6; }
  .card-value.temp-norm { color: var(--accent); }
  .card-value.temp-warm { color: var(--warn); }
  .card-value.temp-hot  { color: var(--danger); }

  /* Spatial position badge */
  .pos-badge {
    position: absolute;
    top: 10px; right: 12px;
    font-family: var(--mono);
    font-size: 9px;
    color: var(--muted);
    background: var(--border);
    padding: 2px 6px;
    border-radius: 4px;
  }

  /* ── Bottom row grid ── */
  .grid-6 {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
    margin-bottom: 24px;
  }
  .grid-3 {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
    margin-bottom: 24px;
  }

  /* ── Battery bar ── */
  .bat-bar-wrap {
    margin-top: 8px;
    height: 6px;
    background: var(--border);
    border-radius: 3px;
    overflow: hidden;
  }
  .bat-bar {
    height: 100%;
    border-radius: 3px;
    background: var(--accent);
    transition: width .6s ease, background .3s;
  }
  .bat-bar.low    { background: var(--danger); }
  .bat-bar.medium { background: var(--warn); }

  /* ── Airflow indicator ── */
  .flow-bars {
    display: flex;
    align-items: flex-end;
    gap: 3px;
    height: 28px;
    margin-top: 8px;
  }
  .flow-bar {
    flex: 1;
    border-radius: 2px;
    background: var(--border);
    transition: background .3s, height .4s;
  }

  /* ── Noise meter ── */
  .noise-meter {
    margin-top: 10px;
    height: 6px;
    background: var(--border);
    border-radius: 3px;
    overflow: hidden;
  }
  .noise-fill {
    height: 100%;
    border-radius: 3px;
    transition: width .4s ease, background .3s;
  }

  /* ── MQTT config panel ── */
  .config-panel {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 20px;
    margin-bottom: 24px;
  }
  .config-panel summary {
    cursor: pointer;
    font-family: var(--mono);
    font-size: 12px;
    color: var(--muted);
    user-select: none;
    list-style: none;
  }
  .config-panel summary::before { content: '▶  '; }
  .config-panel[open] summary::before { content: '▼  '; }
  .config-row {
    display: flex;
    gap: 12px;
    margin-top: 16px;
    flex-wrap: wrap;
  }
  .config-field {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
    min-width: 160px;
  }
  .config-field label {
    font-family: var(--mono);
    font-size: 10px;
    color: var(--muted);
    letter-spacing: 1px;
  }
  .config-field input {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 7px 10px;
    color: var(--text);
    font-family: var(--mono);
    font-size: 12px;
    outline: none;
    transition: border-color .2s;
  }
  .config-field input:focus { border-color: var(--accent); }
  .btn-connect {
    align-self: flex-end;
    padding: 8px 20px;
    background: var(--accent);
    color: #000;
    border: none;
    border-radius: 6px;
    font-family: var(--mono);
    font-size: 12px;
    font-weight: 700;
    cursor: pointer;
    transition: opacity .2s;
  }
  .btn-connect:hover { opacity: .85; }

  /* ── Log ── */
  #log {
    font-family: var(--mono);
    font-size: 11px;
    color: var(--muted);
    margin-top: 14px;
    height: 60px;
    overflow-y: auto;
    line-height: 1.7;
  }
  #log .ok   { color: var(--accent); }
  #log .warn { color: var(--warn); }
  #log .err  { color: var(--danger); }

  /* ── Responsive ── */
  @media (max-width: 700px) {
    .grid-6, .grid-3 { grid-template-columns: 1fr 1fr; }
    header { padding: 14px 16px; }
    main   { padding: 18px 14px 40px; }
  }
  @media (max-width: 450px) {
    .grid-6, .grid-3 { grid-template-columns: 1fr; }
    .temp-spatial { aspect-ratio: unset; }
  }
</style>
</head>
<body>

<header>
  <div class="logo">
    <div class="logo-icon">🌡</div>
    <div>
      <div class="logo-text">Incu-Analyzer</div>
      <div class="logo-sub">INCUBATOR MONITORING SYSTEM</div>
    </div>
  </div>
  <div class="header-right">
    <div id="status-dot" class="connecting"></div>
    <span id="status-label">Menghubungkan...</span>
    <span id="last-update">—</span>
  </div>
</header>

<main>

  <!-- MQTT Config -->
  <details class="config-panel" id="config-details">
    <summary>Konfigurasi MQTT</summary>
    <div class="config-row">
      <div class="config-field">
        <label>BROKER (WebSocket)</label>
        <input id="cfg-broker" value="broker.hivemq.com" />
      </div>
      <div class="config-field">
        <label>PORT WS</label>
        <input id="cfg-port" value="8000" style="max-width:100px"/>
      </div>
      <div class="config-field">
        <label>TOPIC</label>
        <input id="cfg-topic" value="incuanalyzer/sensors" />
      </div>
      <button class="btn-connect" onclick="mqttConnect()">Hubungkan</button>
    </div>
    <div id="log"></div>
  </details>

  <!-- T1–T4 Spatial Layout -->
  <p class="section-title">Distribusi Suhu Inkubator (T1–T4)</p>
  <div class="incubator-label">↑ DEPAN INKUBATOR ↑</div>
  <div class="temp-spatial">
    <!-- T1 Kiri Atas -->
    <div class="card" id="card-T1">
      <span class="pos-badge">Kiri Atas</span>
      <div class="card-label">T1</div>
      <div class="card-value" id="val-T1">—</div>
      <div class="card-unit">°C</div>
    </div>
    <!-- T2 Kanan Atas -->
    <div class="card" id="card-T2">
      <span class="pos-badge">Kanan Atas</span>
      <div class="card-label">T2</div>
      <div class="card-value" id="val-T2">—</div>
      <div class="card-unit">°C</div>
    </div>
    <!-- T3 Bawah Kiri -->
    <div class="card" id="card-T3">
      <span class="pos-badge">Bawah Kiri</span>
      <div class="card-label">T3</div>
      <div class="card-value" id="val-T3">—</div>
      <div class="card-unit">°C</div>
    </div>
    <!-- T4 Bawah Kanan -->
    <div class="card" id="card-T4">
      <span class="pos-badge">Bawah Kanan</span>
      <div class="card-label">T4</div>
      <div class="card-value" id="val-T4">—</div>
      <div class="card-unit">°C</div>
    </div>
  </div>

  <!-- T5, TM, RH -->
  <p class="section-title">Sensor Tambahan</p>
  <div class="grid-3">
    <div class="card" id="card-T5">
      <div class="card-label">T5 — SHT30</div>
      <div class="card-value" id="val-T5">—</div>
      <div class="card-unit">°C</div>
      <div class="card-desc">Suhu ruang referensi</div>
    </div>
    <div class="card" id="card-TM">
      <div class="card-label">TM — Matras</div>
      <div class="card-value" id="val-TM">—</div>
      <div class="card-unit">°C</div>
      <div class="card-desc">MAX6675 thermocouple</div>
    </div>
    <div class="card" id="card-RH">
      <div class="card-label">Kelembapan (RH)</div>
      <div class="card-value" id="val-RH">—</div>
      <div class="card-unit">%</div>
      <div class="card-desc">SHT30</div>
    </div>
  </div>

  <!-- Noise, Airflow, Battery -->
  <p class="section-title">Lingkungan & Sistem</p>
  <div class="grid-3">

    <div class="card" id="card-NOISE">
      <div class="card-label">Kebisingan</div>
      <div class="card-value" id="val-NOISE">—</div>
      <div class="card-unit">dB</div>
      <div class="noise-meter">
        <div class="noise-fill" id="noise-fill" style="width:0%;background:var(--accent)"></div>
      </div>
    </div>

    <div class="card" id="card-FLOW">
      <div class="card-label">Airflow</div>
      <div class="card-value" id="val-FLOW">—</div>
      <div class="card-unit">m/s</div>
      <div class="flow-bars" id="flow-bars">
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
        <div class="flow-bar" style="height:20%"></div>
      </div>
    </div>

    <div class="card" id="card-BAT">
      <div class="card-label">Baterai</div>
      <div class="card-value" id="val-BAT">—</div>
      <div class="card-unit">%  <span style="font-size:10px;color:var(--muted)">(8.4V max)</span></div>
      <div class="bat-bar-wrap">
        <div class="bat-bar" id="bat-bar" style="width:0%"></div>
      </div>
    </div>

  </div>

</main>

<script>
// ── MQTT State ──────────────────────────────────────────────
let client = null;

function log(msg, cls = '') {
  const el = document.getElementById('log');
  const ts = new Date().toLocaleTimeString('id-ID');
  el.innerHTML += `<span class="${cls}">[${ts}] ${msg}</span>\n`;
  el.scrollTop = el.scrollHeight;
}

function setStatus(state, label) {
  const dot = document.getElementById('status-dot');
  const lbl = document.getElementById('status-label');
  dot.className = state;
  lbl.textContent = label;
}

function mqttConnect() {
  if (client) { client.end(true); client = null; }

  const broker = document.getElementById('cfg-broker').value.trim();
  const port   = parseInt(document.getElementById('cfg-port').value.trim());
  const topic  = document.getElementById('cfg-topic').value.trim();
  const clientId = 'incu-web-' + Math.random().toString(16).slice(2, 8);

  setStatus('connecting', 'Menghubungkan...');
  log(`Menghubungkan ke ws://${broker}:${port}`, 'warn');

  const url = `ws://${broker}:${port}/mqtt`;

  client = mqtt.connect(url, {
    clientId,
    clean: true,
    reconnectPeriod: 5000,
    connectTimeout: 10000,
  });

  client.on('connect', () => {
    setStatus('connected', `Terhubung · ${broker}`);
    log(`Terhubung! Subscribe ke: ${topic}`, 'ok');
    client.subscribe(topic, { qos: 0 });
    document.getElementById('config-details').removeAttribute('open');
  });

  client.on('message', (t, payload) => {
    try {
      const data = JSON.parse(payload.toString());
      updateDashboard(data);
      const now = new Date().toLocaleTimeString('id-ID');
      document.getElementById('last-update').textContent = `Update: ${now}`;
    } catch (e) {
      log('JSON parse error: ' + e.message, 'err');
    }
  });

  client.on('error', (err) => {
    setStatus('error', 'Error');
    log('Error: ' + err.message, 'err');
  });

  client.on('offline', () => {
    setStatus('connecting', 'Reconnecting...');
    log('Koneksi terputus, mencoba ulang...', 'warn');
  });

  client.on('close', () => {
    setStatus('error', 'Terputus');
  });
}

// ── Dashboard Update ─────────────────────────────────────────
function tempClass(v) {
  if (isNaN(v)) return '';
  if (v < 30)   return 'temp-cold';
  if (v < 36)   return 'temp-norm';
  if (v < 39)   return 'temp-warm';
  return 'temp-hot';
}

function fmt(v, dec = 1) {
  return (v === null || v === undefined || isNaN(v)) ? '—' : v.toFixed(dec);
}

function flashCard(id) {
  const c = document.getElementById('card-' + id);
  if (!c) return;
  c.classList.remove('fresh');
  void c.offsetWidth;
  c.classList.add('fresh');
  setTimeout(() => c.classList.remove('fresh'), 1200);
}

function updateDashboard(d) {
  // T1–T4
  ['T1','T2','T3','T4'].forEach(k => {
    const el = document.getElementById('val-' + k);
    const v  = d[k];
    el.textContent = fmt(v);
    el.className   = 'card-value ' + tempClass(v);
    flashCard(k);
  });

  // T5, TM
  ['T5','TM'].forEach(k => {
    const el = document.getElementById('val-' + k);
    const v  = d[k] ?? d['RH'];
    el.textContent = (k === 'T5') ? fmt(d.T5) : fmt(d.TM);
    el.className   = 'card-value ' + tempClass(k === 'T5' ? d.T5 : d.TM);
    flashCard(k);
  });

  // RH
  document.getElementById('val-RH').textContent = fmt(d.RH, 1);
  flashCard('RH');

  // Noise
  const noise = d.NOISE ?? 0;
  document.getElementById('val-NOISE').textContent = fmt(noise, 1);
  const noiseW = Math.min(100, Math.max(0, (noise - 30) / 70 * 100));
  const nFill  = document.getElementById('noise-fill');
  nFill.style.width = noiseW + '%';
  nFill.style.background = noise > 80 ? 'var(--danger)' : noise > 60 ? 'var(--warn)' : 'var(--accent)';
  flashCard('NOISE');

  // Airflow
  const flow = d.FLOW ?? 0;
  document.getElementById('val-FLOW').textContent = fmt(flow, 3);
  const bars = document.querySelectorAll('#flow-bars .flow-bar');
  const flowPct = Math.min(1, flow / 5);
  bars.forEach((b, i) => {
    const threshold = (i + 1) / bars.length;
    const h = Math.max(15, threshold <= flowPct ? 90 : 15);
    b.style.height = h + '%';
    b.style.background = threshold <= flowPct
      ? (flow > 3 ? 'var(--warn)' : 'var(--accent2)')
      : 'var(--border)';
  });
  flashCard('FLOW');

  // Battery
  const bat = d.BAT ?? 0;
  document.getElementById('val-BAT').textContent = fmt(bat, 0);
  const bar = document.getElementById('bat-bar');
  bar.style.width = bat + '%';
  bar.className = 'bat-bar' + (bat < 20 ? ' low' : bat < 40 ? ' medium' : '');
  flashCard('BAT');
}

// ── Auto-connect on load ────────────────────────────────────
window.addEventListener('load', () => {
  mqttConnect();
});
</script>
</body>
</html>
