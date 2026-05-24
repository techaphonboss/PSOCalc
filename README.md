<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PSOCalc — KNN + PSO IPS Dashboard v2</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=Noto+Sans+Thai:wght@300;400;500;700;900&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#080c14;--bg2:#0d1220;--bg3:#111827;--bg4:#1a2235;
  --border:rgba(255,255,255,0.07);--border2:rgba(255,255,255,0.12);
  --text:#f0f2f7;--muted:#6b7a99;--muted2:#4a5568;
  --blue:#3b82f6;--blue-dim:rgba(59,130,246,0.12);--blue-b:rgba(59,130,246,0.3);
  --amber:#f59e0b;--amber-dim:rgba(245,158,11,0.1);--amber-b:rgba(245,158,11,0.3);
  --green:#10b981;--green-dim:rgba(16,185,129,0.1);--green-b:rgba(16,185,129,0.3);
  --rose:#f43f5e;--rose-dim:rgba(244,63,94,0.1);--rose-b:rgba(244,63,94,0.3);
  --purple:#a855f7;--purple-dim:rgba(168,85,247,0.1);--purple-b:rgba(168,85,247,0.3);
  --ok:#22c55e;--ok-dim:rgba(34,197,94,0.1);--ok-b:rgba(34,197,94,0.25);
  --err:#ef4444;--err-dim:rgba(239,68,68,0.1);--err-b:rgba(239,68,68,0.25);
  --mono:'IBM Plex Mono',monospace;--sans:'Noto Sans Thai',sans-serif;
  --r:10px;--rs:6px;
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
body{font-family:var(--sans);background:var(--bg);color:var(--text);min-height:100vh;font-size:14px;overflow-x:hidden;}
body::before{content:'';position:fixed;inset:0;z-index:0;
  background-image:linear-gradient(rgba(59,130,246,0.03) 1px,transparent 1px),linear-gradient(90deg,rgba(59,130,246,0.03) 1px,transparent 1px);
  background-size:40px 40px;pointer-events:none;}

/* ─── HEADER ─── */
.site-header{border-bottom:1px solid var(--border);background:rgba(8,12,20,0.85);backdrop-filter:blur(12px);position:sticky;top:0;z-index:100;}
.hdr{display:flex;align-items:center;justify-content:space-between;padding:14px 24px;max-width:1400px;margin:0 auto;}
.logo-badge{font-family:var(--mono);font-size:10px;font-weight:600;letter-spacing:.12em;background:var(--blue-dim);color:var(--blue);border:1px solid var(--blue-b);padding:4px 10px;border-radius:var(--rs);text-transform:uppercase;}
.logo-text{font-size:18px;font-weight:900;color:var(--text);letter-spacing:-.03em;margin-left:10px;}
.logo-text span{color:var(--blue);}
.header-sub{font-size:11px;color:var(--muted);margin-top:1px;}
#engine-status{display:inline-flex;align-items:center;gap:7px;font-size:11px;font-weight:500;font-family:var(--mono);padding:6px 14px;border-radius:99px;border:1px solid var(--border2);color:var(--muted);background:var(--bg3);transition:all .3s;}
.sdot{width:7px;height:7px;border-radius:50%;background:var(--muted2);flex-shrink:0;}
.sdot.ready{background:var(--amber);animation:pulse 2s infinite;}
.sdot.running{background:var(--blue);animation:pulse .8s infinite;}
.sdot.done{background:var(--ok);}
.sdot.error{background:var(--err);}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:.35}}

/* ─── TABS ─── */
.nav-tabs{display:flex;align-items:center;gap:2px;padding:0 24px;max-width:1400px;margin:0 auto;border-bottom:1px solid var(--border);}
.nav-tab{font-size:12px;font-weight:500;padding:10px 18px;color:var(--muted);cursor:pointer;border:none;background:none;border-bottom:2px solid transparent;transition:all .2s;white-space:nowrap;margin-bottom:-1px;}
.nav-tab:hover{color:var(--text);}
.nav-tab.active{color:var(--blue);border-bottom-color:var(--blue);}

/* ─── MAIN ─── */
.main{padding:28px 24px;max-width:1400px;margin:0 auto;}
.tab-panel{display:none;}
.tab-panel.active{display:block;}

/* ─── DROP ZONE ─── */
.drop-zone{border:1.5px dashed var(--border2);border-radius:16px;padding:48px 24px;text-align:center;cursor:pointer;background:var(--bg2);transition:all .25s;margin-bottom:32px;}
.drop-zone:hover,.drop-zone.drag-over{border-color:var(--blue);background:var(--blue-dim);}
.drop-icon{width:56px;height:56px;background:var(--bg4);border:1px solid var(--border2);border-radius:14px;display:flex;align-items:center;justify-content:center;margin:0 auto 16px;font-size:24px;transition:all .25s;}
.drop-zone:hover .drop-icon{border-color:var(--blue-b);background:var(--blue-dim);}
.drop-title{font-size:15px;font-weight:700;color:var(--text);margin-bottom:6px;}
.drop-sub{font-size:12px;color:var(--muted);}
.drop-sub strong{color:var(--blue);font-weight:600;}

.sec-title{font-size:11px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--muted);margin-bottom:14px;}

/* ─── METRIC CARDS ─── */
.metric-grid{display:grid;grid-template-columns:repeat(5,minmax(0,1fr));gap:12px;margin-bottom:24px;}
.metric-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:16px 18px;position:relative;overflow:hidden;}
.metric-accent{position:absolute;left:0;top:0;bottom:0;width:3px;border-radius:3px 0 0 3px;}
.metric-label{font-size:10px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;margin-bottom:6px;}
.metric-value{font-family:var(--mono);font-size:26px;font-weight:600;color:var(--text);line-height:1;}
.metric-desc{font-size:11px;color:var(--muted);margin-top:5px;}

/* ─── CHARTS ─── */
.charts-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:16px;margin-bottom:24px;}
.chart-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:20px;}
.chart-title{font-size:12px;font-weight:600;color:var(--text);margin-bottom:4px;}
.chart-sub{font-size:11px;color:var(--muted);margin-bottom:14px;}
.chart-wrap{height:220px;position:relative;}
.chart-wrap-tall{height:260px;position:relative;}
.chart-card-full{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:20px;margin-bottom:24px;}
.chart-head{display:flex;align-items:flex-start;justify-content:space-between;margin-bottom:14px;}

/* ─── TABLE ─── */
.table-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);overflow:hidden;margin-bottom:24px;}
.table-head{display:flex;align-items:center;justify-content:space-between;padding:14px 20px;border-bottom:1px solid var(--border);background:var(--bg3);gap:12px;flex-wrap:wrap;}
.th-title{font-size:13px;font-weight:700;color:var(--text);}
.th-sub{font-size:11px;color:var(--muted);margin-top:2px;}
.table-head-right{display:flex;align-items:center;gap:10px;flex-shrink:0;}
.count-pill{font-family:var(--mono);font-size:11px;background:var(--blue-dim);color:var(--blue);border:1px solid var(--blue-b);padding:5px 12px;border-radius:var(--rs);}
.btn{display:inline-flex;align-items:center;gap:6px;font-family:var(--sans);font-size:12px;font-weight:600;padding:7px 16px;border-radius:var(--rs);border:1px solid transparent;cursor:pointer;transition:all .15s;white-space:nowrap;}
.btn-export{background:var(--green-dim);color:var(--green);border-color:var(--green-b);}
.btn-export:hover{background:rgba(16,185,129,0.2);}
.btn-export svg{width:14px;height:14px;flex-shrink:0;}
.btn-pso{background:var(--purple-dim);color:var(--purple);border-color:var(--purple-b);}
.btn-pso:hover{background:rgba(168,85,247,0.2);}
.tbl-scroll{overflow-x:auto;max-height:460px;overflow-y:auto;}
table{width:100%;border-collapse:collapse;font-size:12px;}
thead tr{background:var(--bg3);position:sticky;top:0;z-index:2;}
th{padding:10px 14px;text-align:left;font-size:10px;font-weight:700;letter-spacing:.07em;text-transform:uppercase;white-space:nowrap;color:var(--muted);border-bottom:1px solid var(--border);}
th.blue{color:var(--blue);}th.amber{color:var(--amber);}th.green{color:var(--green);}th.rose{color:var(--rose);}th.purple{color:var(--purple);}
td{padding:9px 14px;border-bottom:1px solid var(--border);font-family:var(--mono);font-size:11px;color:var(--text);vertical-align:middle;}
tbody tr:last-child td{border-bottom:none;}
tbody tr:hover{background:rgba(255,255,255,0.025);}

.badge{display:inline-block;padding:2px 9px;border-radius:var(--rs);font-size:10px;font-weight:700;font-family:var(--mono);border:1px solid transparent;white-space:nowrap;}
.badge-ok{background:var(--ok-dim);color:var(--ok);border-color:var(--ok-b);}
.badge-err{background:var(--err-dim);color:var(--err);border-color:var(--err-b);}
.floor-pill{display:inline-block;padding:3px 10px;border-radius:var(--rs);font-size:10px;font-weight:700;font-family:var(--mono);border:1px solid transparent;white-space:nowrap;}
.fp1{background:rgba(245,158,11,.1);color:#f59e0b;border-color:rgba(245,158,11,.3);}
.fp2{background:rgba(59,130,246,.1);color:#3b82f6;border-color:rgba(59,130,246,.3);}
.fp3{background:rgba(16,185,129,.1);color:#10b981;border-color:rgba(16,185,129,.3);}
.fp4{background:rgba(168,85,247,.1);color:#a855f7;border-color:rgba(168,85,247,.3);}

/* ─── FLOOR MAP ─── */
.map-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:16px;margin-bottom:24px;}
.map-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:16px;}
.map-canvas-wrap{position:relative;width:100%;aspect-ratio:36/28;margin-top:12px;}
.map-canvas-wrap canvas{width:100%;height:100%;}

/* ─── PSO PAGE ─── */
.pso-config{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:20px;margin-bottom:20px;}
.pso-grid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:14px;margin-bottom:16px;}
.pso-field label{font-size:11px;color:var(--muted);display:block;margin-bottom:5px;font-weight:600;letter-spacing:.05em;}
.pso-field input{width:100%;background:var(--bg3);border:1px solid var(--border2);border-radius:var(--rs);color:var(--text);font-family:var(--mono);font-size:12px;padding:7px 10px;}
.pso-field input:focus{outline:none;border-color:var(--blue-b);}
.pso-result-grid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:12px;margin-bottom:20px;}
.pso-result-card{background:var(--bg3);border:1px solid var(--border);border-radius:var(--rs);padding:14px;}
.prc-label{font-size:10px;color:var(--muted);font-weight:700;letter-spacing:.08em;text-transform:uppercase;margin-bottom:6px;}
.prc-value{font-family:var(--mono);font-size:20px;font-weight:600;color:var(--text);}
.prc-sub{font-size:11px;color:var(--muted);margin-top:4px;}
.iteration-log{background:var(--bg3);border:1px solid var(--border);border-radius:var(--rs);padding:12px;max-height:200px;overflow-y:auto;font-family:var(--mono);font-size:11px;color:var(--muted);}
.iter-line{padding:2px 0;border-bottom:1px solid var(--border);}
.iter-line:last-child{border-bottom:none;}
.iter-best{color:var(--green);}

/* ─── LINE LEGEND ─── */
.line-legend{display:flex;align-items:center;gap:18px;flex-wrap:wrap;}
.ll-item{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted);}
.ll-dot{width:12px;height:3px;border-radius:2px;}

/* ─── DEV CARDS ─── */
.dev-grid{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:12px;margin-bottom:20px;}
.dev-card{background:var(--bg2);border:1px solid var(--border);border-radius:var(--r);padding:16px 18px;}
.dev-card-title{font-size:10px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;margin-bottom:10px;}
.dev-rows{display:flex;flex-direction:column;gap:6px;}
.dev-row{display:flex;align-items:center;gap:8px;}
.dev-label{font-size:11px;color:var(--muted);min-width:70px;}
.dev-count{font-family:var(--mono);font-size:13px;font-weight:600;color:var(--text);min-width:24px;text-align:right;}
.dev-bar-wrap{flex:1;height:4px;background:var(--bg4);border-radius:2px;}
.dev-bar{height:4px;border-radius:2px;transition:width .4s;}

.empty-state{text-align:center;padding:80px 24px;color:var(--muted);}
.empty-state p{margin-top:10px;font-size:13px;}
.hidden{display:none!important;}

