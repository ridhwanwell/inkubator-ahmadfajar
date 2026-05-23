<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>InCu Analyzer — Dashboard</title>

<!-- MQTT.js via CDN -->
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<!-- Google Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;800&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet" />

<style>
  /* ── Reset & Variables ── */
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg:        #09090f;
    --bg2:       #111120;
    --bg3:       #181830;
    --border:    #2a2a50;
    --accent:    #00e5ff;
    --accent2:   #7c3aed;
    --accent3:   #10b981;
    --warn:      #f59e0b;
    --danger:    #ef4444;
    --text:      #e2e8f0;
    --muted:     #64748b;
    --card-glow: 0 0 40px rgba(0,229,255,0.07);
    --font-head: 'Syne', sans-serif;
    --font-mono: 'JetBrains Mono', monospace;
  }

  html { scroll-behavior: smooth; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--font-head);
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* ── Background Grid ── */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(0,229,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,229,255,0.03) 1px, transparent 1px);
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
    padding: 1.2rem 2rem;
    border-bottom: 1px solid var(--border);
    background: rgba(9,9,15,0.85);
    backdrop-filter: blur(12px);
  }

  .logo {
    display: flex;
    align-items: center;
    gap: 0.75rem;
  }

  .logo-icon {
    width: 38px; height: 38px;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border-radius: 8px;
    display: flex; align-items: center; justify-content: center;
    font-size: 1.1rem;
  }

  .logo-text {
    font-size: 1.25rem;
    font-weight: 800;
    letter-spacing: -0.02em;
  }

  .logo-text span { color: var(--accent); }

  .header-right {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .status-badge {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-family: var(--font-mono);
    font-size: 0.72rem;
    padding: 0.35rem 0.8rem;
    border-radius: 100px;
    border: 1px solid var(--border);
    background: var(--bg2);
    transition: all 0.3s;
  }

  .status-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    background: var(--muted);
    transition: background 0.3s;
  }

  .status-badge.connected .status-dot {
    background: var(--accent3);
    box-shadow: 0 0 8px var(--accent3);
    animation: pulse-dot 2s infinite;
  }

  .status-badge.error .status-dot { background: var(--danger); }

  @keyframes pulse-dot {
    0%, 100% { opacity: 1; } 50% { opacity: 0.4; }
  }

  .last-update {
    font-family: var(--font-mono);
    font-size: 0.68rem;
    color: var(--muted);
  }

  /* ── Main Layout ── */
  main {
    position: relative;
    z-index: 1;
    max-width: 1400px;
    margin: 0 auto;
    padding: 2rem 1.5rem 4rem;
  }

  /* ── Section Title ── */
  .section-title {
    font-size: 0.65rem;
    font-family: var(--font-mono);
    color: var(--accent);
    letter-spacing: 0.15em;
    text-transform: uppercase;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .section-title::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--border);
  }

  /* ── Cards Grid ── */
  .cards-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 1rem;
    margin-bottom: 2.5rem;
  }

  .card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 1.25rem 1.1rem;
    position: relative;
    overflow: hidden;
    transition: transform 0.2s, box-shadow 0.2s, border-color 0.3s;
  }

  .card:hover {
    transform: translateY(-3px);
    box-shadow: var(--card-glow);
    border-color: rgba(0,229,255,0.25);
  }

  .card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: linear-gradient(90deg, var(--accent), var(--accent2));
    opacity: 0;
    transition: opacity 0.3s;
  }

  .card.updated::before { opacity: 1; }

  .card-label {
    font-family: var(--font-mono);
    font-size: 0.68rem;
    color: var(--muted);
    letter-spacing: 0.1em;
    text-transform: uppercase;
    margin-bottom: 0.5rem;
  }

  .card-value {
    font-size: 2rem;
    font-weight: 800;
    line-height: 1;
    font-family: var(--font-mono);
    color: var(--text);
    transition: color 0.3s;
  }

  .card-unit {
    font-size: 0.75rem;
    color: var(--muted);
    margin-top: 0.35rem;
    font-family: var(--font-mono);
  }

  .card-icon {
    position: absolute;
    bottom: 0.75rem; right: 0.9rem;
    font-size: 1.6rem;
    opacity: 0.12;
  }

  /* ── Color coding ── */
  .card[data-type="temp"]  .card-value { color: #f97316; }
  .card[data-type="humid"] .card-value { color: #38bdf8; }
  .card[data-type="noise"] .card-value { color: #a78bfa; }
  .card[data-type="flow"]  .card-value { color: var(--accent3); }
  .card[data-type="mean"]  .card-value { color: var(--accent); }

  /* ── Chart section ── */
  .charts-row {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 1.5rem;
    margin-bottom: 2.5rem;
  }

  @media (max-width: 900px) {
    .charts-row { grid-template-columns: 1fr; }
  }

  .chart-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 1.5rem;
  }

  .chart-title {
    font-size: 0.8rem;
    font-weight: 600;
    margin-bottom: 1rem;
    color: var(--text);
  }

  /* ── Log section ── */
  .log-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 1.5rem;
  }

  .log-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  .btn-clear {
    font-family: var(--font-mono);
    font-size: 0.65rem;
    padding: 0.3rem 0.7rem;
    border-radius: 6px;
    border: 1px solid var(--border);
    background: transparent;
    color: var(--muted);
    cursor: pointer;
    transition: all 0.2s;
  }

  .btn-clear:hover { border-color: var(--danger); color: var(--danger); }

  .log-body {
    height: 180px;
    overflow-y: auto;
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--muted);
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .log-body::-webkit-scrollbar { width: 4px; }
  .log-body::-webkit-scrollbar-track { background: transparent; }
  .log-body::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  .log-entry {
    display: flex;
    gap: 0.75rem;
    padding: 0.2rem 0;
    border-bottom: 1px solid rgba(42,42,80,0.5);
    animation: fadeIn 0.3s ease;
  }

  @keyframes fadeIn { from { opacity: 0; transform: translateY(-4px); } to { opacity: 1; } }

  .log-time { color: var(--accent2); flex-shrink: 0; }
  .log-ok   { color: var(--accent3); }
  .log-warn { color: var(--warn); }
  .log-err  { color: var(--danger); }

  /* ── Connection Config ── */
  .config-bar {
    display: flex;
    gap: 0.75rem;
    align-items: flex-end;
    flex-wrap: wrap;
    margin-bottom: 2.5rem;
  }

  .config-group {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
  }

  .config-group label {
    font-family: var(--font-mono);
    font-size: 0.62rem;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }

  .config-group input {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 0.5rem 0.75rem;
    color: var(--text);
    font-family: var(--font-mono);
    font-size: 0.8rem;
    outline: none;
    transition: border-color 0.2s;
    min-width: 180px;
  }

  .config-group input:focus { border-color: var(--accent); }

  .btn-connect {
    padding: 0.55rem 1.5rem;
    background: linear-gradient(135deg, var(--accent), #0099bb);
    color: #000;
    font-family: var(--font-head);
    font-weight: 700;
    font-size: 0.85rem;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: opacity 0.2s, transform 0.2s;
    white-space: nowrap;
  }

  .btn-connect:hover { opacity: 0.85; transform: scale(1.02); }
  .btn-connect:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }

  .btn-disconnect {
    padding: 0.55rem 1.2rem;
    background: transparent;
    color: var(--danger);
    font-family: var(--font-head);
    font-weight: 600;
    font-size: 0.85rem;
    border: 1px solid var(--danger);
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.2s;
    display: none;
  }

  .btn-disconnect:hover { background: rgba(239,68,68,0.1); }

  /* ── Alert banner ── */
  #alert-banner {
    display: none;
    background: rgba(239,68,68,0.12);
    border: 1px solid rgba(239,68,68,0.4);
    border-radius: 10px;
    padding: 0.75rem 1.25rem;
    margin-bottom: 1.5rem;
    font-family: var(--font-mono);
    font-size: 0.78rem;
    color: var(--danger);
  }

  /* ── Footer ── */
  footer {
    text-align: center;
    padding: 2rem;
    font-family: var(--font-mono);
    font-size: 0.65rem;
    color: var(--muted);
    border-top: 1px solid var(--border);
    position: relative;
    z-index: 1;
  }

  /* ── Sparkline flash ── */
  @keyframes flash {
    0% { background: rgba(0,229,255,0.15); }
    100% { background: transparent; }
  }
  .card.flash { animation: flash 0.6s ease-out; }

  /* ── Responsive ── */
  @media (max-width: 600px) {
    header { flex-direction: column; gap: 0.75rem; padding: 1rem; }
    .cards-grid { grid-template-columns: repeat(2, 1fr); }
    .config-bar { flex-direction: column; }
    .config-group input { min-width: 100%; }
  }
