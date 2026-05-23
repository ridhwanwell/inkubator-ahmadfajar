<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>InCu Analyzer — Dashboard</title>
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;800&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet" />
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
  --bg:#09090f;--bg2:#111120;--bg3:#181830;--border:#2a2a50;
  --accent:#00e5ff;--accent2:#7c3aed;--accent3:#10b981;
  --warn:#f59e0b;--danger:#ef4444;--text:#e2e8f0;--muted:#64748b;
  --font-head:'Syne',sans-serif;--font-mono:'JetBrains Mono',monospace;
}
html{scroll-behavior:smooth}
body{background:var(--bg);color:var(--text);font-family:var(--font-head);min-height:100vh;overflow-x:hidden}
body::before{content:'';position:fixed;inset:0;background-image:linear-gradient(rgba(0,229,255,0.03) 1px,transparent 1px),linear-gradient(90deg,rgba(0,229,255,0.03) 1px,transparent 1px);background-size:40px 40px;pointer-events:none;z-index:0}

/* Header */
header{position:relative;z-index:10;display:flex;align-items:center;justify-content:space-between;padding:1.1rem 2rem;border-bottom:1px solid var(--border);background:rgba(9,9,15,0.9);backdrop-filter:blur(12px)}
.logo{display:flex;align-items:center;gap:.75rem}
.logo-icon{width:36px;height:36px;background:linear-gradient(135deg,var(--accent),var(--accent2));border-radius:8px;display:flex;align-items:center;justify-content:center;font-size:1rem}
.logo-text{font-size:1.2rem;font-weight:800;letter-spacing:-.02em}
.logo-text span{color:var(--accent)}
.header-right{display:flex;align-items:center;gap:1rem}
.status-badge{display:flex;align-items:center;gap:.5rem;font-family:var(--font-mono);font-size:.7rem;padding:.3rem .8rem;border-radius:100px;border:1px solid var(--border);background:var(--bg2);transition:all .3s}
.status-dot{width:8px;height:8px;border-radius:50%;background:var(--muted);transition:background .3s}
.status-badge.connected .status-dot{background:var(--accent3);box-shadow:0 0 8px var(--accent3);animation:pulse-dot 2s infinite}
.status-badge.error .status-dot{background:var(--danger)}
@keyframes pulse-dot{0%,100%{opacity:1}50%{opacity:.4}}
.last-update{font-family:var(--font-mono);font-size:.65rem;color:var(--muted)}

/* Main */
main{position:relative;z-index:1;max-width:1400px;margin:0 auto;padding:1.5rem 1.5rem 4rem}
.section-title{font-size:.62rem;font-family:var(--font-mono);color:var(--accent);letter-spacing:.15em;text-transform:uppercase;margin-bottom:.9rem;display:flex;align-items:center;gap:.5rem}
.section-title::after{content:'';flex:1;height:1px;background:var(--border)}