@media(max-width:1000px){.metric-grid{grid-template-columns:repeat(3,1fr);}.charts-grid,.map-grid,.dev-grid{grid-template-columns:1fr;}.pso-grid{grid-template-columns:repeat(2,1fr);}}
@media(max-width:600px){.metric-grid{grid-template-columns:repeat(2,1fr);}.pso-grid,.pso-result-grid{grid-template-columns:1fr;}}
.anim-grid{display:grid;grid-template-columns:300px 1fr;gap:16px;align-items:start;}
@media(max-width:900px){.anim-grid{grid-template-columns:1fr;}}
.btn:disabled{opacity:0.4;cursor:not-allowed;}
select option{background:#111827;color:#f0f2f7;}
</style>
</head>
<body>

<header class="site-header">
  <div class="hdr">
    <div style="display:flex;align-items:center;gap:10px;">
      <span class="logo-badge">LIVE ENGINE v4.0</span>
      <div>
        <span class="logo-text"><span>PSO</span>Calc</span>
        <div class="header-sub">LoRa IPS — KNN + PSO Engine · AN ติดตั้งชั้น 4 คงที่ · คำนวณระยะ 3D ข้ามชั้น · อาคารคณะวิศวกรรมศาสตร์ มหาวิทยาลัยกรุงเทพธนบุรี</div>
      </div>
    </div>
    <div id="engine-status"><span class="sdot ready" id="dot"></span><span id="status-text">Ready</span></div>
  </div>
  <nav><div class="nav-tabs">
    <button class="nav-tab active" data-tab="overview">📊 ภาพรวม</button>
    <button class="nav-tab" data-tab="pso">🔬 PSO Engine</button>
    <button class="nav-tab" data-tab="animator">🎬 PSO Animator</button>
    <button class="nav-tab" data-tab="deviation">📍 คลาดเคลื่อน</button>
    <button class="nav-tab" data-tab="compare">📈 กราฟเปรียบเทียบ</button>
    <button class="nav-tab" data-tab="floormap">🗺️ แผนผังชั้น</button>
    <button class="nav-tab" data-tab="table">📋 ตารางผล</button>
  </div></nav>
</header>

<main class="main">

  <!-- ═══ TAB: OVERVIEW ═══ -->
  <div id="tab-overview" class="tab-panel active">
    <!-- UPLOAD — อยู่ใน tab overview เท่านั้น -->
    <div id="drop-zone" class="drop-zone">
      <input type="file" id="file-input" accept=".xlsx,.xls,.csv" style="display:none">
      <div class="drop-icon">📂</div>
      <div class="drop-title">ลากและหย่อนไฟล์ข้อมูลดิบที่นี่</div>
      <div class="drop-sub">รองรับ <strong>.xlsx / .xls / .csv</strong> — ข้อมูลสัญญาณ Measurement &amp; RSSI Predict</div>
    </div>
    <div id="ov-empty" class="empty-state"><div style="font-size:48px">📁</div><p>อัพโหลดไฟล์เพื่อเริ่มคำนวณ</p></div>
    <div id="ov-content" class="hidden">
      <div class="sec-title">อัตราความถูกต้อง 4 วิธีคำนวณ (KNN)</div>
      <div class="metric-grid">
        <div class="metric-card"><div class="metric-accent" style="background:var(--blue)"></div><div class="metric-label" style="color:var(--blue)">Measurement (TOP 3)</div><div class="metric-value" id="acc-m-top3">—</div><div class="metric-desc">3 ค่าน้อยสุด ÷ 3 ต่อชั้น</div></div>
        <div class="metric-card"><div class="metric-accent" style="background:var(--amber)"></div><div class="metric-label" style="color:var(--amber)">Measurement (AVG รวม)</div><div class="metric-value" id="acc-m-avg">—</div><div class="metric-desc">ค่าเฉลี่ยรวมทั้งชั้น</div></div>
        <div class="metric-card"><div class="metric-accent" style="background:var(--green)"></div><div class="metric-label" style="color:var(--green)">RSSI Predict (TOP 3)</div><div class="metric-value" id="acc-r-top3">—</div><div class="metric-desc">3 ค่าน้อยสุด ÷ 3 ต่อชั้น</div></div>
        <div class="metric-card"><div class="metric-accent" style="background:var(--rose)"></div><div class="metric-label" style="color:var(--rose)">RSSI Predict (AVG รวม)</div><div class="metric-value" id="acc-r-avg">—</div><div class="metric-desc">ค่าเฉลี่ยรวมทั้งชั้น</div></div>
        <div class="metric-card"><div class="metric-accent" style="background:var(--purple)"></div><div class="metric-label" style="color:var(--purple)">Particles โหลดแล้ว</div><div class="metric-value" id="total-loaded">55</div><div class="metric-desc">พร้อมคำนวณ PSO ทุกตัว</div></div>
      </div>
      <div class="charts-grid">
        <div class="chart-card"><div class="chart-title">อัตราความถูกต้องรวม (%)</div><div class="chart-sub">เปรียบเทียบ 4 วิธีคำนวณ KNN</div><div class="chart-wrap"><canvas id="accChart"></canvas></div></div>
        <div class="chart-card"><div class="chart-title">ความผิดพลาดแยกตามชั้น (จุด)</div><div class="chart-sub">จำนวน Particle ที่ทำนายผิดชั้น</div><div class="chart-wrap"><canvas id="errChart"></canvas></div></div>
      </div>
      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">สรุปผล ถูก vs ผิด แยกชั้น (รวม 4 วิธี)</div><div class="chart-sub">แท่งสีเขียว = ถูก, แท่งสีแดง = ผิด รวมทุกวิธี</div></div></div><div class="chart-wrap-tall"><canvas id="floorSumChart"></canvas></div></div>
    </div>
  </div>

  <!-- ═══ TAB: PSO ENGINE ═══ -->
  <div id="tab-pso" class="tab-panel">
    <div class="pso-config">
      <div class="sec-title">ตั้งค่า PSO Parameters</div>
      <!-- AN Info -->
      <div style="background:var(--bg4);border:1px solid var(--purple-b);border-radius:var(--rs);padding:10px 14px;margin-bottom:14px;font-size:12px;line-height:1.8;">
        <span style="color:var(--purple);font-weight:700;">📡 Anchor Node (ชั้น 4 คงที่)</span>
        <span style="color:var(--muted);margin-left:12px;">AN1 (7.96, 9.07)</span>
        <span style="color:var(--muted);margin-left:12px;">AN2 (28.10, 9.57)</span>
        <span style="color:var(--muted);margin-left:12px;">AN3 (8.77, 22.01)</span>
        <span style="color:var(--muted2);margin-left:16px;">|</span>
        <span style="color:var(--ok);margin-left:12px;">PLd0 = -53 dBm (calibrated 1m)</span>
        <span style="color:var(--muted);margin-left:12px;">FAF โถงเปิด=0 · มีพื้น=15 dBm/ชั้น</span>
        <span style="color:var(--blue);margin-left:12px;">→ d = √(Δx²+Δy²+Δz²)</span>
      </div>
      <div class="pso-grid">
        <div class="pso-field"><label>Swarm Size (n particles)</label><input type="number" id="pso-n" value="100" min="3" max="500"></div>
        <div class="pso-field"><label>Iterations (k)</label><input type="number" id="pso-k" value="30" min="1" max="200"></div>
        <div class="pso-field"><label>Multi-Run (ครั้ง — เฉลี่ย gbest)</label><input type="number" id="pso-runs" value="5" min="1" max="20"></div>
        <div class="pso-field"><label>w_min (inertia min)</label><input type="number" id="pso-wmin" value="0.4" step="0.05"></div>
        <div class="pso-field"><label>w_max (inertia max)</label><input type="number" id="pso-wmax" value="0.9" step="0.05"></div>
        <div class="pso-field"><label>c1 (personal coeff)</label><input type="number" id="pso-c1" value="2.0" step="0.1"></div>
        <div class="pso-field"><label>c2 (global coeff)</label><input type="number" id="pso-c2" value="2.0" step="0.1"></div>
        <div class="pso-field"><label>η โถงเปิด (LoS)</label><input type="number" id="eta-los" value="2.2" step="0.1" min="1.5" max="4.0"></div>
        <div class="pso-field"><label>η มีพื้น (NLoS)</label><input type="number" id="eta-nlos" value="3.8" step="0.1" min="2.0" max="6.0"></div>
      </div>
      <div style="display:flex;gap:10px;flex-wrap:wrap;align-items:center;">
        <button class="btn btn-pso" id="btn-run-pso">▶ รัน PSO ทุก Particle (55 ตัว)</button>
        <button class="btn" id="btn-backsolve" style="background:var(--green-dim);color:var(--green);border-color:var(--green-b);">🔍 Backsolve η จาก F4</button>
        <button class="btn" id="btn-auto-tune" style="background:var(--amber-dim);color:var(--amber);border-color:var(--amber-b);">⚡ Auto-Tune Parameters</button>
        <button class="btn btn-export" id="btn-export-pso" style="display:none">
          <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/></svg>
          Export PSO Results
        </button>
        <span id="pso-progress" style="font-size:12px;color:var(--muted);font-family:var(--mono);"></span>
      </div>
      <div id="tune-status" style="font-size:12px;color:var(--amber);margin-top:10px;font-family:var(--mono);display:none"></div>
      <pre id="tune-log" style="font-size:11px;color:var(--muted);margin-top:6px;font-family:var(--mono);background:var(--bg4);border-radius:var(--rs);padding:10px;display:none;white-space:pre-wrap;"></pre>
    </div>

    <div id="pso-results-section" class="hidden">
      <div class="sec-title">ผลลัพธ์ Hybrid KNN → PSO</div>

      <div style="display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:14px;margin-bottom:20px;">

        <!-- Measurement side -->
        <div style="background:var(--bg3);border:1px solid var(--purple-b);border-radius:var(--r);padding:16px;">
          <div style="font-size:10px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--purple);margin-bottom:12px;">
            Measurement (RSSI วัดจริง)
          </div>
          <div style="display:grid;grid-template-columns:repeat(3,1fr);gap:10px;">
            <div class="pso-result-card"><div class="prc-label" style="color:var(--purple)">KNN+PSO Accuracy</div><div class="prc-value" id="pso-acc-m">—</div><div class="prc-sub">ชั้นถูกต้อง</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--purple)">Avg. XY Error</div><div class="prc-value" id="pso-avg-err-m">—</div><div class="prc-sub">เมตร</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--purple)">Error &lt; 4m</div><div class="prc-value" id="pso-lt4-m">—</div><div class="prc-sub">จำนวน PR</div></div>
          </div>
        </div>

        <!-- RSSI Predict side -->
        <div style="background:var(--bg3);border:1px solid var(--green-b);border-radius:var(--r);padding:16px;">
          <div style="font-size:10px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--green);margin-bottom:12px;">
            RSSI Predict (diff floor model)
          </div>
          <div style="display:grid;grid-template-columns:repeat(3,1fr);gap:10px;">
            <div class="pso-result-card"><div class="prc-label" style="color:var(--green)">KNN+PSO Accuracy</div><div class="prc-value" id="pso-acc-r">—</div><div class="prc-sub">ชั้นถูกต้อง</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--green)">Avg. XY Error</div><div class="prc-value" id="pso-avg-err-r">—</div><div class="prc-sub">เมตร</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--green)">Error &lt; 4m</div><div class="prc-value" id="pso-lt4-r">—</div><div class="prc-sub">จำนวน PR</div></div>
          </div>
        </div>

      </div>

      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">PSO Fitness ลู่เข้าตามรอบ Iteration</div><div class="chart-sub">ค่า Fitness ควรลดลงทุก iteration — แสดงการ converge</div></div></div><div class="chart-wrap-tall"><canvas id="psoConvergeChart"></canvas></div></div>

      <div class="table-card">
        <div class="table-head">
          <div><div class="th-title">ผลลัพธ์ Hybrid KNN→PSO ทุก Particle</div>
          <div class="th-sub">KNN ทำนายชั้น → PSO หาพิกัด XY เฉพาะชั้นนั้น · แยก Measurement vs RSSI Predict</div></div>
          <div class="table-head-right"><span class="count-pill" id="pso-count">0</span></div>
        </div>
        <div class="tbl-scroll">
          <table><thead>
            <tr style="background:var(--bg4);">
              <th rowspan="2">Floor จริง</th>
              <th rowspan="2">Particle</th>
              <th colspan="4" style="color:var(--purple);text-align:center;border-bottom:1px solid var(--purple-b)">Measurement (RSSI วัดจริง)</th>
              <th colspan="4" style="color:var(--green);text-align:center;border-bottom:1px solid var(--green-b)">RSSI Predict (diff floor model)</th>
            </tr>
            <tr>
              <th class="purple">KNN ชั้น</th>
              <th class="purple">PSO ทำนาย</th>
              <th class="purple">gbest X,Y</th>
              <th class="purple">Error (m)</th>
              <th class="green">KNN ชั้น</th>
              <th class="green">PSO ทำนาย</th>
              <th class="green">gbest X,Y</th>
              <th class="green">Error (m)</th>
            </tr>
          </thead>
          <tbody id="pso-table-body"></tbody></table>
        </div>
      </div>
    </div>
    <div id="pso-empty" class="empty-state"><div style="font-size:48px">🔬</div><p>กดปุ่ม "รัน PSO" เพื่อเริ่มการคำนวณ<br><span style="font-size:11px;margin-top:6px;display:block">ไม่ต้องอัพโหลดไฟล์ — ข้อมูล PR ทั้ง 55 ตัวฝังอยู่ใน engine แล้ว</span></p></div>
  </div>

  <!-- ═══ TAB: DEVIATION ═══ -->
  <div id="tab-deviation" class="tab-panel">
    <div id="dev-empty" class="empty-state"><div style="font-size:48px">📍</div><p>อัพโหลดไฟล์เพื่อดูการคลาดเคลื่อน</p></div>
    <div id="dev-content" class="hidden">
      <div class="sec-title">การคลาดเคลื่อนระดับชั้น (KNN Floor Deviation)</div>
      <div class="dev-grid" id="dev-cards"></div>
      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">ปริมาณการคลาดเคลื่อน 0 / 1 / 2 / 3 ชั้น ทั้ง 4 วิธี KNN</div></div></div><div class="chart-wrap-tall"><canvas id="devChart"></canvas></div></div>
    </div>
  </div>

  <!-- ═══ TAB: COMPARE ═══ -->
  <div id="tab-compare" class="tab-panel">
    <div id="cmp-empty" class="empty-state"><div style="font-size:48px">📈</div><p>อัพโหลดไฟล์เพื่อดูกราฟเปรียบเทียบ</p></div>
    <div id="cmp-content" class="hidden">
      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">เปรียบเทียบ Measurement Avg. vs RSSI Predict Avg. แยกตามชั้น</div></div><div class="line-legend"><div class="ll-item"><div class="ll-dot" style="background:var(--amber)"></div>Meas. Avg</div><div class="ll-item"><div class="ll-dot" style="background:var(--rose)"></div>RSSI Avg</div></div></div><div class="chart-wrap-tall"><canvas id="lineAvgChart"></canvas></div></div>
      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">เปรียบเทียบ Measurement Top3 vs RSSI Predict Top3 แยกตามชั้น</div></div><div class="line-legend"><div class="ll-item"><div class="ll-dot" style="background:var(--blue)"></div>Meas. Top3</div><div class="ll-item"><div class="ll-dot" style="background:var(--green)"></div>RSSI Top3</div></div></div><div class="chart-wrap-tall"><canvas id="lineTop3Chart"></canvas></div></div>
      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">รวมทั้ง 4 วิธี KNN เปรียบเทียบกัน</div></div><div class="line-legend"><div class="ll-item"><div class="ll-dot" style="background:var(--blue)"></div>Meas. Top3</div><div class="ll-item"><div class="ll-dot" style="background:var(--amber)"></div>Meas. Avg</div><div class="ll-item"><div class="ll-dot" style="background:var(--green)"></div>RSSI Top3</div><div class="ll-item"><div class="ll-dot" style="background:var(--rose)"></div>RSSI Avg</div></div></div><div class="chart-wrap-tall"><canvas id="lineAllChart"></canvas></div></div>
    </div>
  </div>

  <!-- ═══ TAB: FLOOR MAP ═══ -->
  <div id="tab-floormap" class="tab-panel">
    <div class="sec-title">แผนผังตำแหน่ง PR และ Anchor Node แต่ละชั้น</div>
    <div style="font-size:12px;color:var(--muted);margin-bottom:16px;display:flex;gap:18px;flex-wrap:wrap;">
      <span>▲ = AN Node (ชั้น 4 เท่านั้น)</span>
      <span style="color:#22c55e">● = PR ถูก (KNN)</span>
      <span style="color:#ef4444">● = PR ผิด (KNN)</span>
      <span style="color:rgba(255,255,255,0.25)">● = ยังไม่คำนวณ</span>
      <span style="color:#f59e0b">○ = โถงเปิด (LoS)</span>
      <span style="color:#facc15">★ = gbest PSO</span>
    </div>
    <div class="map-grid">
      <div class="map-card"><div class="chart-title" style="color:#a855f7">Floor 4</div><div class="map-canvas-wrap"><canvas id="map-f4"></canvas></div></div>
      <div class="map-card"><div class="chart-title" style="color:#10b981">Floor 3</div><div class="map-canvas-wrap"><canvas id="map-f3"></canvas></div></div>
      <div class="map-card"><div class="chart-title" style="color:#3b82f6">Floor 2</div><div class="map-canvas-wrap"><canvas id="map-f2"></canvas></div></div>
      <div class="map-card"><div class="chart-title" style="color:#f59e0b">Floor 1</div><div class="map-canvas-wrap"><canvas id="map-f1"></canvas></div></div>
    </div>
    <div style="font-size:11px;color:var(--muted2);margin-top:4px;">* สีของจุด PR อัพเดทอัตโนมัติหลังรัน PSO Engine หรืออัพโหลดไฟล์ข้อมูล</div>
  </div>

  <!-- ═══ TAB: TABLE ═══ -->
  <div id="tab-table" class="tab-panel">
    <div id="tbl-empty" class="empty-state"><div style="font-size:48px">📋</div><p>อัพโหลดไฟล์เพื่อดูตารางผล KNN</p></div>
    <div id="tbl-content" class="hidden">
      <div class="table-card">
        <div class="table-head">
          <div><div class="th-title">ตารางผล KNN ทั้ง 4 วิธี</div><div class="th-sub">Measurement (TOP 3 / AVG) · RSSI Predict (TOP 3 / AVG)</div></div>
          <div class="table-head-right">
            <span class="count-pill" id="total-records">0</span>
            <button class="btn btn-export" id="btn-export">
              <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/></svg>
              Export Excel (3 Sheets)
            </button>
          </div>
        </div>
        <div class="tbl-scroll">
          <table><thead><tr>
            <th>Floor จริง</th><th>Particle</th>
            <th class="blue">Measurement (TOP 3)</th>
            <th class="amber">Measurement (AVG รวม)</th>
            <th class="green">RSSI Predict (TOP 3)</th>
            <th class="rose">RSSI Predict (AVG รวม)</th>
            <th>คลาดเคลื่อนสูงสุด</th>
          </tr></thead>
          <tbody id="table-body"></tbody></table>
        </div>
      </div>
    </div>
  </div>

  <!-- ═══ TAB: PSO ANIMATOR ═══ -->
  <div id="tab-animator" class="tab-panel">
    <div class="anim-grid">

      <!-- LEFT: Controls -->
      <div>
        <!-- Mobile Selector -->
        <div class="chart-card" style="margin-bottom:14px;">
          <div class="chart-title" style="margin-bottom:10px;">เลือก Mobile (PR ที่ใช้ทดสอบ)</div>
          <select id="anim-mobile-select" style="width:100%;background:var(--bg3);border:1px solid var(--border2);border-radius:var(--rs);color:var(--text);font-family:var(--mono);font-size:12px;padding:7px 10px;margin-bottom:10px;">
            <option value="">-- เลือก PR ที่ต้องการดู --</option>
          </select>
          <div id="anim-mobile-info" style="font-size:11px;color:var(--muted);line-height:1.8;display:none;background:var(--bg4);border-radius:var(--rs);padding:8px 10px;"></div>
        </div>

        <!-- RSSI Mode -->
        <div class="chart-card" style="margin-bottom:14px;">
          <div class="chart-title" style="margin-bottom:8px;">โหมด RSSI</div>
          <div style="display:flex;gap:8px;">
            <label style="display:flex;align-items:center;gap:6px;font-size:12px;color:var(--muted);cursor:pointer;">
              <input type="radio" name="anim-rssi-mode" value="meas" checked style="accent-color:var(--purple)"> Measurement
            </label>
            <label style="display:flex;align-items:center;gap:6px;font-size:12px;color:var(--muted);cursor:pointer;">
              <input type="radio" name="anim-rssi-mode" value="pred" style="accent-color:var(--green)"> RSSI Predict
            </label>
          </div>
        </div>

        <!-- PSO Params for animation -->
        <div class="chart-card" style="margin-bottom:14px;">
          <div class="chart-title" style="margin-bottom:10px;">PSO Parameters (Animation)</div>
          <div style="display:grid;grid-template-columns:1fr 1fr;gap:8px;">
            <div class="pso-field"><label>n particles</label><input type="number" id="anim-n" value="30" min="5" max="100"></div>
            <div class="pso-field"><label>k iterations</label><input type="number" id="anim-k" value="20" min="3" max="100"></div>
            <div class="pso-field"><label>Speed (ms/frame)</label><input type="number" id="anim-speed" value="300" min="50" max="2000" step="50"></div>
            <div class="pso-field"><label>η (auto/manual)</label><input type="number" id="anim-eta" value="2.2" step="0.1" min="1.0" max="6.0"></div>
          </div>
        </div>

        <!-- Controls -->
        <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:14px;">
          <button id="anim-btn-init" class="btn btn-pso" style="flex:1;" disabled>⚡ เตรียม PSO</button>
          <button id="anim-btn-play" class="btn" style="flex:1;background:var(--green-dim);color:var(--green);border-color:var(--green-b);" disabled>▶ เล่น</button>
          <button id="anim-btn-pause" class="btn" style="flex:1;background:var(--amber-dim);color:var(--amber);border-color:var(--amber-b);" disabled>⏸ หยุด</button>
          <button id="anim-btn-reset" class="btn" style="flex:1;background:var(--err-dim);color:var(--err);border-color:var(--err-b);" disabled>↺ Reset</button>
        </div>
        <div style="display:flex;gap:8px;margin-bottom:14px;">
          <button id="anim-btn-prev" class="btn" style="flex:1;background:var(--bg3);color:var(--muted);border-color:var(--border2);" disabled>◀ ก่อนหน้า</button>
          <button id="anim-btn-next" class="btn" style="flex:1;background:var(--bg3);color:var(--muted);border-color:var(--border2);" disabled>▶ ถัดไป</button>
        </div>

        <!-- Iteration info -->
        <div class="chart-card" style="margin-bottom:14px;">
          <div class="chart-title">สถานะ Iteration</div>
          <div id="anim-iter-display" style="font-family:var(--mono);font-size:12px;line-height:2;margin-top:6px;">
            <div>รอบที่: <span id="anim-iter-num" style="color:var(--blue)">—</span> / <span id="anim-iter-total">—</span></div>
            <div>gbest cost: <span id="anim-gbest-cost" style="color:var(--green)">—</span></div>
            <div>gbest X,Y: <span id="anim-gbest-xy" style="color:var(--purple)">—</span></div>
            <div>error ถึงจริง: <span id="anim-gbest-err" style="color:var(--amber)">—</span> m</div>
          </div>
        </div>

        <!-- Per-iteration table -->
        <div class="chart-card">
          <div class="chart-title" style="margin-bottom:8px;">บันทึกทุก Iteration</div>
          <div style="max-height:240px;overflow-y:auto;">
            <table style="width:100%;font-size:11px;border-collapse:collapse;">
              <thead><tr style="background:var(--bg4);">
                <th style="padding:5px 8px;color:var(--muted);text-align:left;">k</th>
                <th style="padding:5px 8px;color:var(--blue);text-align:left;">gbest X</th>
                <th style="padding:5px 8px;color:var(--blue);text-align:left;">gbest Y</th>
                <th style="padding:5px 8px;color:var(--green);text-align:left;">cost</th>
                <th style="padding:5px 8px;color:var(--amber);text-align:left;">err(m)</th>
              </tr></thead>
              <tbody id="anim-iter-log" style="font-family:var(--mono);"></tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- RIGHT: Canvas + chart -->
      <div>
        <!-- Floor map canvas -->
        <div class="chart-card" style="margin-bottom:14px;">
          <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:10px;">
            <div>
              <div class="chart-title">แผนที่การเคลื่อนที่ Particle</div>
              <div class="chart-sub" id="anim-floor-label">เลือก Mobile เพื่อเริ่ม</div>
            </div>
            <div style="display:flex;gap:12px;font-size:11px;flex-wrap:wrap;">
              <span style="color:var(--muted)">● Particle</span>
              <span style="color:#facc15">★ gbest</span>
              <span style="color:#22c55e">✦ Mobile จริง</span>
              <span style="color:var(--purple)">▲ AN Node</span>
              <span style="color:#60a5fa">○ PR อื่น</span>
            </div>
          </div>
          <div style="position:relative;width:100%;height:0;padding-top:75%;background:var(--bg3);border-radius:var(--rs);overflow:hidden;">
            <canvas id="anim-canvas" style="position:absolute;top:0;left:0;width:100%;height:100%;display:block;"></canvas>
          </div>
        </div>

        <!-- Convergence chart -->
        <div class="chart-card">
          <div class="chart-title" style="margin-bottom:10px;">กราฟ Fitness ลู่เข้า (Convergence)</div>
          <div style="height:180px;position:relative;"><canvas id="anim-conv-chart"></canvas></div>
        </div>
      </div>
    </div>
  </div>