</style>
</head>
<body>

<!-- ══ HEADER ══ -->
<header>
  <div class="logo">
    <div class="logo-icon">🌡</div>
    <div class="logo-text">InCu<span>Analyzer</span></div>
  </div>
  <div class="header-right">
    <div id="status-badge" class="status-badge">
      <div class="status-dot"></div>
      <span id="status-text">Tidak Terhubung</span>
    </div>
    <div class="last-update" id="last-update">—</div>
  </div>
</header>

<!-- ══ MAIN ══ -->
<main>

  <!-- Config -->
  <div class="section-title">Konfigurasi Koneksi MQTT</div>
  <div class="config-bar">
    <div class="config-group">
      <label>Broker (WebSocket)</label>
      <input type="text" id="cfg-broker" value="wss://broker.hivemq.com:8884/mqtt" />
    </div>
    <div class="config-group">
      <label>Topic</label>
      <input type="text" id="cfg-topic" value="incuanalyzer/sensors" />
    </div>
    <div class="config-group">
      <label>Client ID</label>
      <input type="text" id="cfg-clientid" value="" placeholder="Auto-generate" />
    </div>
    <button class="btn-connect" id="btn-connect" onclick="doConnect()">Hubungkan</button>
    <button class="btn-disconnect" id="btn-disconnect" onclick="doDisconnect()">Putuskan</button>
  </div>

  <!-- Alert -->
  <div id="alert-banner"></div>

  <!-- Sensor Cards -->
  <div class="section-title">Data Sensor Real-time</div>
  <div class="cards-grid">
    <div class="card" data-type="temp" id="card-T1">
      <div class="card-label">Suhu T1</div>
      <div class="card-value" id="val-T1">—</div>
      <div class="card-unit">°C</div>
      <div class="card-icon">🌡</div>
    </div>
    <div class="card" data-type="temp" id="card-T2">
      <div class="card-label">Suhu T2</div>
      <div class="card-value" id="val-T2">—</div>
      <div class="card-unit">°C</div>
      <div class="card-icon">🌡</div>
    </div>
    <div class="card" data-type="temp" id="card-T3">
      <div class="card-label">Suhu T3</div>
      <div class="card-value" id="val-T3">—</div>
      <div class="card-unit">°C</div>
      <div class="card-icon">🌡</div>
    </div>
    <div class="card" data-type="temp" id="card-T4">
      <div class="card-label">Suhu T4</div>
      <div class="card-value" id="val-T4">—</div>
      <div class="card-unit">°C</div>
      <div class="card-icon">🌡</div>
    </div>
    <div class="card" data-type="temp" id="card-T5">
      <div class="card-label">Suhu T5</div>
      <div class="card-value" id="val-T5">—</div>
      <div class="card-unit">°C</div>
      <div class="card-icon">🌡</div>
    </div>
    <div class="card" data-type="mean" id="card-TM">
      <div class="card-label">Suhu Rata-rata</div>
      <div class="card-value" id="val-TM">—</div>
      <div class="card-unit">°C (TM)</div>
      <div class="card-icon">📊</div>
    </div>
    <div class="card" data-type="humid" id="card-RH">
      <div class="card-label">Kelembaban</div>
      <div class="card-value" id="val-RH">—</div>
      <div class="card-unit">% RH</div>
      <div class="card-icon">💧</div>
    </div>
    <div class="card" data-type="noise" id="card-NOISE">
      <div class="card-label">Kebisingan</div>
      <div class="card-value" id="val-NOISE">—</div>
      <div class="card-unit">dB</div>
      <div class="card-icon">🔊</div>
    </div>
    <div class="card" data-type="flow" id="card-FLOW">
      <div class="card-label">Aliran Udara</div>
      <div class="card-value" id="val-FLOW">—</div>
      <div class="card-unit">m/s</div>
      <div class="card-icon">💨</div>
    </div>
  </div>

  <!-- Charts -->
  <div class="section-title">Grafik Historis (50 data terakhir)</div>
  <div class="charts-row">
    <div class="chart-card">
      <div class="chart-title">Suhu T1 – T5 & TM (°C)</div>
      <canvas id="chart-temp" height="120"></canvas>
    </div>
    <div class="chart-card">
      <div class="chart-title">Kelembaban & Kebisingan</div>
      <canvas id="chart-env" height="120"></canvas>
    </div>
  </div>

  <!-- Log -->
  <div class="section-title">Log MQTT</div>
  <div class="log-card">
    <div class="log-header">
      <div class="chart-title" style="margin:0">Pesan Masuk</div>
      <button class="btn-clear" onclick="clearLog()">Bersihkan</button>
    </div>
    <div class="log-body" id="log-body"></div>
  </div>