/* Cards */
.cards-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(155px,1fr));gap:.9rem;margin-bottom:2rem}
.card{background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:1.1rem 1rem;position:relative;overflow:hidden;transition:transform .2s,box-shadow .2s,border-color .3s}
.card:hover{transform:translateY(-2px);box-shadow:0 0 30px rgba(0,229,255,.08);border-color:rgba(0,229,255,.25)}
.card::before{content:'';position:absolute;top:0;left:0;right:0;height:2px;background:linear-gradient(90deg,var(--accent),var(--accent2));opacity:0;transition:opacity .3s}
.card.updated::before{opacity:1}
.card-label{font-family:var(--font-mono);font-size:.62rem;color:var(--muted);letter-spacing:.08em;text-transform:uppercase;margin-bottom:.4rem}
.card-value{font-size:1.9rem;font-weight:800;line-height:1;font-family:var(--font-mono);color:var(--text);transition:color .3s}
.card-unit{font-size:.7rem;color:var(--muted);margin-top:.3rem;font-family:var(--font-mono)}
.card-icon{position:absolute;bottom:.7rem;right:.85rem;font-size:1.5rem;opacity:.1}
.card[data-type="temp"]   .card-value{color:#f97316}
.card[data-type="humid"]  .card-value{color:#38bdf8}
.card[data-type="noise"]  .card-value{color:#a78bfa}
.card[data-type="flow"]   .card-value{color:var(--accent3)}
.card[data-type="matras"] .card-value{color:var(--accent)}
@keyframes flash{0%{background:rgba(0,229,255,.18)}100%{background:transparent}}
.card.flash{animation:flash .6s ease-out}

/* Charts */
.charts-row{display:grid;grid-template-columns:1fr 1fr;gap:1.25rem;margin-bottom:1.25rem}
.charts-row-3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:1.25rem;margin-bottom:2rem}
@media(max-width:900px){.charts-row,.charts-row-3{grid-template-columns:1fr}}
.chart-card{background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:1.25rem}
.chart-title{font-size:.78rem;font-weight:600;margin-bottom:.9rem;color:var(--text)}

/* Storage panel */
.storage-panel{background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:1.5rem;margin-bottom:2rem}
.storage-top{display:flex;align-items:flex-start;justify-content:space-between;flex-wrap:wrap;gap:1rem;margin-bottom:1.25rem}
.storage-settings{display:flex;gap:1.5rem;flex-wrap:wrap;align-items:flex-end}
.inp-group{display:flex;flex-direction:column;gap:.3rem}
.inp-group label{font-family:var(--font-mono);font-size:.6rem;color:var(--muted);text-transform:uppercase;letter-spacing:.08em}
.inp-group input{background:var(--bg3);border:1px solid var(--border);border-radius:7px;padding:.4rem .7rem;color:var(--text);font-family:var(--font-mono);font-size:.8rem;outline:none;width:110px;transition:border-color .2s}
.inp-group input:focus{border-color:var(--accent)}
.storage-btns{display:flex;gap:.75rem;flex-wrap:wrap}
.btn{padding:.5rem 1.1rem;font-family:var(--font-head);font-weight:700;font-size:.8rem;border-radius:8px;cursor:pointer;transition:all .2s;border:none;white-space:nowrap}
.btn-primary{background:linear-gradient(135deg,var(--accent),#0099bb);color:#000}
.btn-primary:hover{opacity:.85;transform:scale(1.02)}
.btn-primary:disabled{opacity:.4;cursor:not-allowed;transform:none}
.btn-success{background:linear-gradient(135deg,var(--accent3),#0d9468);color:#fff}
.btn-success:hover{opacity:.85}
.btn-warn{background:transparent;color:var(--warn);border:1px solid var(--warn)}
.btn-warn:hover{background:rgba(245,158,11,.1)}
.btn-danger{background:transparent;color:var(--danger);border:1px solid var(--danger)}
.btn-danger:hover{background:rgba(239,68,68,.1)}
.storage-stats{display:flex;gap:2rem;flex-wrap:wrap}
.stat-item{display:flex;flex-direction:column;gap:.2rem}
.stat-label{font-family:var(--font-mono);font-size:.58rem;color:var(--muted);text-transform:uppercase;letter-spacing:.08em}
.stat-value{font-family:var(--font-mono);font-size:1.1rem;font-weight:600;color:var(--text)}
.stat-value.active{color:var(--accent3)}
.progress-bar-wrap{margin-top:1rem}
.progress-label{display:flex;justify-content:space-between;font-family:var(--font-mono);font-size:.62rem;color:var(--muted);margin-bottom:.35rem}
.progress-bar{height:5px;background:var(--bg3);border-radius:3px;overflow:hidden}
.progress-fill{height:100%;background:linear-gradient(90deg,var(--accent3),var(--accent));border-radius:3px;transition:width .5s ease;width:0%}

/* Footer */
footer{text-align:center;padding:1.5rem;font-family:var(--font-mono);font-size:.62rem;color:var(--muted);border-top:1px solid var(--border);position:relative;z-index:1}

@media(max-width:600px){
  header{flex-direction:column;gap:.75rem;padding:1rem}
  .cards-grid{grid-template-columns:repeat(2,1fr)}
  .storage-settings{flex-direction:column}
}
</style>
</head>
<body>

<header>
  <div class="logo">
    <div class="logo-icon">🌡</div>
    <div class="logo-text">InCu<span>Analyzer</span></div>
  </div>
  <div class="header-right">
    <div id="status-badge" class="status-badge">
      <div class="status-dot"></div>
      <span id="status-text">Menghubungkan...</span>
    </div>
    <div class="last-update" id="last-update">—</div>
  </div>
</header>

<main>

  <div class="section-title">Data Sensor Real-time</div>
  <div class="cards-grid">
    <div class="card" data-type="temp"   id="card-T1">   <div class="card-label">Suhu T1</div>         <div class="card-value" id="val-T1">—</div>   <div class="card-unit">°C</div>    <div class="card-icon">🌡</div></div>
    <div class="card" data-type="temp"   id="card-T2">   <div class="card-label">Suhu T2</div>         <div class="card-value" id="val-T2">—</div>   <div class="card-unit">°C</div>    <div class="card-icon">🌡</div></div>
    <div class="card" data-type="temp"   id="card-T3">   <div class="card-label">Suhu T3</div>         <div class="card-value" id="val-T3">—</div>   <div class="card-unit">°C</div>    <div class="card-icon">🌡</div></div>
    <div class="card" data-type="temp"   id="card-T4">   <div class="card-label">Suhu T4</div>         <div class="card-value" id="val-T4">—</div>   <div class="card-unit">°C</div>    <div class="card-icon">🌡</div></div>
    <div class="card" data-type="temp"   id="card-T5">   <div class="card-label">Suhu T5</div>         <div class="card-value" id="val-T5">—</div>   <div class="card-unit">°C</div>    <div class="card-icon">🌡</div></div>
    <div class="card" data-type="matras" id="card-TM">   <div class="card-label">Temp. Matras</div>    <div class="card-value" id="val-TM">—</div>   <div class="card-unit">°C (TM)</div><div class="card-icon">🛏</div></div>
    <div class="card" data-type="humid"  id="card-RH">   <div class="card-label">Kelembaban</div>      <div class="card-value" id="val-RH">—</div>   <div class="card-unit">% RH</div>  <div class="card-icon">💧</div></div>
    <div class="card" data-type="noise"  id="card-NOISE"><div class="card-label">Kebisingan</div>      <div class="card-value" id="val-NOISE">—</div><div class="card-unit">dB</div>    <div class="card-icon">🔊</div></div>
    <div class="card" data-type="flow"   id="card-FLOW"> <div class="card-label">Aliran Udara</div>    <div class="card-value" id="val-FLOW">—</div> <div class="card-unit">m/s</div>   <div class="card-icon">💨</div></div>
  </div>

  <div class="section-title">Grafik Suhu (°C)</div>
  <div class="charts-row">
    <div class="chart-card">
      <div class="chart-title">T1 — T5 (Suhu Ruang)</div>
      <canvas id="chart-temp" height="110"></canvas>
    </div>
    <div class="chart-card">
      <div class="chart-title">TM — Temperature Matras (°C)</div>
      <canvas id="chart-tm" height="110"></canvas>
    </div>
  </div>

  <div class="section-title">Grafik Lingkungan</div>
  <div class="charts-row-3">
    <div class="chart-card">
      <div class="chart-title">Kelembaban (% RH)</div>
      <canvas id="chart-rh" height="110"></canvas>
    </div>
    <div class="chart-card">
      <div class="chart-title">Kebisingan (dB)</div>
      <canvas id="chart-noise" height="110"></canvas>
    </div>
    <div class="chart-card">
      <div class="chart-title">Aliran Udara (m/s)</div>
      <canvas id="chart-flow" height="110"></canvas>
    </div>
  </div>

  <div class="section-title">Penyimpanan Data</div>
  <div class="storage-panel">
    <div class="storage-top">
      <div class="storage-settings">
        <div class="inp-group">
          <label>Interval (detik)</label>
          <input type="number" id="cfg-interval" value="1" min="1" max="3600" />
        </div>
        <div class="inp-group">
          <label>Durasi (menit)</label>
          <input type="number" id="cfg-duration" value="5" min="1" max="1440" />
        </div>
        <div class="storage-btns">
          <button class="btn btn-primary" id="btn-save" onclick="startSaving()">▶ Mulai Simpan</button>
          <button class="btn btn-warn"    id="btn-stop" onclick="stopSaving()" disabled>⏹ Stop</button>
          <button class="btn btn-success" onclick="exportExcel()">⬇ Export Excel</button>
          <button class="btn btn-danger"  onclick="resetData()">🗑 Reset Data</button>
        </div>
      </div>
    </div>

    <div class="storage-stats">
      <div class="stat-item"><div class="stat-label">Data Tersimpan</div><div class="stat-value" id="stat-count">0</div></div>
      <div class="stat-item"><div class="stat-label">Status</div><div class="stat-value" id="stat-status">Idle</div></div>
      <div class="stat-item"><div class="stat-label">Waktu Mulai</div><div class="stat-value" id="stat-start">—</div></div>
      <div class="stat-item"><div class="stat-label">Sisa Waktu</div><div class="stat-value active" id="stat-remain">—</div></div>
    </div>

    <div class="progress-bar-wrap">
      <div class="progress-label">
        <span>Progress Penyimpanan</span>
        <span id="progress-pct">0%</span>
      </div>
      <div class="progress-bar"><div class="progress-fill" id="progress-fill"></div></div>
    </div>
  </div>

</main>

<footer>
  InCu Analyzer Dashboard &nbsp;·&nbsp; broker.hivemq.com &nbsp;·&nbsp; incuanalyzer/sensors
</footer>

<script>
// ══ CONFIG (sesuai ESP32) ══════════════════════
const BROKER    = 'wss://broker.hivemq.com:8884/mqtt';
const TOPIC     = 'incuanalyzer/sensors';
const CLIENT_ID = 'WebDash-' + Math.random().toString(16).slice(2,8);
const MAX_CHART = 60;

// ══ State ══════════════════════════════════════
let latestData = null;
let savedRows  = [];
let saveActive = false;
let saveTimer  = null, countdownTimer = null;
let saveEnd = 0, saveStart = null, saveDuration = 0;

const hist = { labels:[], T1:[], T2:[], T3:[], T4:[], T5:[], TM:[], RH:[], NOISE:[], FLOW:[] };

// ══ Charts ════════════════════════════════════
const scaleOpts = {
  x:{ ticks:{ color:'#475569', maxTicksLimit:8, font:{ family:"'JetBrains Mono'", size:9 } }, grid:{ color:'rgba(42,42,80,.6)' } },
  y:{ ticks:{ color:'#94a3b8', font:{ family:"'JetBrains Mono'", size:9 } }, grid:{ color:'rgba(42,42,80,.6)' } }
};

function makeMulti(id, datasets) {
  return new Chart(document.getElementById(id).getContext('2d'), {
    type:'line', data:{ labels:hist.labels, datasets },
    options:{ responsive:true, animation:{ duration:300 }, plugins:{ legend:{ labels:{ color:'#94a3b8', font:{ family:"'JetBrains Mono'", size:9 }, boxWidth:10, padding:8 } } }, scales: scaleOpts }
  });
}
function makeSingle(id, key, color) {
  return new Chart(document.getElementById(id).getContext('2d'), {
    type:'line', data:{ labels:hist.labels, datasets:[{ label:key, data:hist[key], borderColor:color, borderWidth:2, pointRadius:1.5, tension:0.4, fill:false }] },
    options:{ responsive:true, animation:{ duration:300 }, plugins:{ legend:{ display:false } }, scales: scaleOpts }
  });
}

const tempChart  = makeMulti('chart-temp', [
  { label:'T1', data:hist.T1, borderColor:'#f97316', borderWidth:1.5, pointRadius:1, tension:0.4, fill:false },
  { label:'T2', data:hist.T2, borderColor:'#fb923c', borderWidth:1.5, pointRadius:1, tension:0.4, fill:false },
  { label:'T3', data:hist.T3, borderColor:'#fbbf24', borderWidth:1.5, pointRadius:1, tension:0.4, fill:false },
  { label:'T4', data:hist.T4, borderColor:'#f43f5e', borderWidth:1.5, pointRadius:1, tension:0.4, fill:false },
  { label:'T5', data:hist.T5, borderColor:'#e879f9', borderWidth:1.5, pointRadius:1, tension:0.4, fill:false },
]);
const tmChart    = makeSingle('chart-tm',    'TM',    '#00e5ff');
const rhChart    = makeSingle('chart-rh',    'RH',    '#38bdf8');
const noiseChart = makeSingle('chart-noise', 'NOISE', '#a78bfa');
const flowChart  = makeSingle('chart-flow',  'FLOW',  '#10b981');

// ══ UI helpers ════════════════════════════════
function fmt(v, d){ return (v===null||v===undefined||isNaN(v)) ? '—' : parseFloat(v).toFixed(d||2); }

function flashCard(id){
  const el = document.getElementById('card-'+id);
  if(!el) return;
  el.classList.remove('flash','updated');
  void el.offsetWidth;
  el.classList.add('flash','updated');
}

function updateCards(d){
  [['T1',2],['T2',2],['T3',2],['T4',2],['T5',2],['TM',2],['RH',2],['NOISE',1],['FLOW',3]].forEach(([k,dec])=>{
    const el = document.getElementById('val-'+k);
    if(!el) return;
    const nv = fmt(d[k], dec);
    if(el.textContent !== nv){ el.textContent = nv; flashCard(k); }
  });
}

function pushHist(d){
  const t = new Date().toLocaleTimeString('id-ID',{hour12:false});
  hist.labels.push(t);
  ['T1','T2','T3','T4','T5','TM','RH','NOISE','FLOW'].forEach(k => hist[k].push(d[k] ?? null));
  if(hist.labels.length > MAX_CHART){
    hist.labels.shift();
    ['T1','T2','T3','T4','T5','TM','RH','NOISE','FLOW'].forEach(k => hist[k].shift());
  }
  [tempChart, tmChart, rhChart, noiseChart, flowChart].forEach(c => c.update());
}

function setStatus(state, text){
  document.getElementById('status-badge').className = 'status-badge '+state;
  document.getElementById('status-text').textContent = text;
}

// ══ MQTT ═════════════════════════════════════
setStatus('connecting','Menghubungkan...');

const mqttClient = mqtt.connect(BROKER, {
  clientId: CLIENT_ID,
  clean: true,
  reconnectPeriod: 5000,
  connectTimeout: 12000,
});

mqttClient.on('connect', () => {
  setStatus('connected','Terhubung');
  mqttClient.subscribe(TOPIC, { qos:0 });
});
mqttClient.on('message', (topic, payload) => {
  try {
    const d = JSON.parse(payload.toString());
    latestData = d;
    updateCards(d);
    pushHist(d);
    document.getElementById('last-update').textContent =
      'Update: ' + new Date().toLocaleTimeString('id-ID',{hour12:false});
  } catch(e) {}
});
mqttClient.on('error',     () => setStatus('error',     'Error'));
mqttClient.on('close',     () => setStatus('',          'Terputus'));
mqttClient.on('reconnect', () => setStatus('connecting','Mencoba ulang...'));

// ══ Saving ═══════════════════════════════════
function startSaving(){
  if(saveActive) return;
  const interval = Math.max(1, parseInt(document.getElementById('cfg-interval').value)||1) * 1000;
  saveDuration   = Math.max(1, parseInt(document.getElementById('cfg-duration').value)||5) * 60 * 1000;
  saveActive = true;
  saveStart  = new Date();
  saveEnd    = Date.now() + saveDuration;

  document.getElementById('stat-status').textContent = 'Merekam';
  document.getElementById('stat-status').style.color = 'var(--accent3)';
  document.getElementById('stat-start').textContent  = saveStart.toLocaleTimeString('id-ID',{hour12:false});
  document.getElementById('btn-save').disabled = true;
  document.getElementById('btn-stop').disabled = false;

  saveTimer = setInterval(() => {
    if(!latestData) return;
    const now = new Date();
    savedRows.push({
      date:  now.toLocaleDateString('id-ID'),
      time:  now.toLocaleTimeString('id-ID',{hour12:false}),
      T1:    latestData.T1    ?? null,
      T2:    latestData.T2    ?? null,
      T3:    latestData.T3    ?? null,
      T4:    latestData.T4    ?? null,
      T5:    latestData.T5    ?? null,
      TM:    latestData.TM   ?? null,
      RH:    latestData.RH   ?? null,
      FLOW:  latestData.FLOW ?? latestData.Airflow ?? null,
      NOISE: latestData.NOISE ?? null,
    });
    document.getElementById('stat-count').textContent = savedRows.length;
  }, interval);

  countdownTimer = setInterval(() => {
    const remain = saveEnd - Date.now();
    if(remain <= 0){ stopSaving(); return; }
    const m = Math.floor(remain/60000);
    const s = Math.floor((remain%60000)/1000);
    document.getElementById('stat-remain').textContent = `${m}m ${s}s`;
    const elapsed = saveDuration - remain;
    const pct = Math.min(100, Math.round(elapsed / saveDuration * 100));
    document.getElementById('progress-fill').style.width = pct+'%';
    document.getElementById('progress-pct').textContent  = pct+'%';
  }, 500);
}

function stopSaving(){
  if(!saveActive) return;
  saveActive = false;
  clearInterval(saveTimer);
  clearInterval(countdownTimer);
  document.getElementById('stat-status').textContent = 'Selesai';
  document.getElementById('stat-status').style.color = 'var(--warn)';
  document.getElementById('stat-remain').textContent = '—';
  document.getElementById('btn-save').disabled = false;
  document.getElementById('btn-stop').disabled = true;
}

function resetData(){
  if(saveActive) stopSaving();
  if(savedRows.length > 0 && !confirm('Reset semua data yang tersimpan?')) return;
  savedRows = [];
  document.getElementById('stat-count').textContent  = '0';
  document.getElementById('stat-status').textContent = 'Idle';
  document.getElementById('stat-status').style.color = '';
  document.getElementById('stat-start').textContent  = '—';
  document.getElementById('stat-remain').textContent = '—';
  document.getElementById('progress-fill').style.width = '0%';
  document.getElementById('progress-pct').textContent  = '0%';
}

// ══ Export Excel ═══════════════════════════════
function exportExcel(){
  if(savedRows.length === 0){ alert('Belum ada data tersimpan. Mulai perekaman terlebih dahulu.'); return; }

  const wb = XLSX.utils.book_new();

  // ── Sheet 1: Raw Data
  const rawHeader = ['Date','Time','T1 (°C)','T2 (°C)','T3 (°C)','T4 (°C)','T5 (°C)','TM (°C)','Humidity (%)','Airflow (m/s)','Noise (dB)'];
  const rawData   = savedRows.map(r => [r.date, r.time, r.T1, r.T2, r.T3, r.T4, r.T5, r.TM, r.RH, r.FLOW, r.NOISE]);
  const wsRaw     = XLSX.utils.aoa_to_sheet([rawHeader, ...rawData]);

  wsRaw['!cols'] = [14,10,9,9,9,9,9,9,13,12,10].map(w=>({wch:w}));

  const hdrStyle = { font:{ bold:true, color:{ rgb:'FFFFFF' }, name:'Arial' }, fill:{ fgColor:{ rgb:'1F3864' } }, alignment:{ horizontal:'center' } };
  rawHeader.forEach((_,i) => {
    const ref = XLSX.utils.encode_cell({r:0,c:i});
    if(wsRaw[ref]) wsRaw[ref].s = hdrStyle;
  });

  XLSX.utils.book_append_sheet(wb, wsRaw, 'Raw Data');

  // ── Sheet 2: Analisis Statistik
  const sensors = ['T1','T2','T3','T4','T5','TM','RH','FLOW','NOISE'];
  const labels  = { T1:'T1', T2:'T2', T3:'T3', T4:'T4', T5:'T5', TM:'TM (Matras)', RH:'Humidity', FLOW:'Airflow', NOISE:'Noise' };

  function stats(key){
    const vals = savedRows.map(r=>r[key]).filter(v=>v!==null&&!isNaN(v));
    if(!vals.length) return [null,null,null,null];
    const mean  = vals.reduce((a,b)=>a+b,0)/vals.length;
    const stdev = Math.sqrt(vals.reduce((a,b)=>a+(b-mean)**2,0)/vals.length);
    return [
      +Math.min(...vals).toFixed(4),
      +Math.max(...vals).toFixed(4),
      +stdev.toFixed(4),
      +mean.toFixed(4)
    ];
  }

  const statRows = [
    ['ANALISIS STATISTIK', null, null, null, null],
    [],
    ['Parameter','Minimal','Maksimal','STDEV','Mean'],
    [],
    ...sensors.map(k => [labels[k], ...stats(k)]),
    [],
    ['Jumlah Data', savedRows.length],
    ['Waktu Mulai', saveStart ? saveStart.toLocaleString('id-ID') : '—'],
    ['Diekspor',    new Date().toLocaleString('id-ID')],
  ];

  const wsStat = XLSX.utils.aoa_to_sheet(statRows);
  wsStat['!cols'] = [18,11,11,11,11].map(w=>({wch:w}));

  if(wsStat['A1']) wsStat['A1'].s = { font:{ bold:true, sz:14, color:{ rgb:'1F3864' }, name:'Arial' } };
  ['A3','B3','C3','D3','E3'].forEach(ref => {
    if(wsStat[ref]) wsStat[ref].s = {
      font:{ bold:true, color:{ rgb:'FFFFFF' }, name:'Arial' },
      fill:{ fgColor:{ rgb:'2E75B6' } },
      alignment:{ horizontal:'center' }
    };
  });

  XLSX.utils.book_append_sheet(wb, wsStat, 'Analisis Statistik');

  // ── Download
  const now = new Date();
  const ts  = `${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}_${String(now.getHours()).padStart(2,'0')}${String(now.getMinutes()).padStart(2,'0')}`;
  XLSX.writeFile(wb, `INCU_Data_${ts}.xlsx`);
}
</script>
</body>
</html>