</main>

<script>
(function(){
"use strict";

// ══════════════════════════════════════════════
// EMBEDDED DATABASE — PR coordinates + RSSI
// ══════════════════════════════════════════════
const ANCHORS = [
  {id:'AN1', x:7.96,  y:9.07},
  {id:'AN2', x:28.10, y:9.57},
  {id:'AN3', x:8.77,  y:22.01},
];

const PR_DB = [
  {floor:'Floor 4',id:'PR1', x:13.23,y:11.32,mRSSI:[-59.190,-63.042,-66.214],pRSSI:[-68.163,-76.506,-74.276]},
  {floor:'Floor 4',id:'PR2', x:2.09, y:10.65,mRSSI:[-64.607,-66.290,-69.852],pRSSI:[-68.677,-81.310,-75.397]},
  {floor:'Floor 4',id:'PR3', x:1.66, y:2.18, mRSSI:[-60.217,-76.087,-82.529],pRSSI:[-72.403,-81.772,-79.472]},
  {floor:'Floor 4',id:'PR4', x:13.95,y:3.33, mRSSI:[-81.694,-60.838,-72.392],pRSSI:[-71.378,-76.787,-78.749]},
  {floor:'Floor 4',id:'PR5', x:22.21,y:11.51,mRSSI:[-64.111,-58.103,-66.111],pRSSI:[-76.202,-68.850,-77.637]},
  {floor:'Floor 4',id:'PR6', x:22.07,y:2.56, mRSSI:[-66.588,-57.000,-78.529],pRSSI:[-76.829,-72.320,-80.444]},
  {floor:'Floor 4',id:'PR7', x:33.78,y:12.28,mRSSI:[-77.941,-50.087,-77.000],pRSSI:[-81.306,-68.978,-81.574]},
  {floor:'Floor 4',id:'PR8', x:34.04,y:3.23, mRSSI:[-89.000,-70.348,-89.609],pRSSI:[-81.539,-71.778,-82.962]},
  {floor:'Floor 4',id:'PR9', x:13.81,y:20.25,mRSSI:[-65.133,-78.450,-56.947],pRSSI:[-75.020,-78.028,-67.548]},
  {floor:'Floor 4',id:'PR10',x:13.40,y:24.14,mRSSI:[-71.333,-69.059,-55.167],pRSSI:[-77.094,-79.318,-67.145]},
  {floor:'Floor 4',id:'PR11',x:1.98, y:23.73,mRSSI:[-77.938,-82.043,-60.000],pRSSI:[-76.991,-82.458,-69.907]},
  {floor:'Floor 4',id:'PR12',x:3.18, y:19.85,mRSSI:[-73.417,-86.071,-61.214],pRSSI:[-74.432,-81.613,-68.553]},
  {floor:'Floor 4',id:'PR13',x:8.97, y:15.15,mRSSI:[-60.133,-68.611,-62.056],pRSSI:[-68.796,-78.989,-69.730]},
  {floor:'Floor 4',id:'PR14',x:18.03,y:17.03,mRSSI:[-79.500,-64.176,-64.643],pRSSI:[-75.169,-74.961,-73.436]},
  {floor:'Floor 4',id:'PR15',x:30.20,y:15.99,mRSSI:[-78.333,-67.611,-66.929],pRSSI:[-80.344,-69.592,-79.950]},
  {floor:'Floor 4',id:'PR16',x:18.09,y:7.44, mRSSI:[-64.053,-60.000,-70.960],pRSSI:[-73.223,-73.201,-77.759]},
  {floor:'Floor 3',id:'PR17',x:22.21,y:11.51,mRSSI:[-79.529,-52.000,-74.706],pRSSI:[-79.046,-73.087,-80.293]},
  {floor:'Floor 3',id:'PR18',x:33.78,y:12.28,mRSSI:[-77.194,-54.571,-92.636],pRSSI:[-83.527,-73.180,-83.765]},
  {floor:'Floor 3',id:'PR19',x:34.04,y:3.23, mRSSI:[-80.611,-66.364,-91.412],pRSSI:[-83.734,-75.341,-85.001]},
  {floor:'Floor 3',id:'PR20',x:22.07,y:2.56, mRSSI:[-83.737,-71.500,-90.471],pRSSI:[-79.589,-75.779,-82.763]},
  {floor:'Floor 3',id:'PR21',x:13.23,y:11.32,mRSSI:[-65.727,-75.353,-69.000],pRSSI:[-72.594,-79.309,-77.403]},
  {floor:'Floor 3',id:'PR22',x:2.09, y:10.65,mRSSI:[-70.105,-88.154,-74.944],pRSSI:[-72.961,-83.531,-78.355]},
  {floor:'Floor 3',id:'PR23',x:1.66, y:2.18, mRSSI:[-67.167,-86.200,-89.167],pRSSI:[-75.847,-83.941,-81.903]},
  {floor:'Floor 3',id:'PR24',x:13.95,y:3.33, mRSSI:[-71.211,-76.563,-78.000],pRSSI:[-75.020,-79.553,-81.267]},
  {floor:'Floor 3',id:'PR25',x:13.81,y:20.25,mRSSI:[-85.370,-80.947,-56.267],pRSSI:[-78.033,-80.634,-72.166]},
  {floor:'Floor 3',id:'PR26',x:13.40,y:24.14,mRSSI:[-79.737,-87.867,-64.600],pRSSI:[-79.820,-81.768,-71.894]},
  {floor:'Floor 3',id:'PR27',x:1.98, y:23.73,mRSSI:[-77.263,-87.000,-58.529],pRSSI:[-79.730,-84.552,-73.875]},
  {floor:'Floor 3',id:'PR28',x:3.18, y:19.85,mRSSI:[-75.867,-87.000,-55.765],pRSSI:[-77.534,-83.800,-72.871]},
  {floor:'Floor 3',id:'PR29',x:8.97, y:15.15,mRSSI:[-76.286,-80.471,-78.947],pRSSI:[-73.048,-81.478,-73.741]},
  {floor:'Floor 3',id:'PR30',x:18.03,y:17.03,mRSSI:[-75.450,-80.857,-76.813],pRSSI:[-78.160,-77.982,-76.698]},
  {floor:'Floor 3',id:'PR31',x:30.20,y:15.99,mRSSI:[-101.706,-65.000,-77.857],pRSSI:[-82.674,-73.637,-82.326]},
  {floor:'Floor 3',id:'PR32',x:18.09,y:7.44, mRSSI:[-73.061,-74.731,-85.000],pRSSI:[-76.522,-76.503,-80.399]},
  {floor:'Floor 2',id:'PR33',x:13.81,y:20.25,mRSSI:[-86.357,-85.000,-74.500],pRSSI:[-81.324,-83.395,-77.476]},
  {floor:'Floor 2',id:'PR34',x:13.40,y:24.14,mRSSI:[-100.056,-86.615,-73.118],pRSSI:[-82.731,-84.335,-77.338]},
  {floor:'Floor 2',id:'PR35',x:1.98, y:23.73,mRSSI:[-93.222,-99.833,-71.071],pRSSI:[-82.659,-86.706,-78.438]},
  {floor:'Floor 2',id:'PR36',x:3.18, y:19.85,mRSSI:[-87.400,-95.667,-59.000],pRSSI:[-80.945,-86.059,-77.854]},
  {floor:'Floor 2',id:'PR37',x:8.97, y:15.15,mRSSI:[-74.333,-91.667,-71.133],pRSSI:[-77.953,-84.093,-78.357]},
  {floor:'Floor 2',id:'PR38',x:18.03,y:17.03,mRSSI:[-75.176,-80.938,-83.563],pRSSI:[-81.422,-81.285,-80.328]},
  {floor:'Floor 2',id:'PR39',x:30.20,y:15.99,mRSSI:[-92.444,-73.667,-80.733],pRSSI:[-85.098,-78.295,-84.804]},
  {floor:'Floor 1',id:'PR40',x:13.23,y:11.32,mRSSI:[-70.364,-70.944,-78.000],pRSSI:[-82.394,-85.392,-84.329]},
  {floor:'Floor 1',id:'PR41',x:2.09, y:10.65,mRSSI:[-67.750,-81.421,-79.100],pRSSI:[-82.502,-88.160,-84.842]},
  {floor:'Floor 1',id:'PR42',x:1.66, y:2.18, mRSSI:[-63.682,-84.389,-75.444],pRSSI:[-83.580,-88.451,-87.040]},
  {floor:'Floor 1',id:'PR43',x:13.95,y:3.33, mRSSI:[-76.353,-81.308,-77.667],pRSSI:[-83.229,-85.538,-86.618]},
  {floor:'Floor 1',id:'PR44',x:22.21,y:11.51,mRSSI:[-79.583,-73.063,-94.400],pRSSI:[-85.237,-82.540,-85.993]},
  {floor:'Floor 1',id:'PR45',x:22.07,y:2.56, mRSSI:[-79.474,-82.071,-91.000],pRSSI:[-85.559,-83.550,-87.625]},
  {floor:'Floor 1',id:'PR46',x:33.78,y:12.28,mRSSI:[-83.960,-70.955,-89.526],pRSSI:[-88.304,-83.361,-89.212]},
  {floor:'Floor 1',id:'PR47',x:34.04,y:3.23, mRSSI:[-85.042,-67.000,-93.045],pRSSI:[-88.158,-82.570,-88.326]},
  {floor:'Floor 1',id:'PR48',x:13.81,y:20.25,mRSSI:[-98.643,-98.714,-84.000],pRSSI:[-84.664,-86.209,-82.276]},
  {floor:'Floor 1',id:'PR49',x:13.40,y:24.14,mRSSI:[-92.267,-96.933,-85.000],pRSSI:[-85.700,-86.949,-82.205]},
  {floor:'Floor 1',id:'PR50',x:1.98, y:23.73,mRSSI:[-101.429,-107.923,-93.467],pRSSI:[-85.645,-88.888,-82.799]},
  {floor:'Floor 1',id:'PR51',x:3.18, y:19.85,mRSSI:[-105.063,-110.118,-88.000],pRSSI:[-84.398,-88.351,-82.475]},
  {floor:'Floor 1',id:'PR52',x:8.97, y:15.15,mRSSI:[-91.813,-98.882,-90.000],pRSSI:[-82.528,-86.757,-82.753]},
  {floor:'Floor 1',id:'PR53',x:18.03,y:17.03,mRSSI:[-100.737,-90.000,-100.188],pRSSI:[-84.734,-84.637,-83.976]},
  {floor:'Floor 1',id:'PR54',x:30.20,y:15.99,mRSSI:[-103.647,-97.846,-100.000],pRSSI:[-87.564,-82.718,-87.326]},
  {floor:'Floor 1',id:'PR55',x:18.09,y:7.44, mRSSI:[-87.000,-84.083,-88.882],pRSSI:[-83.891,-83.882,-86.060]},
];

const FLOORS = ['Floor 1','Floor 2','Floor 3','Floor 4'];
const C = ['#3b82f6','#f59e0b','#10b981','#f43f5e'];
const GRID_C = 'rgba(255,255,255,0.06)';
const TICK_C = '#6b7a99';

var knnLogs = [];
var psoLogs = [];
var charts = {};
var psoIterAvg = [];

// ── TABS ──
document.querySelectorAll('.nav-tab').forEach(function(btn){
  btn.addEventListener('click',function(){
    document.querySelectorAll('.nav-tab').forEach(function(b){b.classList.remove('active');});
    btn.classList.add('active');
    var id = btn.dataset.tab;
    document.querySelectorAll('.tab-panel').forEach(function(p){p.classList.remove('active');});
    document.getElementById('tab-'+id).classList.add('active');
    if(id==='floormap') renderMaps();
    // reset canvas size cache เมื่อเปิด animator tab
    if(id==='animator'){
      var el = document.getElementById('anim-canvas');
      if(el) el.dataset.w = '';
    }
  });
});

function setStatus(state,txt){
  document.getElementById('dot').className='sdot '+state;
  document.getElementById('status-text').textContent=txt;
}

// ── DROP ZONE ──
var dz=document.getElementById('drop-zone');
var fi=document.getElementById('file-input');
dz.addEventListener('click',function(){fi.click();});
dz.addEventListener('dragover',function(e){e.preventDefault();dz.classList.add('drag-over');});
dz.addEventListener('dragleave',function(){dz.classList.remove('drag-over');});
dz.addEventListener('drop',function(e){e.preventDefault();dz.classList.remove('drag-over');if(e.dataTransfer.files.length)handleFile(e.dataTransfer.files[0]);});
fi.addEventListener('change',function(e){if(e.target.files.length)handleFile(e.target.files[0]);});

function handleFile(file){
  setStatus('running','กำลังประมวลผล...');
  var r=new FileReader();
  r.onload=function(e){
    try{
      var wb=XLSX.read(new Uint8Array(e.target.result),{type:'array'});
      var ws=wb.Sheets[wb.SheetNames[0]];
      var rows=XLSX.utils.sheet_to_json(ws,{header:1});
      runKNN(rows);
    }catch(err){setStatus('error','Error: '+err.message);}
  };
  r.readAsArrayBuffer(file);
}

// ══ KNN ENGINE ══
function runKNN(rawRows){
  var data=[];
  rawRows.forEach(function(row){
    if(!row||!row[0])return;
    var fs=String(row[0]).trim();
    if(!fs.startsWith('Floor'))return;
    var pid=row[1]?String(row[1]).trim():'';
    if(pid.toLowerCase()==='particle')return;
    var m1=parseFloat(row[2]),m2=parseFloat(row[3]),m3=parseFloat(row[4]);
    var r1=parseFloat(row[5]),r2=parseFloat(row[6]),r3=parseFloat(row[7]);
    if(isNaN(m1)||isNaN(m2)||isNaN(m3))return;
    data.push({floor:fs,id:pid,meas:[m1,m2,m3],rssi:[isNaN(r1)?0:r1,isNaN(r2)?0:r2,isNaN(r3)?0:r3]});
  });
  if(!data.length){setStatus('error','ไม่พบข้อมูล');return;}

  knnLogs=[];
  data.forEach(function(mobile){
    var af=mobile.floor;
    var mD={},rD={};
    FLOORS.forEach(function(f){mD[f]=[];rD[f]=[];});
    data.forEach(function(t){
      var tf=t.floor;
      if(!mD[tf])return;
      mD[tf].push(euclid3(mobile.meas,t.meas));
      rD[tf].push(euclid3(mobile.rssi,t.rssi));
    });
    var pMT=predKNN(mD,true,af),pMA=predKNN(mD,false,af);
    var pRT=predKNN(rD,true,af),pRA=predKNN(rD,false,af);
    function res(p){return p===af?'ถูก':'ผิด→'+p.replace('Floor','ชั้น');}
    function gap(p){var a=parseInt(af.replace('Floor ','')),b=parseInt(p.replace('Floor ',''));return isNaN(a)||isNaN(b)?0:Math.abs(a-b);}
    knnLogs.push({floor:af,id:mobile.id,mTop3:res(pMT),mAvg:res(pMA),rTop3:res(pRT),rAvg:res(pRA),gMT3:gap(pMT),gMA:gap(pMA),gRT3:gap(pRT),gRA:gap(pRA)});
  });

  renderKNNAll();
  setStatus('done','KNN เสร็จ — '+knnLogs.length+' Particles');
}

function euclid3(a,b){return Math.sqrt((a[0]-b[0])**2+(a[1]-b[1])**2+(a[2]-b[2])**2);}

function predKNN(dm,top3,af){
  var s={};
  Object.keys(dm).forEach(function(fl){
    var d=dm[fl];
    if(!d.length){s[fl]=999;return;}
    if(top3){
      var sorted=d.slice().sort(function(a,b){return a-b;});
      var f=sorted.filter(function(v){return v>0;});
      if(!f.length)f=sorted;
      s[fl]=f.length>=3?(f[0]+f[1]+f[2])/3:f[0]||999;
    }else{s[fl]=d.reduce(function(a,b){return a+b;},0)/d.length;}
  });
  return Object.keys(s).reduce(function(a,b){return s[a]<s[b]?a:b;});
}

function renderKNNAll(){
  renderOverview();renderDeviation();renderCompare();renderKNNTable();
  ['ov','dev','cmp','tbl'].forEach(function(k){
    document.getElementById(k+'-empty').classList.add('hidden');
    document.getElementById(k+'-content').classList.remove('hidden');
  });
  // ซ่อน drop zone หลังโหลดไฟล์สำเร็จ
  var dz2 = document.getElementById('drop-zone');
  if(dz2) dz2.style.display='none';
  renderMaps();
}

// ── Overview charts ──
function renderOverview(){
  var t=knnLogs.length;
  var ok=function(k){return knnLogs.filter(function(l){return l[k]==='ถูก';}).length;};
  var okMT=ok('mTop3'),okMA=ok('mAvg'),okRT=ok('rTop3'),okRA=ok('rAvg');
  var pct=function(v){return ((v/t)*100).toFixed(1)+'%';};
  document.getElementById('acc-m-top3').textContent=pct(okMT);
  document.getElementById('acc-m-avg').textContent=pct(okMA);
  document.getElementById('acc-r-top3').textContent=pct(okRT);
  document.getElementById('acc-r-avg').textContent=pct(okRA);

  mkChart('accChart',{type:'bar',data:{labels:['Meas. Top3','Meas. Avg','RSSI Top3','RSSI Avg'],datasets:[{data:[+(okMT/t*100).toFixed(1),+(okMA/t*100).toFixed(1),+(okRT/t*100).toFixed(1),+(okRA/t*100).toFixed(1)],backgroundColor:C,borderRadius:6,borderSkipped:false}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},tooltip:{callbacks:{label:function(c){return c.parsed.y.toFixed(1)+'%';}}}},scales:{y:{min:0,max:100,grid:{color:GRID_C},ticks:{color:TICK_C,callback:function(v){return v+'%';}}},x:{grid:{display:false},ticks:{color:TICK_C,font:{size:10}}}}}});

  var errF={};FLOORS.forEach(function(f){errF[f]={mt:0,ma:0,rt:0,ra:0};});
  knnLogs.forEach(function(l){if(!errF[l.floor])return;if(l.mTop3!=='ถูก')errF[l.floor].mt++;if(l.mAvg!=='ถูก')errF[l.floor].ma++;if(l.rTop3!=='ถูก')errF[l.floor].rt++;if(l.rAvg!=='ถูก')errF[l.floor].ra++;});
  mkChart('errChart',{type:'bar',data:{labels:FLOORS,datasets:[{label:'Meas.Top3',data:FLOORS.map(function(f){return errF[f].mt;}),backgroundColor:C[0],borderRadius:4},{label:'Meas.Avg',data:FLOORS.map(function(f){return errF[f].ma;}),backgroundColor:C[1],borderRadius:4},{label:'RSSI Top3',data:FLOORS.map(function(f){return errF[f].rt;}),backgroundColor:C[2],borderRadius:4},{label:'RSSI Avg',data:FLOORS.map(function(f){return errF[f].ra;}),backgroundColor:C[3],borderRadius:4}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:true,labels:{color:TICK_C,font:{size:10},boxWidth:10,padding:10}}},scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C,stepSize:1}},x:{grid:{display:false},ticks:{color:TICK_C}}}}});

  var okF={},wF={};FLOORS.forEach(function(f){okF[f]=0;wF[f]=0;});
  knnLogs.forEach(function(l){if(!okF[l.floor])return;var c=[l.mTop3,l.mAvg,l.rTop3,l.rAvg].filter(function(v){return v==='ถูก';}).length;okF[l.floor]+=c;wF[l.floor]+=(4-c);});
  mkChart('floorSumChart',{type:'bar',data:{labels:FLOORS,datasets:[{label:'ถูก (รวม 4 วิธี)',data:FLOORS.map(function(f){return okF[f];}),backgroundColor:'rgba(34,197,94,0.7)',borderRadius:5},{label:'ผิด (รวม 4 วิธี)',data:FLOORS.map(function(f){return wF[f];}),backgroundColor:'rgba(239,68,68,0.6)',borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:true,labels:{color:TICK_C,font:{size:11},boxWidth:12,padding:14}}},scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C}},x:{grid:{display:false},ticks:{color:TICK_C}}}}});
}