</main>

<footer>
  InCu Analyzer Dashboard &nbsp;·&nbsp;
  MQTT Broker: <span id="footer-broker">broker.hivemq.com</span> &nbsp;·&nbsp;
  Topic: <span id="footer-topic">incuanalyzer/sensors</span>
</footer>

<script>
// ══════════════════════════════════════════════
//  State
// ══════════════════════════════════════════════
let client = null;
const MAX_POINTS = 50;

const history = {
  labels: [],
  T1: [], T2: [], T3: [], T4: [], T5: [], TM: [],
  RH: [], NOISE: []
};

// ══════════════════════════════════════════════
//  Charts
// ══════════════════════════════════════════════
const ctxTemp = document.getElementById('chart-temp').getContext('2d');
const ctxEnv  = document.getElementById('chart-env').getContext('2d');

const commonOptions = {
  responsive: true,
  animation: { duration: 400 },
  plugins: { legend: { labels: { color: '#94a3b8', font: { family: "'JetBrains Mono'" } } } },
  scales: {
    x: { ticks: { color: '#475569', maxTicksLimit: 8, font: { family: "'JetBrains Mono'", size: 10 } }, grid: { color: 'rgba(42,42,80,0.6)' } },
    y: { ticks: { color: '#94a3b8', font: { family: "'JetBrains Mono'", size: 10 } }, grid: { color: 'rgba(42,42,80,0.6)' } }
  }
};