// ── Deviation ──
function renderDeviation(){
  var methods=[{key:'gMT3',label:'Meas.Top3',color:C[0]},{key:'gMA',label:'Meas.Avg',color:C[1]},{key:'gRT3',label:'RSSI Top3',color:C[2]},{key:'gRA',label:'RSSI Avg',color:C[3]}];
  var dc=document.getElementById('dev-cards');dc.innerHTML='';
  var allCounts=[];
  methods.forEach(function(m){
    var d={0:0,1:0,2:0,3:0};
    knnLogs.forEach(function(l){var v=l[m.key];if(d[v]!==undefined)d[v]++;});
    allCounts.push(d);
    var c=document.createElement('div');c.className='dev-card';
    var maxV=Math.max(d[0],d[1],d[2],d[3])||1;
    c.innerHTML='<div class="dev-card-title" style="color:'+m.color+'">'+m.label+'</div><div class="dev-rows">'+
      [['✅ ถูกชั้น',d[0],'#22c55e'],['+1 ชั้น',d[1],C[1]],['+2 ชั้น',d[2],C[3]],['+3 ชั้น',d[3],'#ef4444']].map(function(r){
        return '<div class="dev-row"><span class="dev-label">'+r[0]+'</span><div class="dev-bar-wrap"><div class="dev-bar" style="width:'+((r[1]/maxV)*100).toFixed(0)+'%;background:'+r[2]+'"></div></div><span class="dev-count">'+r[1]+'</span></div>';
      }).join('')+'</div>';
    dc.appendChild(c);
  });
  mkChart('devChart',{type:'bar',data:{labels:['0 ชั้น (ถูก)','+1 ชั้น','+2 ชั้น','+3 ชั้น'],datasets:[{label:'Meas.Top3',data:[allCounts[0][0],allCounts[0][1],allCounts[0][2],allCounts[0][3]],backgroundColor:C[0],borderRadius:5},{label:'Meas.Avg',data:[allCounts[1][0],allCounts[1][1],allCounts[1][2],allCounts[1][3]],backgroundColor:C[1],borderRadius:5},{label:'RSSI Top3',data:[allCounts[2][0],allCounts[2][1],allCounts[2][2],allCounts[2][3]],backgroundColor:C[2],borderRadius:5},{label:'RSSI Avg',data:[allCounts[3][0],allCounts[3][1],allCounts[3][2],allCounts[3][3]],backgroundColor:C[3],borderRadius:5}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:true,labels:{color:TICK_C,font:{size:11},boxWidth:12,padding:12}}},scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C,stepSize:1}},x:{grid:{display:false},ticks:{color:TICK_C}}}}});
}

// ── Compare Line ──
function renderCompare(){
  function floorPct(key,fl){var sub=knnLogs.filter(function(l){return l.floor===fl;});if(!sub.length)return 0;return +((sub.filter(function(l){return l[key]==='ถูก';}).length/sub.length)*100).toFixed(1);}
  function lineDsOpts(color){return{borderColor:color,backgroundColor:color.replace('rgb','rgba').replace(')',',0.08)'),pointBackgroundColor:color,pointRadius:5,pointHoverRadius:7,fill:true,tension:.35,borderWidth:2};}
  var lOpts=function(){return{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:true,labels:{color:TICK_C,font:{size:11},boxWidth:12,padding:12}},tooltip:{callbacks:{label:function(c){return c.dataset.label+': '+c.parsed.y.toFixed(1)+'%';}}}},scales:{y:{min:0,max:100,grid:{color:GRID_C},ticks:{color:TICK_C,callback:function(v){return v+'%';}}},x:{grid:{color:GRID_C},ticks:{color:TICK_C}}}};};
  mkChart('lineAvgChart',{type:'line',data:{labels:FLOORS,datasets:[Object.assign({label:'Meas.Avg',data:FLOORS.map(function(f){return floorPct('mAvg',f);})},lineDsOpts(C[1])),Object.assign({label:'RSSI Avg',data:FLOORS.map(function(f){return floorPct('rAvg',f);})},lineDsOpts(C[3]))]},options:lOpts()});
  mkChart('lineTop3Chart',{type:'line',data:{labels:FLOORS,datasets:[Object.assign({label:'Meas.Top3',data:FLOORS.map(function(f){return floorPct('mTop3',f);})},lineDsOpts(C[0])),Object.assign({label:'RSSI Top3',data:FLOORS.map(function(f){return floorPct('rTop3',f);})},lineDsOpts(C[2]))]},options:lOpts()});
  mkChart('lineAllChart',{type:'line',data:{labels:FLOORS,datasets:[Object.assign({label:'Meas.Top3',data:FLOORS.map(function(f){return floorPct('mTop3',f);})},lineDsOpts(C[0])),Object.assign({label:'Meas.Avg',data:FLOORS.map(function(f){return floorPct('mAvg',f);})},lineDsOpts(C[1])),Object.assign({label:'RSSI Top3',data:FLOORS.map(function(f){return floorPct('rTop3',f);})},lineDsOpts(C[2])),Object.assign({label:'RSSI Avg',data:FLOORS.map(function(f){return floorPct('rAvg',f);})},lineDsOpts(C[3]))]},options:lOpts()});
}