const tempChart = new Chart(ctxTemp, {
  type: 'line',
  data: {
    labels: history.labels,
    datasets: [
      { label: 'T1', data: history.T1, borderColor: '#f97316', borderWidth: 1.5, pointRadius: 2, tension: 0.4, fill: false },
      { label: 'T2', data: history.T2, borderColor: '#fb923c', borderWidth: 1.5, pointRadius: 2, tension: 0.4, fill: false },
      { label: 'T3', data: history.T3, borderColor: '#fbbf24', borderWidth: 1.5, pointRadius: 2, tension: 0.4, fill: false },
      { label: 'T4', data: history.T4, borderColor: '#f43f5e', borderWidth: 1.5, pointRadius: 2, tension: 0.4, fill: false },
      { label: 'T5', data: history.T5, borderColor: '#e879f9', borderWidth: 1.5, pointRadius: 2, tension: 0.4, fill: false },
      { label: 'TM', data: history.TM, borderColor: '#00e5ff', borderWidth: 2.5, pointRadius: 3, tension: 0.4, fill: false },
    ]
  },
  options: { ...commonOptions }
});

const envChart = new Chart(ctxEnv, {
  type: 'line',
  data: {
    labels: history.labels,
    datasets: [
      { label: 'RH (%)', data: history.RH,    borderColor: '#38bdf8', borderWidth: 2, pointRadius: 2, tension: 0.4, fill: false, yAxisID: 'y' },
      { label: 'dB',     data: history.NOISE, borderColor: '#a78bfa', borderWidth: 2, pointRadius: 2, tension: 0.4, fill: false, yAxisID: 'y2' },
    ]
  },
  options: {
    ...commonOptions,
    scales: {
      ...commonOptions.scales,
      y:  { ...commonOptions.scales.y, position: 'left' },
      y2: { ...commonOptions.scales.y, position: 'right', grid: { drawOnChartArea: false } }
    }
  }
});

// ══════════════════════════════════════════════
//  Update UI
// ══════════════════════════════════════════════
function fmt(v, d=2) {
  if (v === null || v === undefined || isNaN(v)) return '—';
  return parseFloat(v).toFixed(d);
}

function flashCard(id) {
  const el = document.getElementById('card-' + id);
  if (!el) return;
  el.classList.remove('flash', 'updated');
  void el.offsetWidth;
  el.classList.add('flash', 'updated');
}

function updateCards(d) {
  const keys = ['T1','T2','T3','T4','T5','TM','RH','NOISE','FLOW'];
  keys.forEach(k => {
    const el = document.getElementById('val-' + k);
    if (!el) return;
    const decimals = k === 'FLOW' ? 3 : k === 'NOISE' ? 1 : 2;
    const newVal = fmt(d[k], decimals);
    if (el.textContent !== newVal) {
      el.textContent = newVal;
      flashCard(k);
    }
  });
}