// ── KNN Table ──
function renderKNNTable(){
  document.getElementById('total-records').textContent=knnLogs.length+' Particles';
  var fp={'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
  var tbody=document.getElementById('table-body');tbody.innerHTML='';
  knnLogs.forEach(function(log){
    var fn=log.floor.replace('Floor ','');
    var maxG=Math.max(log.gMT3,log.gMA,log.gRT3,log.gRA);
    var gc=maxG===0?'var(--ok)':maxG===1?'var(--amber)':'var(--err)';
    tbody.innerHTML+='<tr><td><span class="floor-pill '+(fp[fn]||'')+'">'+log.floor+'</span></td><td style="color:var(--muted)">'+log.id+'</td><td>'+badge(log.mTop3)+'</td><td>'+badge(log.mAvg)+'</td><td>'+badge(log.rTop3)+'</td><td>'+badge(log.rAvg)+'</td><td><span style="font-family:var(--mono);font-size:11px;color:'+gc+'">'+(maxG===0?'ถูกทุกวิธี':maxG+'ชั้น')+'</span></td></tr>';
  });
}

function badge(v){if(v==='ถูก')return '<span class="badge badge-ok">ถูก</span>';var p=v.split('→');return '<span class="badge badge-err">'+(p[1]||v)+'</span>';}

// ══ PSO ENGINE ══
document.getElementById('btn-run-pso').addEventListener('click',runPSOAll);

function runPSOAll(){
  var n    = parseInt(document.getElementById('pso-n').value)    || 100;
  var K    = parseInt(document.getElementById('pso-k').value)    || 30;
  var R    = parseInt(document.getElementById('pso-runs').value) || 5;
  var wmin = parseFloat(document.getElementById('pso-wmin').value) || 0.4;
  var wmax = parseFloat(document.getElementById('pso-wmax').value) || 0.9;
  var c1   = parseFloat(document.getElementById('pso-c1').value)   || 2.0;
  var c2   = parseFloat(document.getElementById('pso-c2').value)   || 2.0;
  var etaLos  = parseFloat(document.getElementById('eta-los').value)  || 2.2;
  var etaNlos = parseFloat(document.getElementById('eta-nlos').value) || 3.8;

  setStatus('running','Hybrid KNN→PSO กำลังประมวลผล...');
  document.getElementById('pso-progress').textContent =
    'PLd0=-53 · ηLoS='+etaLos+' · ηNLoS='+etaNlos+' · k='+K+' · n='+n+' · runs='+R;
  document.getElementById('pso-empty').classList.add('hidden');

  setTimeout(function(){
    psoLogs = [];
    psoIterAvg = Array(K).fill(0);

    PR_DB.forEach(function(mobile, idx){
      document.getElementById('pso-progress').textContent =
        'คำนวณ '+(idx+1)+'/'+PR_DB.length+'... ['+mobile.id+']';

      // ── HYBRID: KNN ทำนายชั้นก่อน → PSO เฉพาะชั้นนั้น ──
      // หา KNN floor prediction จาก mRSSI
      var knnFloorM = knnPredictFloor(mobile.mRSSI, mobile.id);
      // หา KNN floor prediction จาก pRSSI
      var knnFloorR = knnPredictFloor(mobile.pRSSI, mobile.id);

      // รัน PSO เฉพาะใน floor ที่ KNN ทำนาย
      var rM = psoRunHybrid(mobile.mRSSI, mobile, knnFloorM, n, K, wmin, wmax, c1, c2, R, etaLos, etaNlos);
      var rR = psoRunHybrid(mobile.pRSSI, mobile, knnFloorR, n, K, wmin, wmax, c1, c2, R, etaLos, etaNlos);

      rR.iterFitness.forEach(function(v,i){ psoIterAvg[i] += v; });

      // แยก error ระหว่าง Meas. และ RSSI Pred.
      var xyErrM = Math.sqrt((rM.gx-mobile.x)**2 + (rM.gy-mobile.y)**2);
      var xyErrR = Math.sqrt((rR.gx-mobile.x)**2 + (rR.gy-mobile.y)**2);

      psoLogs.push({
        floor:mobile.floor, id:mobile.id,
        px:mobile.x, py:mobile.y,
        knnFloorM:knnFloorM, knnFloorR:knnFloorR,
        predFloorM:rM.predFloor, gxM:rM.gx, gyM:rM.gy, costM:rM.gcost, xyErrM:xyErrM,
        predFloorR:rR.predFloor, gxR:rR.gx, gyR:rR.gy, costR:rR.gcost, xyErrR:xyErrR
      });
    });

    psoIterAvg = psoIterAvg.map(function(v){ return +(v/PR_DB.length).toFixed(3); });
    renderPSOResults();
    setStatus('done','Hybrid KNN→PSO เสร็จ — 55 Particles');
    document.getElementById('pso-progress').textContent = 'เสร็จสิ้น ✓';
    document.getElementById('btn-export-pso').style.display = '';
  }, 50);
}

// ── KNN Floor Prediction (ใช้ข้อมูล PR_DB เป็น reference) ──
function knnPredictFloor(mobileRSSI, mobileId){
  var floorDist = {};
  FLOORS.forEach(function(fl){ floorDist[fl] = []; });

  PR_DB.forEach(function(pr){
    if(pr.id === mobileId) return; // leave-one-out
    var d = euclid3RSSI(mobileRSSI, pr.pRSSI);
    if(floorDist[pr.floor]) floorDist[pr.floor].push(d);
  });

  // Top-3 ต่อชั้น (วิธีที่แม่นที่สุดจาก KNN ของเรา)
  var floorScores = {};
  FLOORS.forEach(function(fl){
    var dList = floorDist[fl].slice().sort(function(a,b){return a-b;});
    var top3 = dList.filter(function(v){return v>0;}).slice(0,3);
    floorScores[fl] = top3.length>0 ? top3.reduce(function(a,b){return a+b;},0)/top3.length : 999;
  });
  return FLOORS.reduce(function(a,b){ return floorScores[a]<floorScores[b]?a:b; });
}

function euclid3RSSI(a, b){
  return Math.sqrt((a[0]-b[0])**2+(a[1]-b[1])**2+(a[2]-b[2])**2);
}

// ── PSO แบบ Hybrid — รันเฉพาะชั้นที่ KNN บอก ──
function psoRunHybrid(mobileRSSI, mobile, targetFloor, n, K, wmin, wmax, c1, c2, R, etaLos, etaNlos){
  var results = [];
  for(var r=0; r<R; r++){
    results.push(psoRunOneFloor(mobileRSSI, mobile, targetFloor, n, K, wmin, wmax, c1, c2, etaLos, etaNlos));
  }
  // เฉลี่ย XY จากทุก run
  var avgX = results.reduce(function(s,r){return s+r.gx;},0)/results.length;
  var avgY = results.reduce(function(s,r){return s+r.gy;},0)/results.length;
  var best = results.reduce(function(a,b){ return a.gcost<b.gcost?a:b; });
  return {
    gx:+(avgX.toFixed(2)), gy:+(avgY.toFixed(2)),
    gcost:+(best.gcost.toFixed(3)), predFloor:targetFloor,
    iterFitness:best.iterFitness
  };
}

// ── FLOOR / ENVIRONMENT CONFIG ──
// AN ติด ceiling ชั้น 4 สูง 1.5m จากพื้น F4
// F4 พื้นอยู่สูง 9m จาก F1, ดังนั้น AN อยู่สูง 10.5m จาก F1
// z = ระยะ vertical จาก AN ลงถึง PR (ใช้ใน d 3D)
var AN_HEIGHT    = 10.5; // m จากพื้น F1
var FLOOR_HEIGHT = {'Floor 4':9.0,'Floor 3':6.0,'Floor 2':3.0,'Floor 1':0.0};
function getZ(fl){ return AN_HEIGHT - (FLOOR_HEIGHT[fl]||0); }
// F4→z=1.5m, F3→z=4.5m, F2→z=7.5m, F1→z=10.5m

var FAF_PER_FLOOR   = 12.0; // dBm/ชั้น คอนกรีต LoRa 433MHz
var PLd0_CALIBRATED = -53.0;
var OPEN_ATRIUM_IDS = new Set(['PR40','PR41','PR42','PR43','PR44','PR45','PR46','PR47','PR55']);

function floorsThrough(prId, targetFloor){
  if(OPEN_ATRIUM_IDS.has(prId)) return 0;
  var fn = parseInt(targetFloor.replace('Floor ',''));
  return Math.max(0, 4-fn);
}

function dist3D(px, py, pz, ax, ay){
  return Math.max(Math.sqrt((px-ax)**2+(py-ay)**2+pz**2), 0.1);
}

function fitness(p, mobileRSSI, faf){
  var pz=p.z||0, fafT=faf||0, total=0;
  ANCHORS.forEach(function(an,i){
    var d = dist3D(p.x,p.y,pz,an.x,an.y);
    var predicted = PLd0_CALIBRATED - 10*p.eta*Math.log10(d) - fafT;
    predicted = Math.max(-120, Math.min(-20, predicted));
    total += (predicted - mobileRSSI[i])**2;
  });
  return Math.sqrt(total/ANCHORS.length);
}

function backsolveEta(){
  var f4prs = PR_DB.filter(function(p){return p.floor==='Floor 4';});
  var pz4   = getZ('Floor 4'); // 1.5m
  var losVals=[], nlosVals=[];
  f4prs.forEach(function(pr){
    ANCHORS.forEach(function(an,i){
      var d=dist3D(pr.x,pr.y,pz4,an.x,an.y);
      var rssi=pr.mRSSI[i];
      if(rssi<=-110||d<0.3) return;
      var eta=(PLd0_CALIBRATED-rssi)/(10*Math.log10(d));
      if(eta<1.0||eta>6.0) return;
      if(OPEN_ATRIUM_IDS.has(pr.id)) losVals.push(eta);
      else nlosVals.push(eta);
    });
  });
  function median(arr){
    if(!arr.length) return null;
    var s=arr.slice().sort(function(a,b){return a-b;});
    return s[Math.floor(s.length/2)];
  }
  var losEta  = +Math.max(1.8,Math.min(3.5, median(losVals)||2.2)).toFixed(2);
  var nlosEta = +Math.max(3.0,Math.min(5.5, median(nlosVals)||3.8)).toFixed(2);
  return {los:losEta, nlos:nlosEta,
    detail:'LoS('+losVals.length+')='+losEta+' NLoS('+nlosVals.length+')='+nlosEta};
}

function knnPredictFloor(mobileRSSI, mobileId){
  var flDist={};
  FLOORS.forEach(function(fl){flDist[fl]=[];});
  PR_DB.forEach(function(pr){
    if(pr.id===mobileId) return;
    var d=Math.sqrt(
      (mobileRSSI[0]-pr.pRSSI[0])**2+
      (mobileRSSI[1]-pr.pRSSI[1])**2+
      (mobileRSSI[2]-pr.pRSSI[2])**2
    );
    if(flDist[pr.floor]) flDist[pr.floor].push(d);
  });
  var scores={};
  FLOORS.forEach(function(fl){
    var s=flDist[fl].slice().sort(function(a,b){return a-b;});
    var top=s.filter(function(v){return v>0;}).slice(0,3);
    scores[fl]=top.length?top.reduce(function(a,b){return a+b;},0)/top.length:999;
  });
  return FLOORS.reduce(function(a,b){return scores[a]<scores[b]?a:b;});
}

function euclid3RSSI(a,b){
  return Math.sqrt((a[0]-b[0])**2+(a[1]-b[1])**2+(a[2]-b[2])**2);
}

// PSO + Local grid search refinement
function psoRunHybrid(mobileRSSI, mobile, targetFloor, n, K, wmin, wmax, c1, c2, R, etaLos, etaNlos){
  var runs=[];
  for(var r=0;r<R;r++){
    runs.push(psoRunOneFloor(mobileRSSI,mobile,targetFloor,n,K,wmin,wmax,c1,c2,etaLos,etaNlos));
  }
  var avgX=runs.reduce(function(s,r){return s+r.gx;},0)/runs.length;
  var avgY=runs.reduce(function(s,r){return s+r.gy;},0)/runs.length;
  var best=runs.reduce(function(a,b){return a.gcost<b.gcost?a:b;});
  // Local grid refinement ±2m step 0.2m
  var pz=getZ(targetFloor);
  var nF=floorsThrough(mobile.id,targetFloor);
  var faf=nF*FAF_PER_FLOOR;
  var etaMid=OPEN_ATRIUM_IDS.has(mobile.id)?etaLos:etaNlos;
  var bestL={x:avgX,y:avgY,c:best.gcost};
  for(var dx=-2;dx<=2;dx+=0.2){
    for(var dy=-2;dy<=2;dy+=0.2){
      var tx=Math.max(0,Math.min(36,avgX+dx));
      var ty=Math.max(0,Math.min(27,avgY+dy));
      var p={x:tx,y:ty,z:pz,eta:etaMid};
      var c=fitness(p,mobileRSSI,faf);
      if(c<bestL.c) bestL={x:tx,y:ty,c:c};
    }
  }
  return{
    gx:+(bestL.x.toFixed(2)),gy:+(bestL.y.toFixed(2)),
    gcost:+(bestL.c.toFixed(3)),predFloor:targetFloor,
    iterFitness:best.iterFitness
  };
}

function psoRunOneFloor(mobileRSSI, mobile, fl, n, K, wmin, wmax, c1, c2, etaLos, etaNlos){
  var scatterX=36, scatterY=27;
  var pz=getZ(fl);
  var nF=floorsThrough(mobile.id,fl);
  var faf=nF*FAF_PER_FLOOR;
  var isLos=OPEN_ATRIUM_IDS.has(mobile.id);
  var etaMid=isLos?etaLos:etaNlos;
  var etaLo=Math.max(1.0,etaMid-0.5);
  var etaHi=Math.min(6.0,etaMid+0.5);
  var flPRs=PR_DB.filter(function(p){return p.floor===fl;});
  var particles=[];
  flPRs.forEach(function(s){
    particles.push({x:s.x+rand(-1.5,1.5),y:s.y+rand(-1.5,1.5),z:pz,
      eta:rand(etaLo,etaHi),vx:0,vy:0,bx:s.x,by:s.y,bc:Infinity});
  });
  while(particles.length<n){
    particles.push({x:rand(0,scatterX),y:rand(0,scatterY),z:pz,
      eta:rand(etaLo,etaHi),vx:0,vy:0,
      bx:rand(0,scatterX),by:rand(0,scatterY),bc:Infinity});
  }
  particles.forEach(function(p){p.bc=fitness(p,mobileRSSI,faf);p.bx=p.x;p.by=p.y;});
  var gbest=getBest(particles);
  var iterFitness=[];
  for(var k=1;k<=K;k++){
    var w=wmax-k*(wmax-wmin)/K;
    particles.forEach(function(p){
      var r1=Math.random(),r2=Math.random();
      p.vx=w*p.vx+c1*r1*(p.bx-p.x)+c2*r2*(gbest.x-p.x);
      p.vy=w*p.vy+c1*r1*(p.by-p.y)+c2*r2*(gbest.y-p.y);
      p.vx=Math.max(-4,Math.min(4,p.vx));
      p.vy=Math.max(-4,Math.min(4,p.vy));
      p.x=Math.max(0,Math.min(scatterX,p.x+p.vx));
      p.y=Math.max(0,Math.min(scatterY,p.y+p.vy));
      var cost=fitness(p,mobileRSSI,faf);
      if(cost<p.bc){p.bc=cost;p.bx=p.x;p.by=p.y;}
      if(cost<gbest.bc){gbest={x:p.x,y:p.y,bc:cost};}
    });
    iterFitness.push(gbest.bc);
  }
  return{gx:gbest.x,gy:gbest.y,gcost:gbest.bc,iterFitness:iterFitness};
}

function getBest(particles){
  return particles.reduce(function(b,p){
    return p.bc<b.bc?{x:p.x,y:p.y,z:p.z||0,bc:p.bc}:b;
  },{x:0,y:0,z:0,bc:Infinity});
}
function rand(a,b){return a+Math.random()*(b-a);}


// ── BACKSOLVE η ──
document.getElementById('btn-backsolve').addEventListener('click', function(){
  var result = backsolveEta();
  document.getElementById('eta-los').value  = result.los;
  document.getElementById('eta-nlos').value = result.nlos;
  document.getElementById('tune-status').style.display = 'block';
  document.getElementById('tune-status').textContent =
    'Backsolve จาก PR F4 ทั้ง '+PR_DB.filter(function(p){return p.floor==='Floor 4';}).length+' ตัว → ηLoS='+result.los+' · ηNLoS='+result.nlos+' · ηMed='+result.med;
});

// ── AUTO-TUNE ──
document.getElementById('btn-auto-tune').addEventListener('click',function(){
  setStatus('running','Auto-Tuning...');
  document.getElementById('tune-status').style.display='block';
  document.getElementById('tune-log').style.display='none';
  document.getElementById('tune-status').textContent='กำลังทดสอบ combinations...';
  setTimeout(function(){
    var solved = backsolveEta();
    var etaL = solved.los, etaN = solved.nlos;

    var candidates=[
      {wmin:0.4,wmax:0.9,c1:2.0,c2:2.0},
      {wmin:0.4,wmax:0.9,c1:1.5,c2:1.5},
      {wmin:0.3,wmax:0.9,c1:2.0,c2:2.0},
      {wmin:0.2,wmax:0.8,c1:2.0,c2:2.0},
      {wmin:0.4,wmax:0.8,c1:1.5,c2:2.0},
      {wmin:0.5,wmax:0.9,c1:2.0,c2:1.5},
      {wmin:0.3,wmax:0.7,c1:1.8,c2:1.8},
      {wmin:0.4,wmax:0.9,c1:2.5,c2:2.5},
      {wmin:0.2,wmax:0.9,c1:1.5,c2:2.5},
      {wmin:0.5,wmax:0.8,c1:2.0,c2:2.0},
    ];
    var bestParams=null, bestAvgErr=Infinity;
    var results=[];
    var n  = parseInt(document.getElementById('pso-n').value) || 100;
    var K  = parseInt(document.getElementById('pso-k').value) || 30;
    var R  = Math.min(parseInt(document.getElementById('pso-runs').value)||5, 3); // cap at 3 for speed

    // sample 1 ใน 5 PR เพื่อความเร็ว
    var sample = PR_DB.filter(function(_,i){return i%5===0;});

    candidates.forEach(function(c){
      var totalErr=0;
      sample.forEach(function(mob){
        var knnFloor = knnPredictFloor(mob.pRSSI, mob.id);
        var r = psoRunHybrid(mob.pRSSI, mob, knnFloor, n, K, c.wmin, c.wmax, c.c1, c.c2, R, etaL, etaN);
        totalErr += Math.sqrt((r.gx-mob.x)**2+(r.gy-mob.y)**2);
      });
      var avgE = totalErr/sample.length;
      results.push({params:c, err:avgE});
      if(avgE < bestAvgErr){ bestAvgErr=avgE; bestParams=c; }
    });

    document.getElementById('pso-wmin').value = bestParams.wmin;
    document.getElementById('pso-wmax').value = bestParams.wmax;
    document.getElementById('pso-c1').value   = bestParams.c1;
    document.getElementById('pso-c2').value   = bestParams.c2;
    document.getElementById('eta-los').value  = etaL;
    document.getElementById('eta-nlos').value = etaN;

    var rank = results.slice().sort(function(a,b){return a.err-b.err;});
    var report = 'PLd0=-53 (calibrated) · ηLoS='+etaL+' · ηNLoS='+etaN+'\n\n';
    report += rank.slice(0,5).map(function(r,i){
      return (i+1)+'. w=['+r.params.wmin+','+r.params.wmax+'] c1='+r.params.c1+' c2='+r.params.c2+' → avgErr='+r.err.toFixed(2)+'m';
    }).join('\n');

    document.getElementById('tune-status').textContent =
      'เสร็จ ✓ เลือก w=['+bestParams.wmin+','+bestParams.wmax+'] c1='+bestParams.c1+' c2='+bestParams.c2+' (avgErr='+bestAvgErr.toFixed(2)+'m)';
    document.getElementById('tune-log').style.display='block';
    document.getElementById('tune-log').textContent=report;
    setStatus('done','Auto-Tune เสร็จสิ้น');
  },50);
});

function renderPSOResults(){
  var t = psoLogs.length;
  var okM  = psoLogs.filter(function(l){return l.predFloorM===l.floor;}).length;
  var okR  = psoLogs.filter(function(l){return l.predFloorR===l.floor;}).length;
  var avgErrM = (psoLogs.reduce(function(s,l){return s+l.xyErrM;},0)/t).toFixed(2);
  var avgErrR = (psoLogs.reduce(function(s,l){return s+l.xyErrR;},0)/t).toFixed(2);
  // error <4m count
  var lt4M = psoLogs.filter(function(l){return l.xyErrM<4;}).length;
  var lt4R = psoLogs.filter(function(l){return l.xyErrR<4;}).length;

  document.getElementById('pso-acc-m').textContent    = ((okM/t)*100).toFixed(1)+'%';
  document.getElementById('pso-acc-r').textContent    = ((okR/t)*100).toFixed(1)+'%';
  document.getElementById('pso-avg-err-m').textContent = avgErrM+' m';
  document.getElementById('pso-avg-err-r').textContent = avgErrR+' m';
  document.getElementById('pso-lt4-m').textContent   = lt4M+'/'+t+' PR';
  document.getElementById('pso-lt4-r').textContent   = lt4R+'/'+t+' PR';
  document.getElementById('pso-count').textContent    = t+' Particles';

  // Converge chart
  mkChart('psoConvergeChart',{type:'line',data:{labels:psoIterAvg.map(function(_,i){return 'Iter '+(i+1);}),datasets:[
    {label:'Avg Fitness',data:psoIterAvg,borderColor:'#a855f7',backgroundColor:'rgba(168,85,247,0.08)',pointBackgroundColor:'#a855f7',pointRadius:4,fill:true,tension:.35,borderWidth:2}
  ]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false}},scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C}},x:{grid:{display:false},ticks:{color:TICK_C}}}}});

  // PSO Table
  var fp={'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
  var tbody=document.getElementById('pso-table-body'); tbody.innerHTML='';
  psoLogs.forEach(function(log){
    var fn=log.floor.replace('Floor ','');
    var ecM=log.xyErrM<2?'var(--ok)':log.xyErrM<4?'var(--amber)':'var(--err)';
    var ecR=log.xyErrR<2?'var(--ok)':log.xyErrR<4?'var(--amber)':'var(--err)';
    var okFM = log.predFloorM===log.floor;
    var okFR = log.predFloorR===log.floor;
    tbody.innerHTML+=
      '<tr>'+
      '<td><span class="floor-pill '+(fp[fn]||'')+'">'+log.floor+'</span></td>'+
      '<td style="color:var(--muted)">'+log.id+'</td>'+
      '<td style="color:var(--muted);font-size:10px">'+log.knnFloorM+'</td>'+
      '<td>'+(okFM?'<span class="badge badge-ok">'+log.predFloorM+'</span>':'<span class="badge badge-err">'+log.predFloorM+'</span>')+'</td>'+
      '<td style="color:var(--muted)">'+log.gxM+', '+log.gyM+'</td>'+
      '<td style="color:'+ecM+';font-weight:600">'+log.xyErrM.toFixed(2)+'</td>'+
      '<td style="color:var(--muted);font-size:10px">'+log.knnFloorR+'</td>'+
      '<td>'+(okFR?'<span class="badge badge-ok">'+log.predFloorR+'</span>':'<span class="badge badge-err">'+log.predFloorR+'</span>')+'</td>'+
      '<td style="color:var(--muted)">'+log.gxR+', '+log.gyR+'</td>'+
      '<td style="color:'+ecR+';font-weight:600">'+log.xyErrR.toFixed(2)+'</td>'+
      '</tr>';
  });

  document.getElementById('pso-results-section').classList.remove('hidden');
  renderMaps();
}

// ── FLOOR MAPS ──
var mapColors={
  'Floor 1':'#f59e0b','Floor 2':'#3b82f6','Floor 3':'#10b981','Floor 4':'#a855f7'
};
var floorCanvases={'Floor 1':'map-f1','Floor 2':'map-f2','Floor 3':'map-f3','Floor 4':'map-f4'};

function renderMaps(){
  FLOORS.forEach(function(fl){
    var cid=floorCanvases[fl];
    var el=document.getElementById(cid);
    if(!el)return;
    var wrap=el.parentElement;
    var W=wrap.clientWidth||360;
    var H=W*(28/36);
    el.width=W*2; el.height=H*2;
    el.style.width=W+'px'; el.style.height=H+'px';
    var ctx=el.getContext('2d');
    ctx.scale(2,2);

    var PAD=24;
    var drawW=W-PAD*2, drawH=H-PAD*2;
    var mapX=36, mapY=27;

    function cx(rx){return PAD+rx/mapX*drawW;}
    function cy(ry){return PAD+(mapY-ry)/mapY*drawH;}

    ctx.fillStyle='#0d1220';
    ctx.fillRect(0,0,W,H);
    ctx.strokeStyle='rgba(255,255,255,0.08)';
    ctx.lineWidth=0.5;
    ctx.strokeRect(PAD,PAD,drawW,drawH);

    var flPRs=PR_DB.filter(function(p){return p.floor===fl;});
    var isF4 = fl==='Floor 4';

    // ── Floor 4 only: draw triangle lines from each AN to the other 2 ANs ──
    if(isF4){
      ctx.strokeStyle='rgba(168,85,247,0.35)';
      ctx.lineWidth=0.8;
      ctx.setLineDash([4,3]);
      // Triangle: AN1-AN2, AN2-AN3, AN3-AN1
      [[0,1],[1,2],[2,0]].forEach(function(pair){
        ctx.beginPath();
        ctx.moveTo(cx(ANCHORS[pair[0]].x), cy(ANCHORS[pair[0]].y));
        ctx.lineTo(cx(ANCHORS[pair[1]].x), cy(ANCHORS[pair[1]].y));
        ctx.stroke();
      });
      ctx.setLineDash([]);
    }

    // ── Floor 4: draw lines from each AN to all PRs on this floor ──
    if(isF4){
      flPRs.forEach(function(pr){
        ANCHORS.forEach(function(an){
          ctx.beginPath();
          ctx.moveTo(cx(an.x), cy(an.y));
          ctx.lineTo(cx(pr.x), cy(pr.y));
          ctx.strokeStyle='rgba(168,85,247,0.08)';
          ctx.lineWidth=0.5;
          ctx.stroke();
        });
      });
    }

    // ── Draw PR points ──
    flPRs.forEach(function(pr){
      var pcx=cx(pr.x), pcy=cy(pr.y);
      var knnLog=knnLogs.find(function(l){return l.id===pr.id;});
      var psoLog=psoLogs.find(function(l){return l.id===pr.id;});
      var dotColor='rgba(255,255,255,0.25)';

      if(knnLog){
        var anyWrong=[knnLog.mTop3,knnLog.mAvg,knnLog.rTop3,knnLog.rAvg].some(function(v){return v!=='ถูก';});
        dotColor=anyWrong?'#ef4444':'#22c55e';
      }

      // Open atrium PR gets a different border
      var isOpen=OPEN_ATRIUM_IDS.has(pr.id);
      ctx.beginPath();
      ctx.arc(pcx,pcy,isOpen?5:4,0,Math.PI*2);
      ctx.fillStyle=dotColor;
      ctx.fill();
      if(isOpen){
        ctx.strokeStyle='#f59e0b';
        ctx.lineWidth=1;
        ctx.stroke();
      }

      ctx.fillStyle='rgba(255,255,255,0.75)';
      ctx.font='7px IBM Plex Mono,monospace';
      ctx.textAlign='center';
      ctx.fillText(pr.id,pcx,pcy-8);

      // PSO gbest star
      if(psoLog){
        var sx=cx(psoLog.gxR), sy=cy(psoLog.gyR);
        ctx.beginPath();
        ctx.arc(sx,sy,3,0,Math.PI*2);
        ctx.fillStyle='#facc15';
        ctx.fill();
      }
    });

    // ── Draw Anchor Nodes — triangle shape, Floor 4 ONLY ──
    if(isF4){
      ANCHORS.forEach(function(an){
        var acx=cx(an.x), acy=cy(an.y);
        var s=7; // half-size
        ctx.beginPath();
        ctx.moveTo(acx, acy-s);
        ctx.lineTo(acx+s, acy+s);
        ctx.lineTo(acx-s, acy+s);
        ctx.closePath();
        ctx.fillStyle='#a855f7';
        ctx.fill();
        ctx.strokeStyle='#d8b4fe';
        ctx.lineWidth=0.8;
        ctx.stroke();
        ctx.fillStyle='rgba(255,255,255,0.9)';
        ctx.font='bold 7px IBM Plex Mono,monospace';
        ctx.textAlign='center';
        ctx.fillText(an.id,acx,acy-s-3);
      });
    }

    // Floor label
    ctx.fillStyle=mapColors[fl];
    ctx.font='bold 9px Noto Sans Thai,sans-serif';
    ctx.textAlign='left';
    ctx.fillText(fl+' ('+flPRs.length+' PR)',PAD+2,PAD-6);
  });
}

// ── CHART HELPER ──
function mkChart(id,cfg){if(charts[id])charts[id].destroy();charts[id]=new Chart(document.getElementById(id).getContext('2d'),cfg);}

// ── EXPORT KNN ──
document.getElementById('btn-export').addEventListener('click',function(){
  if(!knnLogs.length){alert('ไม่พบข้อมูล KNN');return;}
  var rows=knnLogs.map(function(l){return{'Floor จริง':l.floor,'Particle ID':l.id,'Measurement (TOP 3)':l.mTop3,'Measurement (AVG รวม)':l.mAvg,'RSSI Predict (TOP 3)':l.rTop3,'RSSI Predict (AVG รวม)':l.rAvg,'คลาดเคลื่อน Meas.Top3':l.gMT3,'คลาดเคลื่อน Meas.Avg':l.gMA,'คลาดเคลื่อน RSSI.Top3':l.gRT3,'คลาดเคลื่อน RSSI.Avg':l.gRA};});
  var ws1=XLSX.utils.json_to_sheet(rows);
  ws1['!cols']=[14,12,22,22,22,22,20,20,20,20].map(function(w){return{wch:w};});

  var t=knnLogs.length;
  var ok=function(k){return knnLogs.filter(function(l){return l[k]==='ถูก';}).length;};
  var sum=[{วิธี:'Measurement (TOP 3)',ถูก:ok('mTop3'),ผิด:t-ok('mTop3'),Total:t,'%':((ok('mTop3')/t)*100).toFixed(2)+'%'},{วิธี:'Measurement (AVG)',ถูก:ok('mAvg'),ผิด:t-ok('mAvg'),Total:t,'%':((ok('mAvg')/t)*100).toFixed(2)+'%'},{วิธี:'RSSI Predict (TOP 3)',ถูก:ok('rTop3'),ผิด:t-ok('rTop3'),Total:t,'%':((ok('rTop3')/t)*100).toFixed(2)+'%'},{วิธี:'RSSI Predict (AVG)',ถูก:ok('rAvg'),ผิด:t-ok('rAvg'),Total:t,'%':((ok('rAvg')/t)*100).toFixed(2)+'%'}];
  var ws2=XLSX.utils.json_to_sheet(sum);
  ws2['!cols']=[26,8,8,8,10].map(function(w){return{wch:w};});

  var wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws1,'ผลการคำนวณ KNN');
  XLSX.utils.book_append_sheet(wb,ws2,'สรุปความถูกต้อง');
  XLSX.writeFile(wb,'PSOCalc_KNN_Report.xlsx');
});

// ── EXPORT PSO ──
document.getElementById('btn-export-pso').addEventListener('click',function(){
  if(!psoLogs.length){alert('กรุณารัน PSO ก่อน');return;}
  var rows=psoLogs.map(function(l){return{
    'Floor จริง':l.floor,'Particle ID':l.id,'X จริง':l.px,'Y จริง':l.py,
    // Measurement
    'KNN Floor (Meas.)':l.knnFloorM,
    'PSO ทำนาย (Meas.)':l.predFloorM,
    'gbest X (Meas.)':l.gxM,'gbest Y (Meas.)':l.gyM,
    'XY Error Meas. (m)':l.xyErrM.toFixed(3),
    // RSSI Predict
    'KNN Floor (RSSI)':l.knnFloorR,
    'PSO ทำนาย (RSSI)':l.predFloorR,
    'gbest X (RSSI)':l.gxR,'gbest Y (RSSI)':l.gyR,
    'XY Error RSSI (m)':l.xyErrR.toFixed(3),
  };});
  var ws=XLSX.utils.json_to_sheet(rows);
  ws['!cols']=[14,12,10,10,16,16,12,12,14,16,16,12,12,14].map(function(w){return{wch:w};});
  var wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,'Hybrid KNN-PSO Results');
  XLSX.writeFile(wb,'PSOCalc_HybridKNN_PSO_Results.xlsx');
});

// Init map on page load
setTimeout(renderMaps,200);