function pushHistory(d) {
  const t = new Date().toLocaleTimeString('id-ID', { hour12: false });
  history.labels.push(t);
  ['T1','T2','T3','T4','T5','TM','RH','NOISE'].forEach(k => {
    history[k].push(d[k] ?? null);
  });

  if (history.labels.length > MAX_POINTS) {
    history.labels.shift();
    ['T1','T2','T3','T4','T5','TM','RH','NOISE'].forEach(k => history[k].shift());
  }

  tempChart.update();
  envChart.update();
}

// ══════════════════════════════════════════════
//  Log
// ══════════════════════════════════════════════
function addLog(msg, type = 'ok') {
  const box = document.getElementById('log-body');
  const t = new Date().toLocaleTimeString('id-ID', { hour12: false });
  const div = document.createElement('div');
  div.className = 'log-entry';
  div.innerHTML = `<span class="log-time">${t}</span><span class="log-${type}">${escHtml(msg)}</span>`;
  box.prepend(div);
  if (box.children.length > 100) box.lastElementChild.remove();
}

function clearLog() { document.getElementById('log-body').innerHTML = ''; }
function escHtml(s) { return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }

// ══════════════════════════════════════════════
//  Status
// ══════════════════════════════════════════════
function setStatus(state, text) {
  const badge = document.getElementById('status-badge');
  const span  = document.getElementById('status-text');
  badge.className = 'status-badge ' + state;
  span.textContent = text;
}

function showAlert(msg) {
  const el = document.getElementById('alert-banner');
  el.textContent = msg;
  el.style.display = msg ? 'block' : 'none';
}

// ══════════════════════════════════════════════
//  MQTT
// ══════════════════════════════════════════════
function doConnect() {
  const brokerUrl = document.getElementById('cfg-broker').value.trim();
  const topic     = document.getElementById('cfg-topic').value.trim();
  let   clientId  = document.getElementById('cfg-clientid').value.trim();

  if (!clientId) clientId = 'WebDash-' + Math.random().toString(16).slice(2,8);
  if (!brokerUrl || !topic) { showAlert('Isi broker dan topic terlebih dahulu.'); return; }

  showAlert('');
  setStatus('connecting', 'Menghubungkan...');
  addLog(`Menghubungkan ke ${brokerUrl}`, 'warn');

  document.getElementById('btn-connect').disabled = true;

  client = mqtt.connect(brokerUrl, {
    clientId,
    clean: true,
    reconnectPeriod: 5000,
    connectTimeout: 10000,
  });

  client.on('connect', () => {
    setStatus('connected', 'Terhubung');
    showAlert('');
    addLog(`Terhubung! Client ID: ${clientId}`, 'ok');
    document.getElementById('btn-disconnect').style.display = 'inline-block';
    document.getElementById('footer-broker').textContent = brokerUrl;
    document.getElementById('footer-topic').textContent  = topic;

    client.subscribe(topic, { qos: 0 }, (err) => {
      if (err) { addLog('Subscribe gagal: ' + err.message, 'err'); }
      else      { addLog(`Subscribe ke topic: ${topic}`, 'ok'); }
    });
  });

  client.on('message', (t, payload) => {
    const raw = payload.toString();
    addLog(`[${t}] ${raw}`, 'ok');

    try {
      const d = JSON.parse(raw);
      updateCards(d);
      pushHistory(d);
      document.getElementById('last-update').textContent =
        'Update: ' + new Date().toLocaleTimeString('id-ID', { hour12: false });
    } catch (e) {
      addLog('JSON parse error: ' + e.message, 'err');
    }
  });

  client.on('error', (err) => {
    setStatus('error', 'Error');
    addLog('Error: ' + err.message, 'err');
    showAlert('Koneksi error: ' + err.message);
    document.getElementById('btn-connect').disabled = false;
  });

  client.on('close', () => {
    setStatus('', 'Terputus');
    addLog('Koneksi terputus.', 'warn');
    document.getElementById('btn-connect').disabled = false;
    document.getElementById('btn-disconnect').style.display = 'none';
  });

  client.on('reconnect', () => {
    setStatus('connecting', 'Mencoba Ulang...');
    addLog('Mencoba ulang koneksi...', 'warn');
  });
}

function doDisconnect() {
  if (client) {
    client.end(true);
    client = null;
  }
  setStatus('', 'Tidak Terhubung');
  addLog('Koneksi diputus oleh pengguna.', 'warn');
  document.getElementById('btn-disconnect').style.display = 'none';
  document.getElementById('btn-connect').disabled = false;
}

// ── Auto-set clientId on load ──
document.getElementById('cfg-clientid').placeholder =
  'WebDash-' + Math.random().toString(16).slice(2,8);
</script>
</body>
</html>