// ══════════════════════════════════════════════
// PSO ANIMATOR ENGINE
// ══════════════════════════════════════════════
(function(){
  var animState = {
    mobile: null, rssiMode: 'meas',
    particles: [], gbest: null,
    history: [],   // [{particles snapshot, gbest, cost, err}]
    currentK: 0, totalK: 0,
    timer: null, running: false,
    convChart: null
  };

  // Populate mobile selector
  var sel = document.getElementById('anim-mobile-select');
  PR_DB.forEach(function(pr){
    var opt = document.createElement('option');
    opt.value = pr.id;
    opt.textContent = pr.id + ' — ' + pr.floor + ' (X'+pr.x+', Y'+pr.y+')';
    sel.appendChild(opt);
  });

  sel.addEventListener('change', function(){
    resetAnim();
    var pr = PR_DB.find(function(p){return p.id===sel.value;});
    if(!pr) return;
    animState.mobile = pr;
    var isLos = OPEN_ATRIUM_IDS.has(pr.id);
    document.getElementById('anim-mobile-info').style.display='block';
    document.getElementById('anim-mobile-info').innerHTML =
      '<b style="color:var(--purple)">'+pr.id+'</b> · '+pr.floor+
      '<br>📍 X='+pr.x+' m, Y='+pr.y+' m'+
      '<br>📡 AN1: '+pr.mRSSI[0]+' dBm · AN2: '+pr.mRSSI[1]+' dBm · AN3: '+pr.mRSSI[2]+' dBm'+
      '<br>🏗️ Zone: '+(isLos?'<span style="color:#f59e0b">โถงเปิด (LoS)</span>':'<span style="color:#3b82f6">ห้องเรียน (NLoS)</span>');
    document.getElementById('anim-floor-label').textContent = 'ชั้น: '+pr.floor+' | '+pr.id;
    document.getElementById('anim-btn-init').disabled = false;
  });

  document.querySelectorAll('input[name="anim-rssi-mode"]').forEach(function(r){
    r.addEventListener('change',function(){ animState.rssiMode=r.value; resetAnim(); });
  });

  document.getElementById('anim-btn-init').addEventListener('click', initAnim);
  document.getElementById('anim-btn-play').addEventListener('click', playAnim);
  document.getElementById('anim-btn-pause').addEventListener('click', pauseAnim);
  document.getElementById('anim-btn-reset').addEventListener('click', resetAnim);
  document.getElementById('anim-btn-prev').addEventListener('click', function(){ stepAnim(-1); });
  document.getElementById('anim-btn-next').addEventListener('click', function(){ stepAnim(1); });

  function getMobileRSSI(){
    var pr = animState.mobile;
    return animState.rssiMode==='meas' ? pr.mRSSI : pr.pRSSI;
  }

  function initAnim(){
    if(!animState.mobile) return;
    resetAnim();
    var mobile  = animState.mobile;
    var mRSSI   = getMobileRSSI();
    var n       = parseInt(document.getElementById('anim-n').value)||30;
    var K       = parseInt(document.getElementById('anim-k').value)||20;
    var etaInput= parseFloat(document.getElementById('anim-eta').value)||2.2;
    var fl      = knnPredictFloor(mRSSI, mobile.id);
    var pz      = getZ(fl);
    var nF      = floorsThrough(mobile.id, fl);
    var faf     = nF * FAF_PER_FLOOR;
    var isLos   = OPEN_ATRIUM_IDS.has(mobile.id);
    var eta     = etaInput;
    var scX=36, scY=27;

    // Build initial particles: seed from floor PRs + random fill
    var flPRs = PR_DB.filter(function(p){return p.floor===fl;});
    var particles = [];
    flPRs.forEach(function(s){
      particles.push({x:s.x+rand(-2,2),y:s.y+rand(-2,2),z:pz,
        eta:eta+rand(-0.3,0.3),vx:0,vy:0,bx:s.x,by:s.y,bc:Infinity});
    });
    while(particles.length<n){
      particles.push({x:rand(0,scX),y:rand(0,scY),z:pz,
        eta:eta+rand(-0.3,0.3),vx:0,vy:0,
        bx:rand(0,scX),by:rand(0,scY),bc:Infinity});
    }

    // Init fitness
    particles.forEach(function(p){
      p.bc=fitness(p,mRSSI,faf); p.bx=p.x; p.by=p.y;
    });
    var gbest=getBest(particles);
    animState.particles = particles;
    animState.gbest = gbest;
    animState.history = [];
    animState.totalK = K;
    animState.currentK = 0;
    animState.faf = faf;
    animState.mRSSI = mRSSI;
    animState.floor = fl;
    animState.K = K;
    animState.wmin = 0.4; animState.wmax = 0.9;
    animState.c1 = 2.0;   animState.c2 = 2.0;

    // Record k=0
    recordSnapshot(0);
    drawAnimCanvas();
    updateIterInfo(0);
    updateConvChart();

    document.getElementById('anim-btn-play').disabled = false;
    document.getElementById('anim-btn-reset').disabled = false;
    document.getElementById('anim-btn-next').disabled = false;
    document.getElementById('anim-btn-prev').disabled = false;
    document.getElementById('anim-iter-total').textContent = K;
    document.getElementById('anim-floor-label').textContent =
      'KNN → '+fl+' | '+mobile.id+' | '+animState.rssiMode.toUpperCase();
  }

  function recordSnapshot(k){
    var mobile = animState.mobile;
    var gbest  = animState.gbest;
    var err = Math.sqrt((gbest.x-mobile.x)**2+(gbest.y-mobile.y)**2);
    animState.history.push({
      k:k,
      particles: animState.particles.map(function(p){return{x:p.x,y:p.y,bc:p.bc};}),
      gbest:{x:gbest.x,y:gbest.y,bc:gbest.bc},
      err:err
    });
  }

  function doOneIteration(){
    if(animState.currentK >= animState.totalK){ pauseAnim(); return; }
    var k = animState.currentK + 1;
    var w = animState.wmax - k*(animState.wmax-animState.wmin)/animState.totalK;
    var scX=36, scY=27;

    animState.particles.forEach(function(p){
      var r1=Math.random(), r2=Math.random();
      p.vx=w*p.vx+animState.c1*r1*(p.bx-p.x)+animState.c2*r2*(animState.gbest.x-p.x);
      p.vy=w*p.vy+animState.c1*r1*(p.by-p.y)+animState.c2*r2*(animState.gbest.y-p.y);
      p.vx=Math.max(-4,Math.min(4,p.vx));
      p.vy=Math.max(-4,Math.min(4,p.vy));
      p.x=Math.max(0,Math.min(scX,p.x+p.vx));
      p.y=Math.max(0,Math.min(scY,p.y+p.vy));
      var cost=fitness(p,animState.mRSSI,animState.faf);
      if(cost<p.bc){p.bc=cost;p.bx=p.x;p.by=p.y;}
      if(cost<animState.gbest.bc){animState.gbest={x:p.x,y:p.y,bc:cost};}
    });

    animState.currentK = k;
    recordSnapshot(k);
    drawAnimCanvas();
    updateIterInfo(k);
    updateConvChart();
  }

  function playAnim(){
    if(animState.running) return;
    animState.running = true;
    var speed = parseInt(document.getElementById('anim-speed').value)||300;
    animState.timer = setInterval(function(){
      if(animState.currentK >= animState.totalK){ pauseAnim(); return; }
      doOneIteration();
    }, speed);
  }

  function pauseAnim(){
    animState.running = false;
    if(animState.timer){ clearInterval(animState.timer); animState.timer=null; }
  }

  function resetAnim(){
    pauseAnim();
    animState.history = [];
    animState.currentK = 0;
    animState.particles = [];
    animState.gbest = null;
    document.getElementById('anim-btn-play').disabled = true;
    document.getElementById('anim-btn-reset').disabled = true;
    document.getElementById('anim-btn-next').disabled = true;
    document.getElementById('anim-btn-prev').disabled = true;
    document.getElementById('anim-iter-num').textContent='—';
    document.getElementById('anim-iter-total').textContent='—';
    document.getElementById('anim-gbest-cost').textContent='—';
    document.getElementById('anim-gbest-xy').textContent='—';
    document.getElementById('anim-gbest-err').textContent='—';
    document.getElementById('anim-iter-log').innerHTML='';
    clearAnimCanvas();
  }

  function stepAnim(dir){
    pauseAnim();
    if(dir>0 && animState.currentK < animState.totalK) doOneIteration();
    else if(dir<0 && animState.currentK>0){
      // replay from history
      var targetK = animState.currentK - 1;
      var snap = animState.history[targetK];
      if(snap){
        animState.currentK = snap.k;
        animState.gbest = {x:snap.gbest.x,y:snap.gbest.y,bc:snap.gbest.bc};
        drawFromSnapshot(snap);
        updateIterInfo(snap.k);
      }
    }
  }

  // ── DRAW ──
  function getAnimCanvas(){
    var el   = document.getElementById('anim-canvas');
    var wrap = el.parentElement;
    var W    = wrap.offsetWidth  || 560;
    var H    = wrap.offsetHeight || Math.round(W * 0.75);
    if(H < 10) H = Math.round(W * 0.75);
    var dpr = window.devicePixelRatio || 1;
    if(el.dataset.w !== String(W)){
      el.width  = W * dpr;
      el.height = H * dpr;
      el.style.width  = W + 'px';
      el.style.height = H + 'px';
      el.dataset.w = String(W);
    }
    var ctx = el.getContext('2d');
    ctx.setTransform(1,0,0,1,0,0);
    ctx.scale(dpr, dpr);
    return {el:el, ctx:ctx, W:W, H:H};
  }

  var PAD=28, MAP_W=36, MAP_H=27;
  function cx(rx,W){return PAD+rx/MAP_W*(W-PAD*2);}
  function cy(ry,H){return PAD+(MAP_H-ry)/MAP_H*(H-PAD*2);}

  function clearAnimCanvas(){
    var a = getAnimCanvas();
    a.ctx.fillStyle = '#0a0e18';
    a.ctx.fillRect(0, 0, a.W, a.H);
  }

  function drawAnimCanvas(){
    if(!animState.mobile||!animState.particles.length) return;
    drawScene(
      animState.particles.map(function(p){return{x:p.x,y:p.y,bc:p.bc};}),
      animState.gbest, animState.currentK
    );
  }

  function drawFromSnapshot(snap){
    drawScene(snap.particles, snap.gbest, snap.k);
  }

  function drawScene(particles, gbest, k){
    var a   = getAnimCanvas();
    var W   = a.W, H = a.H;
    var ctx = a.ctx;
    // getAnimCanvas already set scale, just clear
    ctx.fillStyle='#0a0e18';
    ctx.fillRect(0,0,W,H);

    // Grid
    ctx.strokeStyle='rgba(255,255,255,0.04)';
    ctx.lineWidth=0.5;
    for(var gx=0;gx<=36;gx+=6){
      var lx=cx(gx,W);
      ctx.beginPath();ctx.moveTo(lx,PAD);ctx.lineTo(lx,H-PAD);ctx.stroke();
    }
    for(var gy=0;gy<=27;gy+=6){
      var ly=cy(gy,H);
      ctx.beginPath();ctx.moveTo(PAD,ly);ctx.lineTo(W-PAD,ly);ctx.stroke();
    }

    // Border
    ctx.strokeStyle='rgba(255,255,255,0.1)';
    ctx.lineWidth=0.8;
    ctx.strokeRect(PAD,PAD,W-PAD*2,H-PAD*2);

    // Scale labels
    ctx.fillStyle='rgba(255,255,255,0.25)';
    ctx.font='6px IBM Plex Mono,monospace';
    ctx.textAlign='center';
    for(var lx2=0;lx2<=36;lx2+=6) ctx.fillText(lx2+'m',cx(lx2,W),H-PAD+10);
    ctx.textAlign='right';
    for(var ly2=0;ly2<=27;ly2+=6) ctx.fillText(ly2+'m',PAD-3,cy(ly2,H)+3);

    // Other PR points (reference)
    var fl = animState.floor;
    PR_DB.forEach(function(pr){
      if(pr.id===animState.mobile.id) return;
      var px=cx(pr.x,W), py=cy(pr.y,H);
      ctx.beginPath();ctx.arc(px,py,2.5,0,Math.PI*2);
      ctx.fillStyle='rgba(96,165,250,0.3)';ctx.fill();
    });

    // AN Nodes (floor 4 only shown as triangles)
    if(fl==='Floor 4'){
      ANCHORS.forEach(function(an){
        var ax=cx(an.x,W), ay=cy(an.y,H);
        var s=5;
        ctx.beginPath();ctx.moveTo(ax,ay-s);ctx.lineTo(ax+s,ay+s);ctx.lineTo(ax-s,ay+s);ctx.closePath();
        ctx.fillStyle='#a855f7';ctx.fill();
        ctx.fillStyle='rgba(255,255,255,0.7)';
        ctx.font='7px IBM Plex Mono,monospace';ctx.textAlign='center';
        ctx.fillText(an.id,ax,ay-s-3);
      });
    }

    // Particles
    var maxCost=Math.max.apply(null,particles.map(function(p){return p.bc||1;}));
    particles.forEach(function(p){
      var ratio = maxCost>0 ? Math.min(1,(p.bc||0)/maxCost) : 0;
      // Color: hot (red=bad) to cool (green=good)
      var r=Math.round(239*ratio + 34*(1-ratio));
      var g=Math.round(68*ratio + 197*(1-ratio));
      var b=Math.round(68*ratio + 94*(1-ratio));
      ctx.beginPath();
      ctx.arc(cx(p.x,W),cy(p.y,H),3,0,Math.PI*2);
      ctx.fillStyle='rgba('+r+','+g+','+b+',0.75)';ctx.fill();
    });

    // Trails from gbest history (last 5)
    var trailStart=Math.max(0,animState.history.length-6);
    for(var ti=trailStart;ti<animState.history.length-1;ti++){
      var h1=animState.history[ti], h2=animState.history[ti+1];
      ctx.beginPath();
      ctx.moveTo(cx(h1.gbest.x,W),cy(h1.gbest.y,H));
      ctx.lineTo(cx(h2.gbest.x,W),cy(h2.gbest.y,H));
      ctx.strokeStyle='rgba(250,204,21,0.4)';ctx.lineWidth=1.5;ctx.stroke();
    }

    // gbest star
    if(gbest){
      var gx2=cx(gbest.x,W), gy2=cy(gbest.y,H);
      ctx.beginPath();ctx.arc(gx2,gy2,6,0,Math.PI*2);
      ctx.fillStyle='rgba(250,204,21,0.15)';ctx.fill();
      ctx.beginPath();ctx.arc(gx2,gy2,4,0,Math.PI*2);
      ctx.fillStyle='#facc15';ctx.fill();
      ctx.fillStyle='rgba(255,255,255,0.9)';
      ctx.font='bold 7px IBM Plex Mono,monospace';ctx.textAlign='center';
      ctx.fillText('gbest',gx2,gy2-8);
      ctx.fillText('('+gbest.x.toFixed(1)+','+gbest.y.toFixed(1)+')',gx2,gy2+14);
    }

    // Mobile (true position) — green star
    var mob=animState.mobile;
    var mx=cx(mob.x,W), my=cy(mob.y,H);
    // Pulse ring
    ctx.beginPath();ctx.arc(mx,my,10,0,Math.PI*2);
    ctx.strokeStyle='rgba(34,197,94,0.3)';ctx.lineWidth=2;ctx.stroke();
    ctx.beginPath();ctx.arc(mx,my,5,0,Math.PI*2);
    ctx.fillStyle='#22c55e';ctx.fill();
    ctx.fillStyle='rgba(255,255,255,0.9)';
    ctx.font='bold 7px IBM Plex Mono,monospace';ctx.textAlign='center';
    ctx.fillText(mob.id,mx,my-9);
    ctx.fillText('(จริง)',mx,my+15);

    // Error line: gbest → mobile
    if(gbest){
      var err=Math.sqrt((gbest.x-mob.x)**2+(gbest.y-mob.y)**2);
      ctx.beginPath();
      ctx.moveTo(cx(gbest.x,W),cy(gbest.y,H));
      ctx.lineTo(mx,my);
      ctx.strokeStyle='rgba(239,68,68,0.6)';ctx.lineWidth=1.2;
      ctx.setLineDash([4,3]);ctx.stroke();ctx.setLineDash([]);
      // Error label midpoint
      var emx=(cx(gbest.x,W)+mx)/2, emy=(cy(gbest.y,H)+my)/2;
      ctx.fillStyle='#ef4444';
      ctx.font='bold 8px IBM Plex Mono,monospace';ctx.textAlign='center';
      ctx.fillText(err.toFixed(2)+'m',emx,emy-4);
    }

    // Iteration label
    ctx.fillStyle='rgba(255,255,255,0.5)';
    ctx.font='bold 9px IBM Plex Mono,monospace';ctx.textAlign='left';
    ctx.fillText('k = '+k+' / '+animState.totalK,PAD+2,PAD-8);
    ctx.fillStyle=OPEN_ATRIUM_IDS.has(mob.id)?'#f59e0b':'#3b82f6';
    ctx.fillText(fl,PAD+2,PAD-18);
  }

  function updateIterInfo(k){
    var snap = animState.history[k];
    if(!snap) return;
    document.getElementById('anim-iter-num').textContent = k;
    document.getElementById('anim-gbest-cost').textContent = snap.gbest.bc.toFixed(3);
    document.getElementById('anim-gbest-xy').textContent = snap.gbest.x.toFixed(2)+', '+snap.gbest.y.toFixed(2);
    document.getElementById('anim-gbest-err').textContent = snap.err.toFixed(2);

    // Append row to log table
    var tbody = document.getElementById('anim-iter-log');
    var errColor = snap.err<2?'#22c55e':snap.err<4?'#f59e0b':'#ef4444';
    var tr = document.createElement('tr');
    tr.style.background = k%2===0?'transparent':'rgba(255,255,255,0.02)';
    tr.innerHTML='<td style="padding:4px 8px;font-family:var(--mono);color:var(--muted)">'+k+'</td>'+
      '<td style="padding:4px 8px;font-family:var(--mono);color:#3b82f6">'+snap.gbest.x.toFixed(2)+'</td>'+
      '<td style="padding:4px 8px;font-family:var(--mono);color:#3b82f6">'+snap.gbest.y.toFixed(2)+'</td>'+
      '<td style="padding:4px 8px;font-family:var(--mono);color:#10b981">'+snap.gbest.bc.toFixed(3)+'</td>'+
      '<td style="padding:4px 8px;font-family:var(--mono);color:'+errColor+'">'+snap.err.toFixed(2)+'</td>';
    tbody.appendChild(tr);
    tbody.parentElement.scrollTop = tbody.parentElement.scrollHeight;
  }

  function updateConvChart(){
    var history = animState.history;
    var labels  = history.map(function(h){return 'k='+h.k;});
    var costData = history.map(function(h){return +h.gbest.bc.toFixed(3);});
    var errData  = history.map(function(h){return +h.err.toFixed(3);});

    if(animState.convChart) animState.convChart.destroy();
    var ctx2 = document.getElementById('anim-conv-chart').getContext('2d');
    animState.convChart = new Chart(ctx2,{
      type:'line',
      data:{labels:labels,datasets:[
        {label:'Fitness (cost)',data:costData,borderColor:'#a855f7',
         backgroundColor:'rgba(168,85,247,0.08)',pointRadius:2,tension:.35,borderWidth:1.5,fill:true,yAxisID:'y'},
        {label:'Error (m)',data:errData,borderColor:'#f59e0b',
         backgroundColor:'rgba(245,158,11,0.06)',pointRadius:2,tension:.35,borderWidth:1.5,fill:false,yAxisID:'y2'}
      ]},
      options:{responsive:true,maintainAspectRatio:false,
        plugins:{legend:{display:true,labels:{color:'#6b7a99',font:{size:10},boxWidth:10,padding:10}}},
        scales:{
          y:{beginAtZero:true,grid:{color:'rgba(255,255,255,0.05)'},ticks:{color:'#6b7a99',font:{size:9}}},
          y2:{position:'right',beginAtZero:true,grid:{display:false},ticks:{color:'#f59e0b',font:{size:9}}}
        }
      }
    });
  }
})(); // end animator

})();
</script>
</body>
</html>
