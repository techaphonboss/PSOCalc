<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PSOCalc — KNN + PSO IPS Dashboard v2</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script>
// Fallback: ถ้า CDN โหลดไม่ได้ (เปิดจาก file://) แสดง instruction
window.addEventListener('load', function(){
  if(typeof XLSX === 'undefined' || typeof Chart === 'undefined'){
    document.body.insertAdjacentHTML('afterbegin',
      '<div style="position:fixed;top:0;left:0;right:0;z-index:9999;'+
      'background:#7f1d1d;color:#fef2f2;padding:12px 20px;font-size:13px;'+
      'font-family:monospace;border-bottom:2px solid #ef4444;text-align:center;">'+
      '⚠️ Library โหลดไม่ได้เพราะเปิดจาก file:// &nbsp;|&nbsp; '+
      'แก้ไข: เปิด CMD แล้วพิมพ์ &nbsp;'+
      '<span style="background:#991b1b;padding:2px 8px;border-radius:4px;">'+
      'python -m http.server 8080</span>&nbsp; '+
      'แล้วเปิด <a href="http://localhost:8080" target="_blank" '+
      'style="color:#fca5a5;">http://localhost:8080</a> ในโฟลเดอร์เดียวกับไฟล์ HTML'+
      '</div>'
    );
  }
});
</script>
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
tbody tr:hover{background:rgba(168,85,247,0.07);}

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
.fp-row{display:flex;align-items:center;justify-content:space-between;gap:10px;}
.fp-row-v{display:flex;flex-direction:column;gap:4px;}
.fp-label{font-size:11px;color:var(--muted);white-space:nowrap;}
.fp-coord{font-size:9px;color:var(--muted2);font-family:var(--mono);}
.fp-input{width:110px;padding:6px 10px;background:var(--bg4);border:1px solid var(--border);
  border-radius:var(--rs);color:var(--text);font-family:var(--mono);font-size:12px;text-align:right;}
.fp-input:focus{outline:none;border-color:var(--purple-b);}
.fp-row-v .fp-input{width:100%;}

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
    <button class="nav-tab" data-tab="floormap">🗺️ แผนผังชั้น</button>
    <button class="nav-tab" data-tab="table">📋 ตารางผล</button>
    <button class="nav-tab" data-tab="fixpos">🎯 Fix Position</button>
    <button id="btn-show-formula" class="nav-tab" style="margin-left:auto;background:rgba(96,165,250,0.10);color:#60a5fa;border:1px solid rgba(96,165,250,0.25);border-radius:6px;padding:4px 12px;">📐 สมการ</button>
  </div></nav>
</header>

<main class="main">

  <!-- ═══ TAB: OVERVIEW ═══ -->
  <div id="tab-overview" class="tab-panel active">
    <!-- UPLOAD -->
    <div id="drop-zone" class="drop-zone" style="cursor:pointer;position:relative;">
      <input type="file" id="file-input" accept=".xlsx,.xls,.csv"
        style="position:absolute;inset:0;width:100%;height:100%;opacity:0;cursor:pointer;z-index:2;">
      <div class="drop-icon">📂</div>
      <div class="drop-title">ลากและหย่อนไฟล์ข้อมูลดิบที่นี่</div>
      <div class="drop-sub">หรือ <strong>คลิกเพื่อเลือกไฟล์</strong> — รองรับ .xlsx / .xls / .csv</div>
      <div id="xlsx-warning" style="display:none;margin-top:14px;padding:10px 14px;
        background:rgba(127,29,29,0.4);border:1px solid rgba(239,68,68,0.5);
        border-radius:8px;font-size:12px;color:#fca5a5;text-align:left;line-height:1.8;">
        ⚠️ <b>ไม่สามารถโหลด XLSX library ได้</b> เพราะเปิดไฟล์จาก <code>file://</code><br>
        <b>วิธีแก้ไข:</b><br>
        1. เปิด Command Prompt / Terminal ในโฟลเดอร์ที่มีไฟล์ HTML<br>
        2. พิมพ์: <code style="background:rgba(0,0,0,0.4);padding:2px 6px;border-radius:3px;">python -m http.server 8080</code><br>
        3. เปิด Chrome แล้วไปที่: <code style="background:rgba(0,0,0,0.4);padding:2px 6px;border-radius:3px;">http://localhost:8080</code><br>
        4. คลิกไฟล์ HTML ที่ต้องการเปิด
      </div>
    </div>
    <div id="ov-empty" class="hidden"></div>
    <div id="ov-content" class="hidden">
      <div class="sec-title">อัตราความถูกต้อง — KNN (2 วิธี)</div>
      <div style="display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:12px;margin-bottom:24px;">
        <div class="metric-card"><div class="metric-accent" style="background:var(--amber)"></div><div class="metric-label" style="color:var(--amber)">Measurement (AVG รวม)</div><div class="metric-value" id="acc-m-avg">—</div><div class="metric-desc">ค่าเฉลี่ยรวมทั้งชั้น</div></div>
        <div class="metric-card"><div class="metric-accent" style="background:var(--rose)"></div><div class="metric-label" style="color:var(--rose)">RSSI Predict (AVG รวม)</div><div class="metric-value" id="acc-r-avg">—</div><div class="metric-desc">ค่าเฉลี่ยรวมทั้งชั้น</div></div>
        <div class="metric-card"><div class="metric-accent" style="background:var(--purple)"></div><div class="metric-label" style="color:var(--purple)">Particles โหลดแล้ว</div><div class="metric-value" id="total-loaded">55</div><div class="metric-desc">พร้อมคำนวณ PSO ทุกตัว</div></div>
      </div>
      <!-- hidden compat ids -->
      <span id="acc-m-top3" style="display:none"></span>
      <span id="acc-r-top3" style="display:none"></span>
      <div class="charts-grid">
        <div class="chart-card">
          <div class="chart-title">อัตราความถูกต้องรวม (%)</div>
          <div class="chart-sub">Meas.Avg vs RSSI Avg</div>
          <div class="chart-wrap"><canvas id="accChart"></canvas></div>
        </div>
        <div class="chart-card">
          <div class="chart-title">ความผิดพลาดแยกตามชั้น (จุด)</div>
          <div class="chart-sub">จำนวน Particle ที่ทำนายผิดชั้น</div>
          <div class="chart-wrap"><canvas id="errChart"></canvas></div>
        </div>
      </div>
      <div class="chart-card-full">
        <div class="chart-head">
          <div>
            <div class="chart-title">% ความถูกต้องแยกตามชั้น — Meas.Avg vs RSSI Avg (กราฟเส้น)</div>
            <div class="chart-sub">แสดงความแม่นยำของแต่ละวิธีในแต่ละชั้น</div>
          </div>
          <div class="line-legend">
            <div class="ll-item"><div class="ll-dot" style="background:var(--amber)"></div>Meas.Avg</div>
            <div class="ll-item"><div class="ll-dot" style="background:var(--rose)"></div>RSSI Avg</div>
          </div>
        </div>
        <div class="chart-wrap-tall"><canvas id="floorSumChart"></canvas></div>
      </div>
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
        <span style="color:var(--muted);margin-left:12px;">η: F4=2.0 · F3=1.8 · F2=1.6 · F1=1.4 | FAF: F3=−5 F2=−10 F1=−15 dBm</span>
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
          <div style="display:grid;grid-template-columns:repeat(5,1fr);gap:8px;">
            <div class="pso-result-card"><div class="prc-label" style="color:var(--purple)">Accuracy</div><div class="prc-value" id="pso-acc-m">—</div><div class="prc-sub">ชั้นถูกต้อง</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--ok)">≤ 3m</div><div class="prc-value" id="pso-lt3-m">—</div><div class="prc-sub">PR</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--amber)">≤ 6m</div><div class="prc-value" id="pso-lt6-m">—</div><div class="prc-sub">PR</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--rose)">≤ 10m</div><div class="prc-value" id="pso-lt10-m">—</div><div class="prc-sub">PR</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--purple)">Avg Error</div><div class="prc-value" id="pso-avg-err-m">—</div><div class="prc-sub">เมตร</div></div>
          </div>
          <!-- Per-floor avg error Meas -->
          <div style="margin-top:10px;" id="pso-floor-err-m"></div>
        </div>

        <!-- RSSI Predict side -->
        <div style="background:var(--bg3);border:1px solid var(--green-b);border-radius:var(--r);padding:16px;">
          <div style="font-size:10px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--green);margin-bottom:12px;">
            RSSI Predict (diff floor model)
          </div>
          <div style="display:grid;grid-template-columns:repeat(5,1fr);gap:8px;">
            <div class="pso-result-card"><div class="prc-label" style="color:var(--green)">Accuracy</div><div class="prc-value" id="pso-acc-r">—</div><div class="prc-sub">ชั้นถูกต้อง</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--ok)">≤ 3m</div><div class="prc-value" id="pso-lt3-r">—</div><div class="prc-sub">PR</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--amber)">≤ 6m</div><div class="prc-value" id="pso-lt6-r">—</div><div class="prc-sub">PR</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--rose)">≤ 10m</div><div class="prc-value" id="pso-lt10-r">—</div><div class="prc-sub">PR</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--green)">Avg Error</div><div class="prc-value" id="pso-avg-err-r">—</div><div class="prc-sub">เมตร</div></div>
          </div>
          <!-- Per-floor avg error RSSI -->
          <div style="margin-top:10px;" id="pso-floor-err-r"></div>
        </div>
      </div>

      <div style="display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-bottom:20px;">
        <div class="chart-card-full" style="margin-bottom:0">
          <div class="chart-head"><div>
            <div class="chart-title" style="color:var(--purple)">Convergence — Measurement (AVG รวม)</div>
            <div class="chart-sub">Fitness ลู่เข้าตาม iteration · Meas. RSSI</div>
          </div></div>
          <div class="chart-wrap-tall"><canvas id="psoConvergeChartM"></canvas></div>
        </div>
        <div class="chart-card-full" style="margin-bottom:0">
          <div class="chart-head"><div>
            <div class="chart-title" style="color:var(--green)">Convergence — RSSI Predict (AVG รวม)</div>
            <div class="chart-sub">Fitness ลู่เข้าตาม iteration · RSSI Pred.</div>
          </div></div>
          <div class="chart-wrap-tall"><canvas id="psoConvergeChart"></canvas></div>
        </div>
      </div>

      <div class="table-card">
        <div class="table-head">
          <div><div class="th-title">ผลลัพธ์ Hybrid KNN→PSO ทุก Particle</div>
          <div class="th-sub">คลิกที่แถว (ซ้าย=Meas, ขวา=RSSI) เพื่อดู Particle ที่นำไป PSO · KNN ทำนายชั้น → PSO หาพิกัด XY เฉพาะชั้นนั้น</div></div>
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
              <th class="purple">KNN ทำนาย</th>
              <th class="purple">gbest X,Y</th>
              <th class="purple">Error (m)</th>
              <th class="purple">Seed (คลิกแถว)</th>
              <th class="green">KNN ทำนาย</th>
              <th class="green">gbest X,Y</th>
              <th class="green">Error (m)</th>
              <th class="green">Seed (คลิกแถว)</th>
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
      <div class="chart-card-full"><div class="chart-head"><div><div class="chart-title">ปริมาณการคลาดเคลื่อน 0 / 1 / 2 / 3 ชั้น ทั้ง 2 วิธี KNN</div></div></div><div class="chart-wrap-tall"><canvas id="devChart"></canvas></div></div>

      <!-- CDF Chart -->
      <div class="chart-card-full" style="margin-top:16px;">
        <div class="chart-head">
          <div>
            <div class="chart-title">CDF — Cumulative Distribution Function ของ Floor Error</div>
            <div class="chart-sub">ความน่าจะเป็นสะสม (%) ที่ค่า Error ≤ n ชั้น · Meas.Avg vs RSSI Avg</div>
          </div>
          <div class="line-legend">
            <div class="ll-item"><div class="ll-dot" style="background:rgba(245,158,11,0.9)"></div>Meas.Avg</div>
            <div class="ll-item"><div class="ll-dot" style="background:rgba(244,63,94,0.9)"></div>RSSI Avg</div>
          </div>
        </div>
        <div class="chart-wrap-tall"><canvas id="cdfChart"></canvas></div>
      </div>
    </div>
  </div>

  <!-- ═══ TAB: COMPARE ═══ -->
  <div id="tab-compare" class="tab-panel">
    <div id="cmp-empty" class="empty-state"><div style="font-size:48px">📈</div><p>อัพโหลดไฟล์เพื่อดูกราฟเปรียบเทียบ</p></div>
    <div id="cmp-content" class="hidden">
      <div class="chart-card-full">
        <div class="chart-head">
          <div>
            <div class="chart-title">เปรียบเทียบ Measurement (AVG รวม) vs RSSI Predict (AVG รวม) แยกตามชั้น</div>
            <div class="chart-sub">กราฟเส้น — % ความถูกต้องในแต่ละชั้น</div>
          </div>
          <div class="line-legend">
            <div class="ll-item"><div class="ll-dot" style="background:var(--amber)"></div>Meas. Avg</div>
            <div class="ll-item"><div class="ll-dot" style="background:var(--rose)"></div>RSSI Avg</div>
          </div>
        </div>
        <div class="chart-wrap-tall"><canvas id="lineAvgChart"></canvas></div>
      </div>
      <!-- hidden compat canvases -->
      <canvas id="lineTop3Chart" style="display:none"></canvas>
      <canvas id="lineAllChart"  style="display:none"></canvas>
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
    <div id="tbl-empty" class="empty-state"><div style="font-size:48px">📋</div><p>กด "รัน PSO" หรืออัพโหลดไฟล์เพื่อดูตารางผล</p></div>
    <div id="tbl-content" class="hidden">
      <div class="table-card">
        <div class="table-head">
          <div><div class="th-title">ตารางผลการทำนายชั้น</div>
          <div class="th-sub">KNN ใช้วิธี Avg รวม · ผลตรงกับตาราง PSO ทุก Particle</div></div>
          <div class="table-head-right">
            <span class="count-pill" id="total-records">0</span>
            <button class="btn btn-export" id="btn-export">
              <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/></svg>
              Export Excel
            </button>
          </div>
        </div>
        <div class="tbl-scroll">
          <table><thead>
            <tr style="background:var(--bg4);">
              <th rowspan="2">Floor จริง</th><th rowspan="2">Particle</th>
              <th colspan="3" style="color:var(--amber);text-align:center;border-bottom:1px solid var(--amber-b);">Measurement (AVG รวม)</th>
              <th colspan="3" style="color:var(--rose);text-align:center;border-bottom:1px solid var(--rose-b);">RSSI Predict (AVG รวม)</th>
              <th rowspan="2">คลาดเคลื่อน<br>Meas.Avg</th>
              <th rowspan="2">คลาดเคลื่อน<br>RSSI Avg</th>
            </tr>
            <tr>
              <th class="amber">KNN ทำนาย</th>
              <th class="amber">PSO gbest</th>
              <th class="amber">XY Error</th>
              <th class="rose">KNN ทำนาย</th>
              <th class="rose">PSO gbest</th>
              <th class="rose">XY Error</th>
            </tr>
          </thead>
          <tbody id="table-body"></tbody></table>
        </div>
      </div>
    </div>
  </div>

  </div>

  <!-- ═══ TAB: FIX POSITION ═══ -->
  <div id="tab-fixpos" class="tab-panel">
    <div class="sec-title">🎯 Real-world Validation Test — ระบุตำแหน่ง Mobile จากค่า RSSI จริง</div>
    <div class="th-sub" style="margin-bottom:20px;">กรอก RSSI ที่วัดจริง → PSO หาพิกัด (X,Y) → เปรียบเทียบกับตำแหน่งจริง (ถ้าทราบ) · AN Node ตำแหน่งเดิม · ไม่ต้องรู้ตำแหน่ง Mobile ล่วงหน้า</div>

    <div style="display:grid;grid-template-columns:360px 1fr;gap:18px;align-items:start;">

      <!-- ══ LEFT: INPUT PANEL ══ -->
      <div style="display:flex;flex-direction:column;gap:14px;">

        <!-- 1. RSSI Inputs -->
        <div class="chart-card" style="padding:18px;">
          <div class="chart-title" style="margin-bottom:14px;">📡 ค่า RSSI ที่วัดได้ (dBm)</div>
          <div style="display:flex;flex-direction:column;gap:10px;">
            <div class="fp-row">
              <label class="fp-label">AN1 <span class="fp-coord">(7.96, 9.07) F4</span></label>
              <input id="fp-rssi-an1" type="number" class="fp-input" value="-65" step="0.1">
            </div>
            <div class="fp-row">
              <label class="fp-label">AN3 <span class="fp-coord">(28.10, 9.57) F4</span></label>
              <input id="fp-rssi-an2" type="number" class="fp-input" value="-70" step="0.1">
            </div>
            <div class="fp-row">
              <label class="fp-label">AN4 <span class="fp-coord">(8.77, 22.01) F4</span></label>
              <input id="fp-rssi-an3" type="number" class="fp-input" value="-72" step="0.1">
            </div>
          </div>
          <!-- Realtime RSSI Predict preview -->
          <div style="margin-top:12px;padding:10px;background:var(--bg4);border-radius:var(--rs);font-size:11px;">
            <div style="color:var(--muted);margin-bottom:5px;">📊 RSSI Predict — realtime</div>
            <div id="fp-predict-preview" style="font-family:var(--mono);line-height:1.9;color:var(--blue);">—</div>
          </div>
        </div>

        <!-- 2. Floor Selection -->
        <div class="chart-card" style="padding:18px;">
          <div class="chart-title" style="margin-bottom:12px;">🏢 ชั้นที่ทดสอบ</div>
          <div style="display:flex;flex-direction:column;gap:8px;">
            <label style="display:flex;align-items:center;gap:8px;font-size:12px;color:var(--muted);cursor:pointer;">
              <input type="radio" name="fp-floor-mode" value="auto" checked style="accent-color:var(--purple);">
              <span><b style="color:var(--text)">Auto</b> — PSO รันทุกชั้น เลือก cost ต่ำสุด <span style="color:var(--amber);font-size:10px;">(แนะนำ)</span></span>
            </label>
            <label style="display:flex;align-items:center;gap:8px;font-size:12px;color:var(--muted);cursor:pointer;">
              <input type="radio" name="fp-floor-mode" value="manual" style="accent-color:var(--green);">
              <span><b style="color:var(--text)">Manual</b> — ระบุชั้นเอง (รู้อยู่แล้ว)</span>
            </label>
            <select id="fp-floor-select" style="width:100%;margin-top:4px;background:var(--bg4);border:1px solid var(--border2);
              border-radius:var(--rs);color:var(--muted);font-size:12px;padding:6px 10px;display:none;">
              <option value="Floor 4">Floor 4</option>
              <option value="Floor 3">Floor 3</option>
              <option value="Floor 2">Floor 2</option>
              <option value="Floor 1">Floor 1</option>
            </select>
            <!-- Multi-floor cost comparison (shown after run) -->
            <div id="fp-floor-costs" style="display:none;margin-top:6px;padding:8px 10px;background:var(--bg4);border-radius:var(--rs);font-size:11px;font-family:var(--mono);"></div>
          </div>
        </div>

        <!-- 3. Path Loss Parameters -->
        <div class="chart-card" style="padding:18px;">
          <div class="chart-title" style="margin-bottom:12px;">⚙️ Path Loss Parameters</div>
          <div style="display:flex;flex-direction:column;gap:9px;">
            <div class="fp-row">
              <label class="fp-label">PLd₀ (dBm) <span class="fp-coord">@ 1m calibrated</span></label>
              <input id="fp-pld0" type="number" class="fp-input" value="-53" step="0.5">
            </div>
            <div class="fp-row">
              <label class="fp-label">AN Height (m)</label>
              <input id="fp-an-height" type="number" class="fp-input" value="10.5" step="0.1">
            </div>
            <div class="fp-row">
              <label class="fp-label">Mobile Height (m)</label>
              <input id="fp-mob-height" type="number" class="fp-input" value="1.5" step="0.1">
            </div>
          </div>
          <div style="margin-top:8px;font-size:10px;color:var(--muted2);font-family:var(--mono);">
            η ใช้ตาม floor: F4=2.0 · F3=1.8 · F2=1.6 · F1=1.4<br>
            FAF: F3=−5 · F2=−10 · F1=−15 dBm
          </div>
        </div>

        <!-- 4. PSO Parameters -->
        <div class="chart-card" style="padding:18px;">
          <div class="chart-title" style="margin-bottom:10px;">🔬 PSO Parameters</div>
          <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;">
            <div class="fp-row-v"><label class="fp-label">n particles</label><input id="fp-n" type="number" class="fp-input" value="60"></div>
            <div class="fp-row-v"><label class="fp-label">k iterations</label><input id="fp-k" type="number" class="fp-input" value="50"></div>
            <div class="fp-row-v"><label class="fp-label">Multi-run R</label><input id="fp-r" type="number" class="fp-input" value="5"></div>
            <div class="fp-row-v"><label class="fp-label">Grid ±m</label><input id="fp-grid" type="number" class="fp-input" value="2.0" step="0.5"></div>
          </div>
        </div>

        <!-- 5. True Position (optional) -->
        <div class="chart-card" style="padding:18px;border-color:rgba(34,197,94,0.25);">
          <div class="chart-title" style="margin-bottom:10px;color:var(--green);">📍 ตำแหน่งจริง (optional)</div>
          <div style="font-size:11px;color:var(--muted);margin-bottom:10px;">กรอกเพื่อคำนวณ error — ถ้าไม่ทราบ เว้นไว้</div>
          <div style="display:flex;flex-direction:column;gap:9px;">
            <div class="fp-row">
              <label class="fp-label">X จริง (m)</label>
              <input id="fp-true-x" type="number" class="fp-input" placeholder="เช่น 13.23" step="0.01">
            </div>
            <div class="fp-row">
              <label class="fp-label">Y จริง (m)</label>
              <input id="fp-true-y" type="number" class="fp-input" placeholder="เช่น 11.32" step="0.01">
            </div>
            <div class="fp-row">
              <label class="fp-label">ชั้นจริง</label>
              <select id="fp-true-floor" class="fp-input" style="width:110px;">
                <option value="">—</option>
                <option value="Floor 4">Floor 4</option>
                <option value="Floor 3">Floor 3</option>
                <option value="Floor 2">Floor 2</option>
                <option value="Floor 1">Floor 1</option>
              </select>
            </div>
          </div>
        </div>

        <!-- Run button -->
        <button id="fp-btn-run" class="btn btn-pso" style="width:100%;padding:13px;font-size:14px;">
          ⚡ Run 2-Stage PSO — ตรวจชั้น → หาตำแหน่ง
        </button>
      </div>

      <!-- ══ RIGHT: RESULTS ══ -->
      <div style="display:flex;flex-direction:column;gap:14px;">

        <!-- 2-Stage Progress -->
        <div id="fp-stage-panel" style="display:none;">
          <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:10px;">
            <!-- Stage 1 -->
            <div id="fp-stage1-card" style="padding:14px;border-radius:var(--rs);
              border:1px solid rgba(168,85,247,0.3);background:rgba(168,85,247,0.06);">
              <div style="font-size:10px;color:#a855f7;font-weight:700;margin-bottom:8px;">
                STAGE 1 — Floor Detection</div>
              <div id="fp-stage1-status" style="font-size:11px;color:var(--muted);">รอ...</div>
              <div id="fp-stage1-costs" style="margin-top:8px;font-size:11px;font-family:var(--mono);"></div>
              <div id="fp-stage1-result" style="display:none;margin-top:10px;">
                <div style="font-size:20px;font-weight:700;color:#facc15;" id="fp-stage1-floor">—</div>
                <div style="display:flex;align-items:center;gap:8px;margin-top:6px;">
                  <div style="font-size:11px;color:var(--muted);">Confidence</div>
                  <div style="flex:1;height:6px;background:var(--bg4);border-radius:3px;">
                    <div id="fp-conf-bar" style="height:6px;border-radius:3px;width:0%;
                      background:linear-gradient(90deg,#ef4444,#f59e0b,#22c55e);transition:width 0.5s;"></div>
                  </div>
                  <div id="fp-conf-pct" style="font-size:12px;font-weight:700;color:#22c55e;">—</div>
                </div>
                <div id="fp-ambig-warn" style="display:none;margin-top:8px;padding:6px 10px;
                  background:rgba(245,158,11,0.12);border:1px solid rgba(245,158,11,0.3);
                  border-radius:6px;font-size:10px;color:#f59e0b;">
                  ⚠️ ไม่แน่ใจ — cost ต่างกันน้อย อาจเป็น 2 ชั้น
                </div>
              </div>
            </div>
            <!-- Stage 2 -->
            <div id="fp-stage2-card" style="padding:14px;border-radius:var(--rs);
              border:1px solid rgba(250,204,21,0.2);background:rgba(250,204,21,0.04);opacity:0.4;">
              <div style="font-size:10px;color:#facc15;font-weight:700;margin-bottom:8px;">
                STAGE 2 — PSO Positioning</div>
              <div id="fp-stage2-status" style="font-size:11px;color:var(--muted);">รอ Stage 1...</div>
              <div id="fp-stage2-result" style="display:none;margin-top:10px;">
                <div style="font-size:18px;font-weight:700;color:#60a5fa;"
                  id="fp-stage2-xy">—</div>
                <div style="font-size:11px;color:var(--muted);margin-top:4px;"
                  id="fp-stage2-cost">RMSE: —</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Result cards -->
        <div id="fp-result-cards" style="display:none;">
          <div style="display:grid;grid-template-columns:repeat(5,1fr);gap:10px;margin-bottom:14px;">
            <div class="pso-result-card"><div class="prc-label" style="color:var(--amber)">Est. X</div><div class="prc-value" id="fp-res-x">—</div><div class="prc-sub">เมตร</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--amber)">Est. Y</div><div class="prc-value" id="fp-res-y">—</div><div class="prc-sub">เมตร</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--purple)">Floor</div><div class="prc-value" id="fp-res-floor">—</div><div class="prc-sub">ประมาณ</div></div>
            <div class="pso-result-card"><div class="prc-label" style="color:var(--green)">RMSE Cost</div><div class="prc-value" id="fp-res-cost">—</div><div class="prc-sub">dBm</div></div>
            <div class="pso-result-card" id="fp-error-card" style="display:none;"><div class="prc-label" style="color:var(--rose)">XY Error</div><div class="prc-value" id="fp-res-error">—</div><div class="prc-sub">เมตร</div></div>
          </div>
          <div id="fp-dist-info" style="padding:9px 14px;background:var(--bg3);border-radius:var(--rs);font-size:11px;font-family:var(--mono);color:var(--muted);margin-bottom:4px;"></div>
          <div id="fp-error-detail" style="display:none;padding:9px 14px;background:rgba(34,197,94,0.07);border:1px solid rgba(34,197,94,0.2);border-radius:var(--rs);font-size:12px;font-family:var(--mono);color:var(--ok);margin-bottom:4px;"></div>
        </div>

        <!-- Convergence chart -->
        <div class="chart-card" style="padding:14px;">
          <div class="chart-title">📈 Convergence — Cost per Iteration</div>
          <div class="chart-wrap"><canvas id="fp-conv-chart"></canvas></div>
        </div>

        <!-- Map -->
        <div class="chart-card" style="padding:0;overflow:hidden;position:relative;">
          <div style="position:absolute;top:8px;left:10px;z-index:2;font-size:10px;
            color:var(--muted);font-family:var(--mono);" id="fp-map-label">แผนผัง Floor 4</div>
          <canvas id="fp-map-canvas" style="width:100%;display:block;"></canvas>
        </div>

        <!-- Test Log -->
        <div class="chart-card" style="padding:16px;">
          <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px;">
            <div class="chart-title">📋 Test Log — ประวัติการทดสอบ</div>
            <button id="fp-btn-clear-log" class="btn" style="font-size:10px;padding:3px 10px;
              background:rgba(239,68,68,0.1);color:#ef4444;border-color:rgba(239,68,68,0.3);">ล้าง</button>
          </div>
          <div id="fp-test-log-empty" style="font-size:12px;color:var(--muted2);text-align:center;padding:20px;">
            ยังไม่มี test cases — รัน PSO และกด "บันทึก" เพื่อเพิ่ม
          </div>
          <div id="fp-test-log" style="display:none;">
            <table style="width:100%;border-collapse:collapse;font-size:11px;">
              <thead><tr style="background:var(--bg4);">
                <th style="padding:5px 8px;color:var(--muted);text-align:left;">#</th>
                <th style="padding:5px 8px;color:var(--amber);">Est. Floor</th>
                <th style="padding:5px 8px;color:var(--amber);">Est. X,Y</th>
                <th style="padding:5px 8px;color:var(--green);">True Floor</th>
                <th style="padding:5px 8px;color:var(--green);">True X,Y</th>
                <th style="padding:5px 8px;color:var(--rose);">XY Error</th>
                <th style="padding:5px 8px;color:var(--muted);">Floor ✓</th>
                <th style="padding:5px 8px;color:var(--muted);">RMSE</th>
              </tr></thead>
              <tbody id="fp-log-body"></tbody>
            </table>
            <!-- Summary -->
            <div id="fp-log-summary" style="margin-top:10px;padding:8px 12px;background:var(--bg4);
              border-radius:var(--rs);font-size:11px;font-family:var(--mono);color:var(--muted);"></div>
          </div>
          <button id="fp-btn-save" class="btn btn-pso" style="width:100%;margin-top:10px;
            padding:8px;font-size:12px;display:none;">
            💾 บันทึก Test Case นี้
          </button>
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
<!-- Deviation Popup Modal -->
<div id="dev-popup-overlay" style="display:none;position:fixed;inset:0;background:rgba(0,0,0,0.7);z-index:999;backdrop-filter:blur(4px);" onclick="document.getElementById('dev-popup-overlay').style.display='none'"></div>
<div id="dev-popup" style="display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);z-index:1000;background:var(--bg2);border:1px solid var(--border2);border-radius:var(--r);padding:20px;min-width:360px;max-width:520px;max-height:80vh;overflow-y:auto;box-shadow:0 20px 60px rgba(0,0,0,0.5);">
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:14px;">
    <div id="dev-popup-title" style="font-size:14px;font-weight:700;color:var(--text);"></div>
    <button onclick="document.getElementById('dev-popup-overlay').style.display='none';document.getElementById('dev-popup').style.display='none';" style="background:none;border:none;color:var(--muted);font-size:18px;cursor:pointer;line-height:1;">×</button>
  </div>
  <div id="dev-popup-body"></div>
</div>

<!-- ═══ FORMULA MODAL ═══ -->
<div id="formula-overlay" style="display:none;position:fixed;inset:0;background:rgba(0,0,0,0.75);z-index:1000;backdrop-filter:blur(4px);" onclick="document.getElementById('formula-overlay').style.display='none'"></div>
<div id="formula-modal" style="display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  width:min(900px,95vw);max-height:88vh;overflow-y:auto;z-index:1001;
  background:#0d1220;border:1px solid rgba(96,165,250,0.3);border-radius:12px;padding:28px 32px;">
  <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:20px;">
    <div style="font-size:16px;font-weight:700;color:#60a5fa;">📐 สมการและวิธีคิดคำนวณ PSO Indoor Positioning</div>
    <button onclick="document.getElementById('formula-modal').style.display='none';document.getElementById('formula-overlay').style.display='none';"
      style="background:none;border:none;color:#6b7a99;font-size:20px;cursor:pointer;padding:4px 8px;">✕</button>
  </div>
  <div id="formula-body" style="font-size:13px;line-height:1.9;color:#c8d4e8;"></div>
</div>

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


// ── Floor Plan Images (base64 embedded) ──
var FLOOR_IMAGES = {
  "Floor 4": "data:image/png;base64,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MgMQWvDIV4ZAQFrwyFeGQDAhlXZhkSvDICAteGQrwyAhcBXhkWuzDIDEFrwyFeGQEBa8MhXhkAwIZV2YZErwyAgLXhkK8MgGBC4EAAAAbEKAiw0fFcrWr0IibVNc3o+yI5uDVVqcELUxcl89XL7sXnTwGrl9yLzp4AFrQg1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl9yLzp4DVy+5F508ABaA1cvuRedPAauBTqRedPAFwFoE1cvuRedPAauX3IvOngALQGrl9yLzp4DVy+5F508ABaA1cCnUi86eA1cvuRedPAuBBaA1cvuRedPAauX3IvOngALQGrl9yLzp4DVwKdSLzp4AuAtAmrl9yLzp4DVy+5F508ABaA1cvuRedPAauX3IvOngALQGrgU6kXnTwGrl9yLzp4FwILQGrl92Lzp4GEWAiMV8JyuanSiptQzM4G2M1uDlurwUiaYLtIuBC4FEoAAAAAuBC4EAAAC4ELgQAAABvTHvon11/E0TemPfRPrr+JalEvmAd49Dno9i+kO3ZyQW1oVkyslK+sR5p8FYtEV7WNajEVFVVc5MdiVL2RMxG90cHftPfRlaOhmj77RtacT1lbUmJKBLQ4DlSJBguVjphX19lqvS61FSq7ew4vQP0d6aadNmnaKWDHtJkrRIz2vZDY1V6EvPciKuzoTaRTMVReFqommbS6tgQ7vZnon9ItpWbas/I6KzsaFZMeJLzzUcxIkKIxEVzdWrr7lRFRfZRek+s76HvSVJxrHgzOisxDfbL9XIfpoSpFfcV91VR1GrdRVo6nQovHd2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qqjUVH3rq1XZsXp2dJoxpWYgwIceLCuMiOc1qrSqq2ldnTj/APtCZmUbHxRzkSiOVE4mTnvr1ndCY/AxrwyK9duHQmHwKrF9++7MX3067syV4ZCuzDIC3377sxffvuzJXhkK8MgLffvuzF9++7MleGQrwyAt99Ou7MX377syV2YZCvDIC3377sxffvuzJXhkK8MgLffvuzF99Ou7MleGQrswyAt9++7MX377syV4ZCvDIC3377sxffvuzJXhkK8MgLffTruzF9++7MldmGQrwyAt9++7MX377syV4ZCvDIC3377sxffTruzJXhkK7MMgLffvuzF9++7MleGQrwyAt9++7MX377syV4ZCvDIC33067sxffvuzJXZhkK8MgLffvuzF9++7MleGQrwyAt9++7MX3067syV4ZCuzDIC3377sxffvuzJXhkK8MgLffvuzF9++7MleGQrwyAt99Ou7MX377syV2YZCvDIC3377sxffvuzJXhkK8MgLffvuzF99Ou7MleGQrswyAt9++7MX377syV4ZCvDIC3377sxffvuzJXhkK8MgLffTruzF9++7MldmGQrwyAt9++7MX377syV4ZCvDIC3377szKK9993tu6y4mFeGRlEX23dHWXACX377sxffvuzJXhkK8MgLffvuzF9++7MleGQrwyAt99Ou7MX377syV2YZCvDIC3377sxffvuzJXhkK8MgLffvuzF99Ou7MleGQrswyAt9++7MX377syV4ZCvDIC3377sxffvuzJXhkK8MgLffTruzF9++7MldmGQrwyAt9++7MX377syV4ZCvDIC3377sxffTruzJXhkK7MMgLffvuzF9++7MleGQrwyAt9++7MX377syV4ZCvDIC33067sxffvuzJXZhkK8MgLffvuzF9++7MleGQrwyAt9++7MX3067syV4ZCuzDIC3377sxffvuzJXhkK8MgLffvuzF9++7MleGQrwyAt99Ou7MX377syV2YZCvDIC3377sxffvuzJXhkK8MgLffvuzF99Ou7MleGQrswyAt9++7MX377syV4ZCvDIC3377sxffvuzJXhkK8MgLffTruzF9++7MldmGQrwyAt9++7MX377syV4ZCvDIC3377sxffTruzJXhkK7MMgM4b33l9t3QuPwMb7992Yhr7S9HQuHwJXhkBb7992Yvv33Zkrwy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",
  "Floor 3": "data:image/png;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/4gHYSUNDX1BST0ZJTEUAAQEAAAHIAAAAAAQwAABtbnRyUkdCIFhZWiAH4AABAAEAAAAAAABhY3NwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAA9tYAAQAAAADTLQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAlkZXNjAAAA8AAAACRyWFlaAAABFAAAABRnWFlaAAABKAAAABRiWFlaAAABPAAAABR3dHB0AAABUAAAABRyVFJDAAABZAAAAChnVFJDAAABZAAAAChiVFJDAAABZAAAAChjcHJ0AAABjAAAADxtbHVjAAAAAAAAAAEAAAAMZW5VUwAAAAgAAAAcAHMAUgBHAEJYWVogAAAAAAAAb6IAADj1AAADkFhZWiAAAAAAAABimQAAt4UAABjaWFlaIAAAAAAAACSgAAAPhAAAts9YWVogAAAAAAAA9tYAAQAAAADTLXBhcmEAAAAAAAQAAAACZmYAAPKnAAANWQAAE9AAAApbAAAAAAAAAABtbHVjAAAAAAAAAAEAAAAMZW5VUwAAACAAAAAcAEcAbwBvAGcAbABlACAASQBuAGMALgAgADIAMAAxADb/2wBDAAUDBAQEAwUEBAQFBQUGBwwIBwcHBw8LCwkMEQ8SEhEPERETFhwXExQaFRERGCEYGh0dHx8fExciJCIeJBweHx7/2wBDAQUFBQcGBw4ICA4eFBEUHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh7/wAARCAM0BOgDASIAAhEBAxEB/8QAHAABAQADAQEBAQAAAAAAAAAAAAYDBQcECAIB/8QAXhAAAQQBAgMBCggKBgcGBAMJAAECAwQFBhEHEiETFBYXIjFXdZPT1BU2QVFUVZa0CCMyN1ZhlbGz0TVCcXOBlCQlM1JikdIYNENEdMJTY3KhOIKSo8EmWGalg7Lw/8QAGgEBAQEBAQEBAAAAAAAAAAAAAAECAwQFBv/EADsRAQABAgMFBgQGAQQCAgMAAAABAhEDITEEEkFR8GFxgZGhsQUiwdETFDNS4fEyBhVCsoLCFnIjYqL/2gAMAwEAAhEDEQA/APnEAHocQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAdPvW9M6V0Xo98mhcLmbeUxkluzauzWUer+6p40REjla1ERrGp5DVd/OmvNVpT1973gcS/idw79ASffrJCkF1386a81WlPX3veB386a81WlPX3veCFBbC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVaU9fe94HfzprzVaU9fe94IUCwuu/nTXmq0p6+97wO/nTXmq0p6+97wQoFhdd/OmvNVpT1973gd/OmvNVpT1973ghQLC67+dNearSnr73vA7+dNearSnr73vBCgWF1386a81WlPX3veB386a81WlPX3veCFAsLrv5015qtKevve8Dv5015qtKevve8EKBYXXfzprzVa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AAAAAAAAAAAAAAAAdp19pXTmoMljMna4hYXCTPwmNY6nfq2mzM5KkTd1RI9tl23RU6KiopO+DvSnne0p6m17I8XHH440fQGJ+4wkKSFdH8HelPO9pT1Nr2Q8HelPO9pT1Nr2RzgCyOj+DvSnne0p6m17IxWeH+lomI5nFvSj13227C5/+6FTnoAuu8bTXnV0p6i97uO8bTXnV0p6i97uQoKLrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7jvG0151dKeove7kKALrvG0151dKeove7nt8HelPO9pT1Nr2RzgEHR/B3pTzvaU9Ta9kPB3pTzvaU9Ta9kc4AsOj+DvSnne0p6m17IeDvSnne0p6m17I5wBYdr4W6L0xieJGncnFxO09kX1clBM2rVq2nzTK16LyMb2XVy+REJS7onTb7k7/CnpZvNI5eV1e8ip18i/6OeLgZ+eXR3pmr/EaSeS/pGz/fP/AHqBZ942mvOrpT1F73cd42mvOrpT1F73chQUXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAdttaR03a4TYTFycScBAyvl7szbbqtxIZVfHXRWNcsKbubyoqp/xNJnwd6U872lPU2vZHizf5idMen8l/BqEKRXR/B3pTzvaU9Ta9kPB3pTzvaU9Ta9kc4Asjo/g70p53tKepteyHg70p53tKepteyOcAWHQbWgdLxcvJxa0o/fff8Rc6f8oVMHeNprzq6U9Re93I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AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAF1w0+J3ET0BH9+rEKXXDT4ncRPQEf36sSRCgAoAAAAAAAAAAAAAAAAAAAAAAAAAAAAAOn36mmNVaN0hG/XmFw9vFYySnarXYLXOj1szSIqLHE5qpyyN6ovzmq7xtNedXSnqL3u5Cglhdd42mvOrpT1F73cd42mvOrpT1F73chQUXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93HeNprzq6U9Re93IUAXXeNprzq6U9Re93Ntj6emdLaM1gxmvMJmLWUxkdStWpQ2ke56WoZFVVkia1E5Y3fKcvBLAACgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAP/9k=",
  "Floor 2": null,
  "Floor 1": "data:image/png;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/4gHYSUNDX1BST0ZJTEUAAQEAAAHIAAAAAAQwAABtbnRyUkdCIFhZWiAH4AABAAEAAAAAAABhY3NwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAA9tYAAQAAAADTLQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAlkZXNjAAAA8AAAACRyWFlaAAABFAAAABRnWFlaAAABKAAAABRiWFlaAAABPAAAABR3dHB0AAABUAAAABRyVFJDAAABZAAAAChnVFJDAAABZAAAAChiVFJDAAABZAAAAChjcHJ0AAABjAAAADxtbHVjAAAAAAAAAAEAAAAMZW5VUwAAAAgAAAAcAHMAUgBHAEJYWVogAAAAAAAAb6IAADj1AAADkFhZWiAAAAAAAABimQAAt4UAABjaWFlaIAAAAAAAACSgAAAPhAAAts9YWVogAAAAAAAA9tYAAQAAAADTLXBhcmEAAAAAAAQAAAACZmYAAPKnAAANWQAAE9AAAApbAAAAAAAAAABtbHVjAAAAAAAAAAEAAAAMZW5VUwAAACAAAAAcAEcAbwBvAGcAbABlACAASQBuAGMALgAgADIAMAAxADb/2wBDAAUDBAQEAwUEBAQFBQUGBwwIBwcHBw8LCwkMEQ8SEhEPERETFhwXExQaFRERGCEYGh0dHx8fExciJCIeJBweHx7/2wBDAQUFBQcGBw4ICA4eFBEUHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh4eHh7/wAARCAM0BOgDASIAAhEBAxEB/8QAHAABAQADAQEBAQAAAAAAAAAAAAYDBQcECAIB/8QAWxAAAQQBAgIDCQ0GAggCCAQHAAECAwQFBhEHEhMUIRciMVZ1k5XR1BUWIzU2N0FUV3Oxs7QIMlFVltNCYSQlM0NxgZShUpE0REVTYnJ0pViEkqKyJ2NlgoOj/8QAGgEBAQEBAQEBAAAAAAAAAAAAAAECAwQFBv/EADYRAQACAQIDBQgCAgIBBQEAAAABEQIhMQNB8AQSUWGhBXGBkbHB0eEi8RMyM0IUFSNSYnLS/9oADAMBAAIRAxEAPwD5xAB6HEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/UTHSSNjYm7nKjWp/FVA/IOjZrhpiMNl7eJyfE7Ste7TmdBYiWG6qse1dlTdIFRdlT6Dx+8bTX2q6U8xe9nJZSFBde8bTX2q6U8xe9nHvG019qulPMXvZy2IUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce8bTX2q6U8xe9nFiFBde8bTX2q6U8xe9nHvG019qulPMXvZxYhQXXvG019qulPMXvZx7xtNfarpTzF72cWIUF17xtNfarpTzF72ce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qP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqHcp4meIWo/R0nqJPr1363Y84o69d+t2POKBWdyniZ4haj9HSeodyniZ4haj9HSeok+vXfrdjzijr1363Y84oFZ3KeJniFqP0dJ6h3KeJniFqP0dJ6iT69d+t2POKOvXfrdjzigVncp4meIWo/R0nqNnxkxeRw2B0DjMtRsUbsGBekteeNWSMVbtlU3Re1OxUX/AJkB1679bsecUxyyyzO5pZHyORNt3OVV2IPwACgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA/9k=",
};

var knnLogs = [];
var psoLogs = [];
var charts = {};
var psoIterAvg  = [];
var psoIterAvgM = [];

// ── TABS ──
document.querySelectorAll('.nav-tab').forEach(function(btn){
  btn.addEventListener('click',function(){
    // ปุ่ม formula ไม่ใช่ tab — ข้ามไป
    if(btn.id === 'btn-show-formula') return;
    document.querySelectorAll('.nav-tab').forEach(function(b){
      if(b.id !== 'btn-show-formula') b.classList.remove('active');
    });
    btn.classList.add('active');
    var id = btn.dataset.tab;
    document.querySelectorAll('.tab-panel').forEach(function(p){p.classList.remove('active');});
    document.getElementById('tab-'+id).classList.add('active');
    if(id==='floormap') renderMaps();
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

// ── FILE UPLOAD ──
(function(){
  var dz = document.getElementById('drop-zone');
  var fi = document.getElementById('file-input');

  // File selected via dialog
  fi.addEventListener('change', function(){
    if(fi.files && fi.files.length > 0) processFile(fi.files[0]);
  });

  // Drag & drop
  dz.addEventListener('dragover',  function(e){ e.preventDefault(); e.stopPropagation(); dz.classList.add('drag-over'); });
  dz.addEventListener('dragleave', function(e){ e.preventDefault(); dz.classList.remove('drag-over'); });
  dz.addEventListener('drop', function(e){
    e.preventDefault(); e.stopPropagation();
    dz.classList.remove('drag-over');
    var files = e.dataTransfer ? e.dataTransfer.files : null;
    if(files && files.length > 0) processFile(files[0]);
  });

  function processFile(file){
    if(!file) return;

    // ตรวจสอบว่า XLSX โหลดแล้วหรือยัง
    if(typeof XLSX === 'undefined'){
      setStatus('error', '⚠️ ไม่พบ XLSX library — กรุณาเปิดผ่าน web server หรือเชื่อมต่ออินเตอร์เน็ต');
      document.getElementById('drop-zone').innerHTML +=
        '<div style="margin-top:12px;padding:10px;background:rgba(239,68,68,0.15);border:1px solid rgba(239,68,68,0.4);border-radius:8px;font-size:12px;color:#fca5a5;">'+
        '⚠️ XLSX library โหลดไม่ได้<br>'+
        'วิธีแก้: เปิดไฟล์ผ่าน VS Code Live Server หรือ Python HTTP Server<br>'+
        '<code style="font-size:10px;color:#94a3b8;">python -m http.server 8080</code></div>';
      return;
    }

    setStatus('running', 'กำลังอ่านไฟล์: ' + file.name);
    var reader = new FileReader();
    reader.onload = function(ev){
      try{
        var data = ev.target.result;
        var wb   = XLSX.read(new Uint8Array(data), {type:'array'});
        if(!wb.SheetNames.length){ setStatus('error','ไม่พบ Sheet ในไฟล์'); return; }
        var ws   = wb.Sheets[wb.SheetNames[0]];
        var rows = XLSX.utils.sheet_to_json(ws, {header:1, defval:''});
        if(rows.length < 2){ setStatus('error','ข้อมูลในไฟล์น้อยเกินไป ('+rows.length+' rows)'); return; }
        console.log('File loaded:', file.name, '| Rows:', rows.length);
        runKNN(rows);
      } catch(err){
        setStatus('error', 'อ่านไฟล์ไม่ได้: ' + err.message);
        console.error('File read error:', err);
      }
    };
    reader.onerror = function(e){
      setStatus('error', 'FileReader error: ' + (e.target.error ? e.target.error.message : 'unknown'));
    };
    reader.readAsArrayBuffer(file);
  }

  window.handleFileFromDrop = processFile;

  // แสดง warning ถ้า XLSX ไม่โหลด
  setTimeout(function(){
    if(typeof XLSX === 'undefined'){
      setStatus('error','XLSX library ไม่โหลด — ต้องเปิดผ่าน web server');
      var warn = document.getElementById('xlsx-warning');
      if(warn) warn.style.display = 'block';
      var dz2 = document.getElementById('drop-zone');
      if(dz2) dz2.style.borderColor = 'rgba(239,68,68,0.6)';
    } else {
      console.log('✓ XLSX library ready | Chart.js:', typeof Chart !== 'undefined' ? 'ready' : 'missing');
    }
  }, 800);
})();

// ══ KNN ENGINE — ใช้ข้อมูลจากไฟล์ Excel คำนวณ Avg รวม 2 วิธี ══
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
    // Avg รวม แยกชั้น (ใช้ข้อมูลจากไฟล์ — เหมือนเดิมก่อนแก้ไข)
    var mD={},rD={};
    FLOORS.forEach(function(f){mD[f]=[];rD[f]=[];});
    data.forEach(function(t){
      var tf=t.floor;
      if(!mD[tf])return;
      mD[tf].push(euclid3(mobile.meas,t.meas));
      rD[tf].push(euclid3(mobile.rssi,t.rssi));
    });
    var pMA=predAvg(mD);
    var pRA=predAvg(rD);
    function res(p){return p===af?'ถูก':'ผิด→ชั้น '+p.replace('Floor ','');}
    function gap(p){return Math.abs(parseInt(af.replace('Floor ',''))-parseInt(p.replace('Floor ','')));}
    knnLogs.push({
      floor:af, id:mobile.id,
      mAvg:res(pMA), gMA:gap(pMA),
      rAvg:res(pRA), gRA:gap(pRA),
      mTop3:res(pMA), gMT3:gap(pMA),
      rTop3:res(pRA), gRT3:gap(pRA)
    });
  });

  renderKNNAll();
  setStatus('done','KNN เสร็จ — '+knnLogs.length+' Particles');
}

// Avg รวม: เฉลี่ยทุกตัวในชั้น ชั้นที่ avg ต่ำสุด = ทำนาย
function predAvg(dm){
  var s={};
  Object.keys(dm).forEach(function(fl){
    var d=dm[fl];
    s[fl]=d.length?d.reduce(function(a,b){return a+b;},0)/d.length:999;
  });
  return Object.keys(s).reduce(function(a,b){return s[a]<s[b]?a:b;});
}

function euclid3(a,b){return Math.sqrt((a[0]-b[0])**2+(a[1]-b[1])**2+(a[2]-b[2])**2);}


function renderKNNAll(){
  renderOverview();renderDeviation();renderCompare();renderKNNTable();
  ['ov','dev','cmp','tbl'].forEach(function(k){
    var el=document.getElementById(k+'-empty');
    var el2=document.getElementById(k+'-content');
    if(el) el.classList.add('hidden');
    if(el2) el2.classList.remove('hidden');
  });
  var dz2 = document.getElementById('drop-zone');
  if(dz2) dz2.style.display='none';
  renderCDF();
  renderMaps();
}

// ── Overview charts ──
function renderOverview(){
  var t=knnLogs.length;
  var ok=function(k){return knnLogs.filter(function(l){return l[k]==='ถูก';}).length;};
  var okMA=ok('mAvg'), okRA=ok('rAvg');
  var pct=function(v){return ((v/t)*100).toFixed(1)+'%';};
  document.getElementById('acc-m-avg').textContent=pct(okMA);
  document.getElementById('acc-r-avg').textContent=pct(okRA);
  try{document.getElementById('acc-m-top3').textContent=pct(okMA);}catch(e){}
  try{document.getElementById('acc-r-top3').textContent=pct(okRA);}catch(e){}

  var GC='rgba(255,255,255,0.04)', TC='#6b7a99';
  var AMBER='rgba(245,158,11,0.85)', ROSE='rgba(244,63,94,0.85)';
  var AMBER_S='rgba(245,158,11,0.5)', ROSE_S='rgba(244,63,94,0.5)';

  // Bar: accuracy 2 วิธี
  mkChart('accChart',{type:'bar',data:{
    labels:['Meas. AVG รวม','RSSI Predict AVG รวม'],
    datasets:[{data:[+(okMA/t*100).toFixed(1),+(okRA/t*100).toFixed(1)],
      backgroundColor:[AMBER,ROSE],borderColor:['rgba(245,158,11,1)','rgba(244,63,94,1)'],
      borderWidth:1,borderRadius:6,borderSkipped:false}]},
    options:{responsive:true,maintainAspectRatio:false,
      plugins:{legend:{display:false},tooltip:{callbacks:{label:function(c){return c.parsed.y.toFixed(1)+'%';}}}},
      scales:{y:{min:0,max:100,grid:{color:GC},ticks:{color:TC,callback:function(v){return v+'%';}}},
        x:{grid:{display:false},ticks:{color:TC,font:{size:10}}}}}
  });

  // Bar: errors per floor 2 วิธี
  var errF={};FLOORS.forEach(function(f){errF[f]={ma:0,ra:0};});
  knnLogs.forEach(function(l){
    if(!errF[l.floor])return;
    if(l.mAvg!=='ถูก')errF[l.floor].ma++;
    if(l.rAvg!=='ถูก')errF[l.floor].ra++;
  });
  mkChart('errChart',{type:'bar',data:{labels:FLOORS,datasets:[
    {label:'Meas.Avg',data:FLOORS.map(function(f){return errF[f].ma;}),
     backgroundColor:AMBER_S,borderColor:'rgba(245,158,11,0.9)',borderWidth:1,borderRadius:4},
    {label:'RSSI Avg',data:FLOORS.map(function(f){return errF[f].ra;}),
     backgroundColor:ROSE_S,borderColor:'rgba(244,63,94,0.9)',borderWidth:1,borderRadius:4}
  ]},options:{responsive:true,maintainAspectRatio:false,
    plugins:{legend:{display:true,labels:{color:'#9ca3af',font:{size:10},boxWidth:10,padding:10}}},
    scales:{y:{beginAtZero:true,grid:{color:GC},ticks:{color:TC,stepSize:1}},
      x:{grid:{display:false},ticks:{color:TC}}}}
  });

  // Line: % ถูกต้องแยกชั้น
  function floorPct(key,fl){
    var sub=knnLogs.filter(function(l){return l.floor===fl;});
    if(!sub.length)return 0;
    return +((sub.filter(function(l){return l[key]==='ถูก';}).length/sub.length)*100).toFixed(1);
  }
  function lineDsOpt(color,dash){
    return{borderColor:color,backgroundColor:'transparent',
      pointBackgroundColor:color,pointBorderColor:'var(--bg2)',
      pointBorderWidth:1.5,pointRadius:5,pointHoverRadius:7,
      fill:false,tension:.4,borderWidth:2.5,borderDash:dash||[]};
  }
  mkChart('floorSumChart',{type:'line',data:{labels:FLOORS,datasets:[
    Object.assign({label:'Meas.Avg',data:FLOORS.map(function(f){return floorPct('mAvg',f);})},
      lineDsOpt('rgba(245,158,11,0.9)')),
    Object.assign({label:'RSSI Avg',data:FLOORS.map(function(f){return floorPct('rAvg',f);})},
      lineDsOpt('rgba(244,63,94,0.9)',[6,3]))
  ]},options:{responsive:true,maintainAspectRatio:false,
    plugins:{legend:{display:true,labels:{color:'#9ca3af',font:{size:11},boxWidth:12,padding:14}},
      tooltip:{callbacks:{label:function(c){return c.dataset.label+': '+c.parsed.y.toFixed(1)+'%';}}}},
    scales:{y:{min:0,max:100,grid:{color:GC},ticks:{color:TC,callback:function(v){return v+'%';}}},
      x:{grid:{color:GC},ticks:{color:TC}}},
    interaction:{mode:'index',intersect:false}}
  });
}


// ── Deviation ──
function renderDeviation(){
  var dc = document.getElementById('dev-cards');
  dc.innerHTML = '';
  var allCounts = [];

  // ใช้แค่ 2 วิธี: Meas.Avg และ RSSI.Avg (จาก KNN logs)
  var methods = [
    {key:'gMA',  label:'Meas.Avg',  color:C[1], resultKey:'mAvg'},
    {key:'gRA',  label:'RSSI Avg',  color:C[3], resultKey:'rAvg'}
  ];

  // สร้าง lookup ว่า PR แต่ละตัวทำนายชั้นไหน (สำหรับ popup)
  function getWrongPRs(gapVal, methodResultKey){
    return knnLogs.filter(function(l){
      var gapKey = methodResultKey==='mAvg' ? 'gMA' : 'gRA';
      return l[gapKey] === gapVal && gapVal > 0;
    }).map(function(l){
      var predRaw = l[methodResultKey]; // เช่น "ผิด→ชั้น 3"
      var predicted = predRaw.replace('ผิด→','');
      return {id:l.id, actualFloor:l.floor, predicted:predicted, gap:gapVal};
    });
  }

  methods.forEach(function(m){
    var d = {0:0,1:0,2:0,3:0};
    knnLogs.forEach(function(l){
      var v = l[m.key];
      if(d[v]!==undefined) d[v]++;
    });
    allCounts.push(d);

    var maxV = Math.max(d[0],d[1],d[2],d[3]) || 1;
    var card = document.createElement('div');
    card.className = 'dev-card';

    var rows = [
      {label:'✅ ถูกชั้น', gap:0, color:'#22c55e'},
      {label:'+1 ชั้น',    gap:1, color:C[1]},
      {label:'+2 ชั้น',    gap:2, color:C[3]},
      {label:'+3 ชั้น',    gap:3, color:'#ef4444'}
    ].map(function(r){
      var cnt = d[r.gap];
      var clickable = r.gap > 0 && cnt > 0;
      var countEl;
      if(clickable){
        // store data in window registry to avoid inline JSON quote issues
        var key = 'dev_'+m.resultKey+'_gap'+r.gap;
        window._devData = window._devData || {};
        window._devData[key] = getWrongPRs(r.gap, m.resultKey);
        var popupTitle = m.label+' — คลาดเคลื่อน '+r.gap+' ชั้น ('+cnt+' PR)';
        window._devData[key+'_title'] = popupTitle;
        countEl = '<span class="dev-count" style="color:'+r.color+
          ';cursor:pointer;text-decoration:underline dotted;border-bottom:1px dotted '+r.color+';" '+
          'onclick="window.showDevPopup(window._devData[\''+key+'_title\'],window._devData[\''+key+'\'])">'+
          cnt+'</span>';
      } else {
        countEl = '<span class="dev-count" style="color:'+r.color+'">'+cnt+'</span>';
      }
      return '<div class="dev-row">'+
        '<span class="dev-label">'+r.label+'</span>'+
        '<div class="dev-bar-wrap"><div class="dev-bar" style="width:'+((cnt/maxV)*100).toFixed(0)+'%;background:'+r.color+'"></div></div>'+
        countEl+'</div>';
    }).join('');

    card.innerHTML = '<div class="dev-card-title" style="color:'+m.color+'">'+m.label+'</div>'+
      '<div class="dev-rows">'+rows+'</div>';
    dc.appendChild(card);
  });

  // Bar chart เฉพาะ 2 วิธี
  mkChart('devChart',{
    type:'bar',
    data:{
      labels:['0 ชั้น (ถูก)','+1 ชั้น','+2 ชั้น','+3 ชั้น'],
      datasets:[
        {label:'Meas. Avg',data:[allCounts[0][0],allCounts[0][1],allCounts[0][2],allCounts[0][3]],backgroundColor:C[1],borderRadius:5},
        {label:'RSSI Avg', data:[allCounts[1][0],allCounts[1][1],allCounts[1][2],allCounts[1][3]],backgroundColor:C[3],borderRadius:5}
      ]
    },
    options:{responsive:true,maintainAspectRatio:false,
      plugins:{legend:{display:true,labels:{color:TICK_C,font:{size:11},boxWidth:12,padding:12}}},
      scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C,stepSize:1}},
              x:{grid:{display:false},ticks:{color:TICK_C}}}}
  });
}

// ── CDF Chart ──────────────────────────────────────────────────
// ข้อมูลจากไฟล์ CDF_Indoor_24-5-69.xlsx (hardcoded)
// และคำนวณ CDF แบบ realtime จาก knnLogs ด้วย
function renderCDF(){
  var t = knnLogs.length || 55;
  var GC = 'rgba(255,255,255,0.04)', TC = '#6b7a99';

  // คำนวณ CDF จาก knnLogs ถ้ามีข้อมูล
  var mCounts={0:0,1:0,2:0,3:0}, rCounts={0:0,1:0,2:0,3:0};
  knnLogs.forEach(function(l){
    var gm=Math.min(l.gMA||0,3); var gr=Math.min(l.gRA||0,3);
    mCounts[gm]++; rCounts[gr]++;
  });

  // ถ้าไม่มี knnLogs ใช้ค่าจากไฟล์ Excel โดยตรง
  if(!knnLogs.length){
    mCounts={0:38,1:8,2:7,3:2};
    rCounts={0:45,1:10,2:0,3:0};
    t=55;
  }

  // คำนวณ CDF = สะสม ÷ total
  function makeCDF(counts){
    var cdf=[], cum=0;
    [0,1,2,3].forEach(function(k){ cum+=counts[k]; cdf.push(+(cum/t*100).toFixed(2)); });
    return cdf;
  }
  var mCDF = makeCDF(mCounts);
  var rCDF = makeCDF(rCounts);

  mkChart('cdfChart',{
    type:'line',
    data:{
      labels:['0 ชั้น (ถูก)','≤ 1 ชั้น','≤ 2 ชั้น','≤ 3 ชั้น'],
      datasets:[
        {label:'Meas.Avg',data:mCDF,
         borderColor:'rgba(245,158,11,0.9)',backgroundColor:'transparent',
         pointBackgroundColor:'rgba(245,158,11,0.9)',
         pointBorderColor:'var(--bg2)',pointBorderWidth:1.5,
         pointRadius:6,pointHoverRadius:8,
         fill:false,tension:.3,borderWidth:2.5},
        {label:'RSSI Avg',data:rCDF,
         borderColor:'rgba(244,63,94,0.9)',backgroundColor:'transparent',
         pointBackgroundColor:'rgba(244,63,94,0.9)',
         pointBorderColor:'var(--bg2)',pointBorderWidth:1.5,
         pointRadius:6,pointHoverRadius:8,
         fill:false,tension:.3,borderWidth:2.5,borderDash:[6,3]}
      ]
    },
    options:{
      responsive:true, maintainAspectRatio:false,
      plugins:{
        legend:{display:true,labels:{color:'#9ca3af',font:{size:11},boxWidth:12,padding:14}},
        tooltip:{
          callbacks:{
            label:function(c){return c.dataset.label+': '+c.parsed.y.toFixed(2)+'%';},
            afterLabel:function(c){
              var counts = c.dataset.label.includes('Meas') ? mCounts : rCounts;
              var k = c.dataIndex;
              return '('+counts[k]+' PR)';
            }
          }
        }
      },
      scales:{
        y:{min:0,max:100,grid:{color:GC},
          ticks:{color:TC,callback:function(v){return v+'%';}},
          title:{display:true,text:'ความน่าจะเป็นสะสม (%)',color:TC,font:{size:11}}},
        x:{grid:{color:GC},ticks:{color:TC},
          title:{display:true,text:'Floor Error (ชั้น)',color:TC,font:{size:11}}}
      },
      interaction:{mode:'index',intersect:false}
    }
  });
}
function renderCompare(){
  function floorPct(key,fl){
    var sub=knnLogs.filter(function(l){return l.floor===fl;});
    if(!sub.length)return 0;
    return +((sub.filter(function(l){return l[key]==='ถูก';}).length/sub.length)*100).toFixed(1);
  }
  function lineDsOpt(color,dash){
    return{borderColor:color,backgroundColor:'transparent',
      pointBackgroundColor:color,pointBorderColor:'var(--bg2)',
      pointBorderWidth:1.5,pointRadius:5,pointHoverRadius:7,
      fill:false,tension:.4,borderWidth:2.5,borderDash:dash||[]};
  }
  var lOpts={responsive:true,maintainAspectRatio:false,
    plugins:{legend:{display:true,labels:{color:'#9ca3af',font:{size:11},boxWidth:12,padding:14}},
      tooltip:{callbacks:{label:function(c){return c.dataset.label+': '+c.parsed.y.toFixed(1)+'%';}}}},
    scales:{y:{min:0,max:100,grid:{color:'rgba(255,255,255,0.04)'},
      ticks:{color:'#6b7a99',callback:function(v){return v+'%';}}},
      x:{grid:{color:'rgba(255,255,255,0.04)'},ticks:{color:'#6b7a99'}}},
    interaction:{mode:'index',intersect:false}};

  // เฉพาะ Avg ทั้ง 2 วิธี
  mkChart('lineAvgChart',{type:'line',data:{labels:FLOORS,datasets:[
    Object.assign({label:'Meas.Avg',data:FLOORS.map(function(f){return floorPct('mAvg',f);})},
      lineDsOpt('rgba(245,158,11,0.9)')),
    Object.assign({label:'RSSI Avg',data:FLOORS.map(function(f){return floorPct('rAvg',f);})},
      lineDsOpt('rgba(244,63,94,0.9)',[6,3]))
  ]},options:lOpts});
  // compat charts (hidden)
  try{mkChart('lineTop3Chart',{type:'line',data:{labels:FLOORS,datasets:[]},options:{responsive:true,maintainAspectRatio:false}});}catch(e){}
  try{mkChart('lineAllChart',{type:'line',data:{labels:FLOORS,datasets:[]},options:{responsive:true,maintainAspectRatio:false}});}catch(e){}
}

// ── KNN Table — แสดง Avg 2 วิธี + PSO result รวมในตารางเดียว ──
function renderKNNTable(){
  document.getElementById('total-records').textContent = knnLogs.length+' Particles';
  var fp = {'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
  var tbody = document.getElementById('table-body');
  tbody.innerHTML = '';
  knnLogs.forEach(function(log){
    var fn  = log.floor.replace('Floor ','');
    var maxG = Math.max(log.gMA||0, log.gRA||0);
    var gc  = maxG===0?'var(--ok)':maxG===1?'var(--amber)':'var(--err)';

    function gapLabel(g){
      return g===0
        ? '<span style="font-family:var(--mono);font-size:11px;color:var(--ok)">ถูก</span>'
        : '<span style="font-family:var(--mono);font-size:11px;color:'+(g===1?'var(--amber)':'var(--err)')+'">'+g+' ชั้น</span>';
    }

    // หา PSO log ที่ตรงกัน
    var psoLog = psoLogs.find(function(p){return p.id===log.id;});

    function knnBadge(v){
      return v==='ถูก'
        ? '<span class="badge badge-ok">ถูก</span>'
        : '<span class="badge badge-err">'+v.replace('ผิด→','')+'</span>';
    }
    var psoMBadge = psoLog
      ? '<span class="badge '+(psoLog.predFloorM===log.floor?'badge-ok':'badge-err')+'">'+
        psoLog.predFloorM+(psoLog.predFloorM!==log.floor?' ✗':'')+'</span>'
      : '<span style="color:var(--muted2)">—</span>';
    var psoRBadge = psoLog
      ? '<span class="badge '+(psoLog.predFloorR===log.floor?'badge-ok':'badge-err')+'">'+
        psoLog.predFloorR+(psoLog.predFloorR!==log.floor?' ✗':'')+'</span>'
      : '<span style="color:var(--muted2)">—</span>';
    var ecM = psoLog ? (psoLog.xyErrM<=3?'var(--ok)':psoLog.xyErrM<=6?'var(--amber)':'var(--err)') : 'var(--muted2)';
    var ecR = psoLog ? (psoLog.xyErrR<=3?'var(--ok)':psoLog.xyErrR<=6?'var(--amber)':'var(--err)') : 'var(--muted2)';
    var errM = psoLog ? '<span style="font-family:var(--mono);font-size:10px;color:'+ecM+'">'+psoLog.xyErrM.toFixed(2)+'m</span>' : '—';
    var errR = psoLog ? '<span style="font-family:var(--mono);font-size:10px;color:'+ecR+'">'+psoLog.xyErrR.toFixed(2)+'m</span>' : '—';
    var gbM  = psoLog ? '<span style="color:var(--muted);font-size:10px">'+psoLog.gxM+','+psoLog.gyM+'</span>' : '—';
    var gbR  = psoLog ? '<span style="color:var(--muted);font-size:10px">'+psoLog.gxR+','+psoLog.gyR+'</span>' : '—';

    tbody.innerHTML +=
      '<tr>'+
      '<td><span class="floor-pill '+(fp[fn]||'')+'">'+log.floor+'</span></td>'+
      '<td style="color:var(--muted)">'+log.id+'</td>'+
      // Measurement Avg
      '<td>'+knnBadge(log.mAvg)+'</td>'+
      '<td>'+gbM+'</td>'+
      '<td>'+errM+'</td>'+
      // RSSI Predict Avg
      '<td>'+knnBadge(log.rAvg)+'</td>'+
      '<td>'+gbR+'</td>'+
      '<td>'+errR+'</td>'+
      '<td>'+gapLabel(log.gMA||0)+'</td>'+
      '<td>'+gapLabel(log.gRA||0)+'</td>'+
      '</tr>';
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
  document.getElementById('pso-progress').textContent = 'กำลัง pre-compute PR reliability cache...';
  document.getElementById('pso-empty').classList.add('hidden');

  setTimeout(function(){
    // Reset cache ก่อนเริ่มใหม่
    _prKnnCache = null;
    // Build cache ครั้งเดียว (55 PR × 55 PR = 3025 operations)
    var cache = buildPrKnnCache();
    // แสดงสรุป cache
    var unreliable = Object.values(cache).filter(function(v){return v.deviation>=2;}).length;
    document.getElementById('pso-progress').textContent =
      'Cache พร้อม: PR ที่คลาดเคลื่อน>2ชั้น = '+unreliable+'/55 | PLd0=-53 · k='+K+' · n='+n+' · runs='+R;

    psoLogs = [];
    psoIterAvg  = Array(K).fill(0);
    psoIterAvgM = Array(K).fill(0);

    PR_DB.forEach(function(mobile, idx){
      document.getElementById('pso-progress').textContent =
        'คำนวณ '+(idx+1)+'/'+PR_DB.length+'... ['+mobile.id+']';

      // KNN Avg รวม — ทำนายชั้น แล้ว PSO รันในชั้นที่ KNN ทำนาย
      var knnM = knnPredictFloor(mobile.mRSSI, mobile.id, null, 'meas');
      var knnR = knnPredictFloor(mobile.pRSSI, mobile.id, null, 'pred');

      // PSO เฉพาะใน floor ที่ KNN ทำนาย โดยใช้แค่ PR ในชั้นนั้น
      var rM = psoRunHybrid(mobile.mRSSI, mobile, knnM, n, K, wmin, wmax, c1, c2, R);
      var rR = psoRunHybrid(mobile.pRSSI, mobile, knnR, n, K, wmin, wmax, c1, c2, R);

      rM.iterFitness.forEach(function(v,i){ psoIterAvgM[i] += v; });
      rR.iterFitness.forEach(function(v,i){ psoIterAvg[i]  += v; });

      var xyErrM = Math.sqrt((rM.gx-mobile.x)**2 + (rM.gy-mobile.y)**2);
      var xyErrR = Math.sqrt((rR.gx-mobile.x)**2 + (rR.gy-mobile.y)**2);

      psoLogs.push({
        floor:mobile.floor, id:mobile.id,
        px:mobile.x, py:mobile.y,
        knnFloorM:knnM.knnFloor, seedIdsM:rM.seedIds, filteredCountM:knnM.filteredCount||0,
        knnFloorR:knnR.knnFloor, seedIdsR:rR.seedIds, filteredCountR:knnR.filteredCount||0,
        predFloorM:rM.predFloor,
        gxM:rM.gx, gyM:rM.gy, costM:rM.gcost, xyErrM:xyErrM,
        predFloorR:rR.predFloor,
        gxR:rR.gx, gyR:rR.gy, costR:rR.gcost, xyErrR:xyErrR,
        gapM: Math.abs(parseInt(mobile.floor.replace('Floor ',''))-parseInt(knnM.knnFloor.replace('Floor ',''))),
        gapR: Math.abs(parseInt(mobile.floor.replace('Floor ',''))-parseInt(knnR.knnFloor.replace('Floor ','')))
      });
    });

    psoIterAvg  = psoIterAvg.map(function(v){ return +(v/PR_DB.length).toFixed(3); });
    psoIterAvgM = psoIterAvgM.map(function(v){ return +(v/PR_DB.length).toFixed(3); });
    renderPSOResults();
    setStatus('done','Hybrid KNN→PSO เสร็จ — 55 Particles');
    document.getElementById('pso-progress').textContent = 'เสร็จสิ้น ✓';
    document.getElementById('btn-export-pso').style.display = '';
  }, 50);
}

// ── FORMULA MODAL ──────────────────────────────────────────────
document.getElementById('btn-show-formula').addEventListener('click', function(){
  renderFormulaModal();
  document.getElementById('formula-modal').style.display   = 'block';
  document.getElementById('formula-overlay').style.display = 'block';
});

function renderFormulaModal(){
  // ดึงค่า realtime จาก UI
  var n    = document.getElementById('pso-n').value    || 100;
  var K    = document.getElementById('pso-k').value    || 30;
  var R    = document.getElementById('pso-runs').value || 5;
  var wmin = document.getElementById('pso-wmin').value || 0.4;
  var wmax = document.getElementById('pso-wmax').value || 0.9;
  var c1   = document.getElementById('pso-c1').value   || 2.0;
  var c2   = document.getElementById('pso-c2').value   || 2.0;

  // ตัวอย่าง: คำนวณจาก PR1 Floor 4
  var exPR  = PR_DB.find(function(p){return p.id==='PR1';}) || PR_DB[0];
  var exAn  = ANCHORS[0];
  var pz4   = getZ('Floor 4');
  var d_ex  = Math.sqrt((exPR.x-exAn.x)**2+(exPR.y-exAn.y)**2+pz4**2);
  var pred4 = predictRSSI(d_ex,'Floor 4');
  var pred3 = predictRSSI(Math.sqrt((exPR.x-exAn.x)**2+(exPR.y-exAn.y)**2+getZ('Floor 3')**2),'Floor 3');

  var h = function(t){ return '<span style="color:#f59e0b;font-weight:700;">'+t+'</span>'; };
  var b = function(t){ return '<span style="color:#60a5fa;">'+t+'</span>'; };
  var g = function(t){ return '<span style="color:#22c55e;">'+t+'</span>'; };
  var m = function(t){ return '<span style="color:#a855f7;">'+t+'</span>'; };
  var code = function(t){ return '<code style="background:rgba(255,255,255,0.06);padding:2px 8px;border-radius:4px;font-family:IBM Plex Mono,monospace;font-size:12px;color:#e2e8f0;">'+t+'</code>'; };
  var eq   = function(t){ return '<div style="background:rgba(96,165,250,0.07);border-left:3px solid #60a5fa;padding:8px 14px;margin:8px 0;border-radius:0 6px 6px 0;font-family:IBM Plex Mono,monospace;font-size:12px;color:#e2e8f0;">'+t+'</div>'; };
  var sec  = function(t){ return '<div style="font-size:14px;font-weight:700;color:#60a5fa;margin:22px 0 10px;border-bottom:1px solid rgba(96,165,250,0.2);padding-bottom:6px;">'+t+'</div>'; };
  var sub  = function(t){ return '<div style="font-size:12px;font-weight:600;color:#94a3b8;margin:14px 0 6px;">'+t+'</div>'; };
  var note = function(t){ return '<div style="font-size:11px;color:#6b7a99;margin-top:4px;">'+t+'</div>'; };

  var html = '';

  // ── STEP 1: Distance ──
  html += sec('STEP 1 — คำนวณระยะทาง 3D จาก Mobile ไป AN แต่ละตัว');
  html += eq('d = √( (x<sub>AN</sub>−x<sub>PR</sub>)² + (y<sub>AN</sub>−y<sub>PR</sub>)² + (z<sub>AN</sub>−z<sub>PR</sub>)² )');
  html += '<div style="margin:8px 0;">โดย z = ระยะแนวตั้งระหว่าง AN (ceiling F4) กับ Mobile แต่ละชั้น:</div>';
  html += '<table style="width:100%;border-collapse:collapse;font-size:12px;font-family:IBM Plex Mono,monospace;margin:8px 0;">';
  ['Floor 4','Floor 3','Floor 2','Floor 1'].forEach(function(fl){
    var pz=getZ(fl);
    html += '<tr style="border-bottom:1px solid rgba(255,255,255,0.05);">';
    html += '<td style="padding:5px 10px;color:#94a3b8;">'+fl+'</td>';
    html += '<td style="padding:5px 10px;">AN height = '+AN_HEIGHT+'m, floor height = '+FLOOR_HEIGHT[fl]+'m</td>';
    html += '<td style="padding:5px 10px;color:#facc15;">z = '+pz.toFixed(2)+' m</td>';
    html += '</tr>';
  });
  html += '</table>';
  html += note('ตัวอย่าง: PR1('+exPR.x+','+exPR.y+') → AN1('+exAn.x+','+exAn.y+') | z(F4)='+pz4.toFixed(2)+'m → d = '+d_ex.toFixed(3)+' m');

  // ── STEP 2: Path Loss Model ──
  html += sec('STEP 2 — คำนวณ RSSI Predict จาก Path Loss Model');
  html += sub('📡 สูตรทั่วไป:');
  html += eq('RSSI<sub>predict</sub> = PLd₀ − 10×η×log₁₀(d) − FAF');
  html += '<table style="width:100%;border-collapse:collapse;font-size:12px;font-family:IBM Plex Mono,monospace;margin:8px 0;">';
  html += '<tr style="background:rgba(255,255,255,0.04);"><th style="padding:6px 10px;text-align:left;color:#94a3b8;">ชั้น</th><th style="padding:6px 10px;color:#94a3b8;">PLd₀</th><th style="padding:6px 10px;color:#94a3b8;">η</th><th style="padding:6px 10px;color:#94a3b8;">FAF</th><th style="padding:6px 10px;color:#94a3b8;">สูตรเต็ม</th></tr>';
  var floorDefs = [
    {fl:'Floor 4',eta:2.0,faf:0},
    {fl:'Floor 3',eta:1.8,faf:5},
    {fl:'Floor 2',eta:1.6,faf:10},
    {fl:'Floor 1',eta:1.4,faf:15},
  ];
  floorDefs.forEach(function(f){
    var fafStr = f.faf>0?'−'+f.faf:'0';
    html += '<tr style="border-bottom:1px solid rgba(255,255,255,0.05);">';
    html += '<td style="padding:5px 10px;color:#a855f7;">'+f.fl+'</td>';
    html += '<td style="padding:5px 10px;color:#f59e0b;">'+PLd0_CALIBRATED+'</td>';
    html += '<td style="padding:5px 10px;color:#22c55e;">'+f.eta+'</td>';
    html += '<td style="padding:5px 10px;color:#ef4444;">'+fafStr+' dBm</td>';
    html += '<td style="padding:5px 10px;color:#e2e8f0;">−53 − 10×'+f.eta+'×log₁₀(d)'+(f.faf?' − '+f.faf:'')+'</td>';
    html += '</tr>';
  });
  html += '</table>';
  html += note('ตัวอย่าง PR1 → AN1 (F4): d='+d_ex.toFixed(3)+'m → RSSI<sub>pred</sub> = −53 − 10×2.0×log₁₀('+d_ex.toFixed(3)+') = <b style="color:#60a5fa;">'+pred4.toFixed(2)+' dBm</b>');

  // ── STEP 3: KNN ──
  html += sec('STEP 3 — KNN Avg รวม: ทำนายชั้น');
  html += sub('วิธีที่ 1 — Measurement (RSSI วัดจริง):');
  html += eq('distance<sub>Meas</sub> = √( Σᵢ(RSSI<sub>Mobile,i</sub> − RSSI<sub>PR,i</sub>)² )  &nbsp;i = AN1,AN2,AN3');
  html += '<div style="margin:6px 0 2px;">เปรียบเทียบ RSSI ที่ '+b('วัดจริง')+" กับ RSSI ที่ "+b('วัดจริงของ PR อื่น')+" ใน PR_DB</div>";
  html += note('score_floor = ค่าเฉลี่ย distance ของ PR ทุกตัวในชั้นนั้น → ชั้นที่ score ต่ำสุด = ชั้นที่ทำนาย');

  html += sub('วิธีที่ 2 — RSSI Predict (diff floor model):');
  html += eq('distance<sub>Pred</sub> = √( Σᵢ(RSSI<sub>Mobile,i</sub> − RSSI<sub>PR_Predict,i</sub>)² )');
  html += '<div style="margin:6px 0 2px;">เปรียบเทียบกับ RSSI ที่ '+m('คำนวณจาก Path Loss Model')+" ของ PR แต่ละตัว</div>";
  html += note('Leave-one-out: Mobile ตัวเองถูกตัดออกจาก reference set');

  // ── STEP 4: Fitness ──
  html += sec('STEP 4 — PSO Fitness Function (RMSE)');
  html += eq('F(x,y) = √( Σᵢ(RSSI<sub>predict,i</sub>(x,y) − RSSI<sub>measured,i</sub>)² / 3 )');
  html += '<div style="margin:8px 0;">PSO หาตำแหน่ง (x,y) ที่ทำให้ F ต่ำที่สุด = RSSI ที่คำนวณใกล้เคียง RSSI ที่วัดมากที่สุด</div>';
  html += sub('Measurement mode: RSSI<sub>measured</sub> = mRSSI (ค่าวัดจริง)');
  html += sub('RSSI Predict mode: RSSI<sub>measured</sub> = pRSSI (ค่า predict จาก model)');

  // ── STEP 5: PSO Update ──
  html += sec('STEP 5 — PSO Velocity & Position Update');
  html += eq('v<sub>k+1</sub> = w·v<sub>k</sub> + c1·r1·(pbest − x<sub>k</sub>) + c2·r2·(gbest − x<sub>k</sub>)');
  html += eq('x<sub>k+1</sub> = x<sub>k</sub> + v<sub>k+1</sub>');

  html += '<table style="width:100%;border-collapse:collapse;font-size:12px;margin:10px 0;">';
  html += '<tr style="background:rgba(255,255,255,0.04);"><th style="padding:6px 10px;text-align:left;color:#94a3b8;">ตัวแปร</th><th style="padding:6px 10px;text-align:left;color:#94a3b8;">ความหมาย</th><th style="padding:6px 10px;text-align:left;color:#facc15;">ค่าปัจจุบัน</th></tr>';
  [
    {v:'w',   d:'Inertia weight — ความเฉื่อยจากความเร็วเดิม',        val:wmin+' → '+wmax+' (ลดทีละน้อยตาม k)'},
    {v:'c1',  d:'Cognitive — ดึงเข้าหาตำแหน่ง best ของตัวเอง',       val:c1},
    {v:'c2',  d:'Social — ดึงเข้าหา gbest ของ swarm ทั้งหมด',         val:c2},
    {v:'r1,r2',d:'Random [0,1] สุ่มใหม่ทุก iteration',               val:'random'},
    {v:'n',   d:'จำนวน particle ใน swarm',                           val:n+' ตัว'},
    {v:'K',   d:'จำนวน iteration',                                    val:K+' รอบ'},
    {v:'R',   d:'จำนวน multi-run (เฉลี่ย gbest)',                      val:R+' ครั้ง'},
  ].forEach(function(row){
    html += '<tr style="border-bottom:1px solid rgba(255,255,255,0.04);">';
    html += '<td style="padding:5px 10px;font-family:IBM Plex Mono,monospace;color:#a855f7;">'+row.v+'</td>';
    html += '<td style="padding:5px 10px;color:#c8d4e8;">'+row.d+'</td>';
    html += '<td style="padding:5px 10px;color:#facc15;">'+row.val+'</td>';
    html += '</tr>';
  });
  html += '</table>';

  // ── STEP 6: Grid Search ──
  html += sec('STEP 6 — Local Grid Search Refinement');
  html += eq('gbest<sub>final</sub> = argmin F(x,y)  &nbsp; x ∈ [gbest.x±2m],  y ∈ [gbest.y±2m],  step=0.2m');
  html += note('หลัง PSO จบ ทำ exhaustive search รอบ gbest ±2m เพื่อ refine ตำแหน่ง');

  // ── STEP 7: Error ──
  html += sec('STEP 7 — XY Error (ระยะคลาดเคลื่อน)');
  html += eq('Error = √( (x<sub>gbest</sub> − x<sub>true</sub>)² + (y<sub>gbest</sub> − y<sub>true</sub>)² )  &nbsp;(เมตร)');

  // ── Realtime summary ──
  html += sec('📊 Summary — ค่าปัจจุบันที่ใช้ในการคำนวณ');
  if(psoLogs.length){
    var t=psoLogs.length;
    var avgM=(psoLogs.reduce(function(s,l){return s+l.xyErrM;},0)/t).toFixed(2);
    var avgR=(psoLogs.reduce(function(s,l){return s+l.xyErrR;},0)/t).toFixed(2);
    var okM=psoLogs.filter(function(l){return l.predFloorM===l.floor;}).length;
    var okR=psoLogs.filter(function(l){return l.predFloorR===l.floor;}).length;
    html += '<div style="display:grid;grid-template-columns:repeat(2,1fr);gap:12px;margin-top:8px;">';
    html += '<div style="background:rgba(245,158,11,0.08);border:1px solid rgba(245,158,11,0.2);border-radius:8px;padding:12px;">';
    html += '<div style="font-size:11px;color:#f59e0b;font-weight:700;margin-bottom:8px;">Measurement (RSSI วัดจริง)</div>';
    html += '<div style="font-family:IBM Plex Mono,monospace;font-size:12px;line-height:2;">Floor Accuracy: <b style="color:#22c55e;">'+((okM/t)*100).toFixed(1)+'%</b><br>Avg XY Error: <b style="color:#facc15;">'+avgM+' m</b></div>';
    html += '</div>';
    html += '<div style="background:rgba(244,63,94,0.08);border:1px solid rgba(244,63,94,0.2);border-radius:8px;padding:12px;">';
    html += '<div style="font-size:11px;color:#f43f5e;font-weight:700;margin-bottom:8px;">RSSI Predict (diff floor model)</div>';
    html += '<div style="font-family:IBM Plex Mono,monospace;font-size:12px;line-height:2;">Floor Accuracy: <b style="color:#22c55e;">'+((okR/t)*100).toFixed(1)+'%</b><br>Avg XY Error: <b style="color:#facc15;">'+avgR+' m</b></div>';
    html += '</div></div>';
  } else {
    html += '<div style="color:#6b7a99;font-size:12px;">— ยังไม่มีผล PSO — กด "รัน PSO" ก่อน —</div>';
  }

  document.getElementById('formula-body').innerHTML = html;
}
// AN ติด ceiling ชั้น 4, สูง 1.5m จากพื้น F4
// z = ระยะ vertical จาก Mobile PR ถึง AN
var AN_HEIGHT    = 10.5; // m จากพื้น F1
var FLOOR_HEIGHT = {'Floor 4':9.0,'Floor 3':6.0,'Floor 2':3.0,'Floor 1':0.0};
function getZ(fl){ return AN_HEIGHT - (FLOOR_HEIGHT[fl]||0); }
// F4→z=1.5m, F3→z=4.5m, F2→z=7.5m, F1→z=10.5m

// ── Path Loss Model (ตามสูตรที่ calibrate จริง) ──
// F4: RSSI = -53 - 10*(2.0)*log10(d)
// F3: RSSI = -53 - 10*(1.8)*log10(d) - 5
// F2: RSSI = -53 - 10*(1.6)*log10(d) - 10
// F1: RSSI = -53 - 10*(1.4)*log10(d) - 15
var PLd0_CALIBRATED = -53.0;
var ETA_FLOOR = {'Floor 4':2.0, 'Floor 3':1.8, 'Floor 2':1.6, 'Floor 1':1.4};
var FAF_FLOOR = {'Floor 4':0,   'Floor 3':5,   'Floor 2':10,  'Floor 1':15};

var OPEN_ATRIUM_IDS = new Set(['PR40','PR41','PR42','PR43','PR44','PR45','PR46','PR47','PR55']);

function floorsThrough(prId, targetFloor){
  if(OPEN_ATRIUM_IDS.has(prId)) return 0;
  var fn = parseInt(targetFloor.replace('Floor ',''));
  return Math.max(0, 4-fn);
}

function dist3D(px, py, pz, ax, ay){
  return Math.max(Math.sqrt((px-ax)**2+(py-ay)**2+pz**2), 0.1);
}

// ── Predict RSSI ตาม model จริง ──
function predictRSSI(d, floor){
  var eta = ETA_FLOOR[floor] || 2.0;
  var faf = FAF_FLOOR[floor] || 0;
  return PLd0_CALIBRATED - 10*eta*Math.log10(Math.max(d,0.1)) - faf;
}

// ── Fitness: RMSE ระหว่าง RSSI_predict กับ RSSI_measured ──
function fitness(p, mobileRSSI, floor){
  var pz=p.z||getZ(floor||'Floor 4');
  var total=0;
  ANCHORS.forEach(function(an,i){
    var d = dist3D(p.x,p.y,pz,an.x,an.y);
    var predicted = predictRSSI(d, floor||'Floor 4');
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

// ── Pre-compute: KNN floor ของทุก PR (สำหรับใช้กรอง seed) ──────
// คำนวณครั้งเดียวแล้วเก็บใน cache
var _prKnnCache = null;

function buildPrKnnCache(){
  if(_prKnnCache) return _prKnnCache;
  _prKnnCache = {};
  PR_DB.forEach(function(pr){
    // คำนวณ KNN floor ของ PR นี้ (leave-one-out, ใช้ pRSSI เทียบ pRSSI)
    var flDist = {};
    FLOORS.forEach(function(fl){ flDist[fl] = []; });
    PR_DB.forEach(function(other){
      if(other.id === pr.id) return;
      var d = Math.sqrt(
        (pr.pRSSI[0]-other.pRSSI[0])**2+
        (pr.pRSSI[1]-other.pRSSI[1])**2+
        (pr.pRSSI[2]-other.pRSSI[2])**2
      );
      if(flDist[other.floor]) flDist[other.floor].push(d);
    });
    var scores = {};
    FLOORS.forEach(function(fl){
      var ds = flDist[fl];
      scores[fl] = ds.length ? ds.reduce(function(a,b){return a+b;},0)/ds.length : 999;
    });
    var knnFl = FLOORS.reduce(function(a,b){return scores[a]<scores[b]?a:b;});
    var knnNum = parseInt(knnFl.replace('Floor ',''));
    var trueNum = parseInt(pr.floor.replace('Floor ',''));
    _prKnnCache[pr.id] = {
      knnFloor: knnFl,
      deviation: Math.abs(knnNum - trueNum) // คลาดเคลื่อนกี่ชั้นจากชั้นจริง
    };
  });
  return _prKnnCache;
}

// ── KNN + PSO Seed selection ─────────────────────────────────────
// Rules:
// 1. KNN Avg รวม: เปรียบเทียบ mobileRSSI กับ PR ทุกตัว ยกเว้นตัวเอง
// 2. PSO รันในชั้นจริงของ Mobile เสมอ
// 3. Seeds = PR ในชั้นจริง ยกเว้น Mobile ตัวเอง
// 4. ตัด PR ที่ KNN ของตัวมันเองคลาดเคลื่อน >= 2 ชั้น (deviation 0,1 เท่านั้นผ่าน)
// ────────────────────────────────────────────────────────────────
function knnPredictFloor(mobileRSSI, mobileId, mobileActualFloor, useMode){
  var cache = buildPrKnnCache();

  var flDist = {};
  FLOORS.forEach(function(fl){ flDist[fl] = []; });

  PR_DB.forEach(function(pr){
    if(pr.id === mobileId) return; // leave-one-out
    var ref = (useMode === 'meas') ? pr.mRSSI : pr.pRSSI;
    var d = Math.sqrt(
      (mobileRSSI[0]-ref[0])**2 +
      (mobileRSSI[1]-ref[1])**2 +
      (mobileRSSI[2]-ref[2])**2
    );
    if(flDist[pr.floor]) flDist[pr.floor].push({pr:pr, d:d});
  });

  // Avg รวม แยกชั้น → ชั้นที่ avg ต่ำสุด = KNN prediction
  var scores = {};
  FLOORS.forEach(function(fl){
    var items = flDist[fl];
    scores[fl] = items.length ? items.reduce(function(s,x){return s+x.d;},0)/items.length : 999;
  });
  var knnFloor = FLOORS.reduce(function(a,b){return scores[a]<scores[b]?a:b;});

  // PSO รันในชั้นที่ KNN ทำนาย (ถ้าไม่ได้ส่ง override)
  var targetFloor = mobileActualFloor || knnFloor;

  // Seeds = PR ในชั้นจริง ยกเว้น Mobile ตัวเอง
  var allInFloor = (flDist[targetFloor] || []).slice();

  // กรอง: ตัด PR ที่ KNN ของตัวมันเองคลาดเคลื่อน >= 2 ชั้น จากชั้นจริงของมัน
  // (deviation 0 หรือ 1 เท่านั้น = น่าเชื่อถือพอเป็น seed)
  var reliableSeeds = allInFloor.filter(function(item){
    var info = cache[item.pr.id];
    if(!info) return true; // ไม่มี cache = ผ่าน (fallback)
    return info.deviation < 2; // ตัด: deviation >= 2 ออก (2 ชั้น, 3 ชั้น)
  });

  // fallback: ถ้า filter แล้วเหลือน้อยกว่า 3 ตัว ใช้ทั้งหมด
  var usedSeeds = reliableSeeds.length >= 3 ? reliableSeeds : allInFloor;
  var filtered  = allInFloor.length - reliableSeeds.length;

  var seeds = usedSeeds
    .sort(function(a,b){return a.d - b.d;})
    .map(function(x){return x.pr;});

  return {
    floor: targetFloor,
    knnFloor: knnFloor,
    seeds: seeds,
    scores: scores,
    filteredCount: reliableSeeds.length >= 3 ? filtered : 0,
    totalInFloor: allInFloor.length
  };
}

function euclid3RSSI(a,b){
  return Math.sqrt((a[0]-b[0])**2+(a[1]-b[1])**2+(a[2]-b[2])**2);
}

// ── PSO: ใช้เฉพาะ PR ในชั้นที่ KNN ทำนาย เป็น seed ──
function psoRunHybrid(mobileRSSI, mobile, knnResult, n, K, wmin, wmax, c1, c2, R, etaLos, etaNlos){
  var targetFloor = knnResult.floor;
  var seeds       = knnResult.seeds;
  var runs = [];
  for(var r=0; r<R; r++){
    runs.push(psoRunOneFloor(mobileRSSI, mobile, targetFloor, seeds, n, K, wmin, wmax, c1, c2));
  }
  var avgX = runs.reduce(function(s,r){return s+r.gx;},0)/runs.length;
  var avgY = runs.reduce(function(s,r){return s+r.gy;},0)/runs.length;
  var best = runs.reduce(function(a,b){return a.gcost<b.gcost?a:b;});

  // Local grid refinement ±2m step 0.2m
  var pz   = getZ(targetFloor);
  var bestL = {x:avgX, y:avgY, c:best.gcost};
  for(var dx=-2; dx<=2; dx+=0.2){
    for(var dy=-2; dy<=2; dy+=0.2){
      var tx = Math.max(0,Math.min(G.MW,avgX+dx));
      var ty = Math.max(0,Math.min(G.MH,avgY+dy));
      var p  = {x:tx, y:ty, z:pz};
      var c  = fitness(p, mobileRSSI, targetFloor);
      if(c < bestL.c) bestL = {x:tx, y:ty, c:c};
    }
  }
  return{
    gx: +(bestL.x.toFixed(2)), gy: +(bestL.y.toFixed(2)),
    gcost: +(bestL.c.toFixed(3)), predFloor: targetFloor,
    iterFitness: best.iterFitness,
    seedIds: seeds.map(function(s){return s.id;})
  };
}

function psoRunOneFloor(mobileRSSI, mobile, fl, seeds, n, K, wmin, wmax, c1, c2){
  var pz = getZ(fl);
  var particles = [];

  // Seed จาก PR ในชั้น
  seeds.forEach(function(s){
    particles.push({x:s.x+rand(-1.5,1.5), y:s.y+rand(-1.5,1.5), z:pz,
      vx:0, vy:0, bx:s.x, by:s.y, bc:Infinity});
  });
  // เติม random จนครบ n
  while(particles.length < n){
    particles.push({x:rand(0,G.MW), y:rand(0,G.MH), z:pz,
      vx:0, vy:0, bx:rand(0,G.MW), by:rand(0,G.MH), bc:Infinity});
  }

  particles.forEach(function(p){ p.bc=fitness(p,mobileRSSI,fl); p.bx=p.x; p.by=p.y; });
  var gbest = getBest(particles);
  var iterFitness = [];
  for(var k=1; k<=K; k++){
    var w = wmax - k*(wmax-wmin)/K;
    particles.forEach(function(p){
      var r1=Math.random(), r2=Math.random();
      p.vx = w*p.vx + c1*r1*(p.bx-p.x) + c2*r2*(gbest.x-p.x);
      p.vy = w*p.vy + c1*r1*(p.by-p.y) + c2*r2*(gbest.y-p.y);
      p.vx = Math.max(-4,Math.min(4,p.vx));
      p.vy = Math.max(-4,Math.min(4,p.vy));
      p.x  = Math.max(0,Math.min(G.MW, p.x+p.vx));
      p.y  = Math.max(0,Math.min(G.MH, p.y+p.vy));
      var cost = fitness(p, mobileRSSI, fl);
      if(cost < p.bc){ p.bc=cost; p.bx=p.x; p.by=p.y; }
      if(cost < gbest.bc){ gbest={x:p.x,y:p.y,bc:cost}; }
    });
    iterFitness.push(gbest.bc);
  }
  return {gx:gbest.x, gy:gbest.y, gcost:gbest.bc, iterFitness:iterFitness};
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
  // KNN accuracy = กี่ตัวที่ KNN ทำนายถูกชั้น
  var okM = psoLogs.filter(function(l){return l.knnFloorM===l.floor;}).length;
  var okR = psoLogs.filter(function(l){return l.knnFloorR===l.floor;}).length;
  var avgErrM = (psoLogs.reduce(function(s,l){return s+l.xyErrM;},0)/t).toFixed(2);
  var avgErrR = (psoLogs.reduce(function(s,l){return s+l.xyErrR;},0)/t).toFixed(2);

  // Error thresholds
  var lt3M  = psoLogs.filter(function(l){return l.xyErrM<=3;}).length;
  var lt6M  = psoLogs.filter(function(l){return l.xyErrM<=6;}).length;
  var lt10M = psoLogs.filter(function(l){return l.xyErrM<=10;}).length;
  var lt3R  = psoLogs.filter(function(l){return l.xyErrR<=3;}).length;
  var lt6R  = psoLogs.filter(function(l){return l.xyErrR<=6;}).length;
  var lt10R = psoLogs.filter(function(l){return l.xyErrR<=10;}).length;

  document.getElementById('pso-acc-m').textContent     = ((okM/t)*100).toFixed(1)+'%';
  document.getElementById('pso-acc-r').textContent     = ((okR/t)*100).toFixed(1)+'%';
  document.getElementById('pso-avg-err-m').textContent = avgErrM+' m';
  document.getElementById('pso-avg-err-r').textContent = avgErrR+' m';
  document.getElementById('pso-lt3-m').textContent     = lt3M+'/'+t;
  document.getElementById('pso-lt6-m').textContent     = lt6M+'/'+t;
  document.getElementById('pso-lt10-m').textContent    = lt10M+'/'+t;
  document.getElementById('pso-lt3-r').textContent     = lt3R+'/'+t;
  document.getElementById('pso-lt6-r').textContent     = lt6R+'/'+t;
  document.getElementById('pso-lt10-r').textContent    = lt10R+'/'+t;
  document.getElementById('pso-count').textContent     = t+' Particles';

  // Per-floor avg error breakdown
  function floorErrHTML(errKey, color){
    return FLOORS.map(function(fl){
      var sub = psoLogs.filter(function(l){return l.floor===fl;});
      if(!sub.length) return '';
      var avg = (sub.reduce(function(s,l){return s+l[errKey];},0)/sub.length).toFixed(2);
      var fn = fl.replace('Floor ','');
      var fpCls = {'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
      return '<span style="display:inline-flex;align-items:center;gap:5px;margin-right:10px;font-size:11px;">'+
        '<span class="floor-pill '+(fpCls[fn]||'')+'" style="font-size:9px;padding:1px 6px;">'+fl+'</span>'+
        '<span style="font-family:var(--mono);color:'+color+'">'+avg+'m</span></span>';
    }).join('');
  }
  document.getElementById('pso-floor-err-m').innerHTML = floorErrHTML('xyErrM','#a855f7');
  document.getElementById('pso-floor-err-r').innerHTML = floorErrHTML('xyErrR','#10b981');

  // 2 Convergence Charts แยก Meas / RSSI
  var cOpts = function(data, color){
    return {type:'line',data:{labels:data.map(function(_,i){return 'k='+(i+1);}),
      datasets:[{data:data,borderColor:color,
        backgroundColor:color+'22',pointRadius:2,tension:.35,borderWidth:2,fill:true}]},
      options:{responsive:true,maintainAspectRatio:false,
        plugins:{legend:{display:false}},
        scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C}},
                x:{grid:{display:false},ticks:{color:TICK_C,maxTicksLimit:10}}}}};
  };
  mkChart('psoConvergeChartM', cOpts(psoIterAvgM,'#a855f7'));
  mkChart('psoConvergeChart',  cOpts(psoIterAvg, '#10b981'));

  // Table — แต่ละแถวคลิกที่ Particle ID เพื่อดู Seed PR ที่นำไป PSO
  window._seedData = window._seedData || {};
  var fp = {'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
  var tbody = document.getElementById('pso-table-body');
  tbody.innerHTML = '';

  psoLogs.forEach(function(log, idx){
    var fn   = log.floor.replace('Floor ','');
    var ecM  = log.xyErrM<=3?'var(--ok)':log.xyErrM<=6?'var(--amber)':log.xyErrM<=10?'var(--rose)':'var(--err)';
    var ecR  = log.xyErrR<=3?'var(--ok)':log.xyErrR<=6?'var(--amber)':log.xyErrR<=10?'var(--rose)':'var(--err)';
    var okFM = log.predFloorM===log.floor;
    var okFR = log.predFloorR===log.floor;

    var seedLabelM = log.seedIdsM ? log.seedIdsM.length+
      (log.filteredCountM>0?' / '+(log.seedIdsM.length+log.filteredCountM)+' PR':' PR') : '—';
    var seedLabelR = log.seedIdsR ? log.seedIdsR.length+
      (log.filteredCountR>0?' / '+(log.seedIdsR.length+log.filteredCountR)+' PR':' PR') : '—';
    var seedColorM = log.filteredCountM>0 ? 'var(--amber)' : 'var(--muted)';
    var seedColorR = log.filteredCountR>0 ? 'var(--amber)' : 'var(--muted)';
    var keyM = 'seed_'+idx+'_M';
    var keyR = 'seed_'+idx+'_R';
    window._seedData[keyM] = {
      title: 'Measurement Seeds — '+log.id,
      seeds: (log.seedIdsM||[]).map(function(id){
        var pr=PR_DB.find(function(p){return p.id===id;})||{};
        return {id:id,floor:pr.floor||'',x:pr.x||0,y:pr.y||0};
      }),
      result: {id:log.id,floor:log.floor,gx:log.gxM,gy:log.gyM,err:log.xyErrM.toFixed(2),filteredCount:log.filteredCountM||0}
    };
    window._seedData[keyR] = {
      title: 'RSSI Predict Seeds — '+log.id,
      seeds: (log.seedIdsR||[]).map(function(id){
        var pr=PR_DB.find(function(p){return p.id===id;})||{};
        return {id:id,floor:pr.floor||'',x:pr.x||0,y:pr.y||0};
      }),
      result: {id:log.id,floor:log.floor,gx:log.gxR,gy:log.gyR,err:log.xyErrR.toFixed(2),filteredCount:log.filteredCountR||0}
    };

    // knnFloor = ชั้นที่ KNN ทำนาย (อาจผิด), predFloor = ชั้นจริงที่รัน PSO
    var knnOkM = log.knnFloorM === log.floor;
    var knnOkR = log.knnFloorR === log.floor;
    var seedLabelM = (log.seedIdsM||[]).length + (log.filteredCountM>0?' / '+((log.seedIdsM||[]).length+log.filteredCountM)+' PR':' PR');
    var seedLabelR = (log.seedIdsR||[]).length + (log.filteredCountR>0?' / '+((log.seedIdsR||[]).length+log.filteredCountR)+' PR':' PR');
    var seedColorM = (log.filteredCountM||0)>0 ? 'var(--amber)' : 'var(--muted)';
    var seedColorR = (log.filteredCountR||0)>0 ? 'var(--amber)' : 'var(--muted)';

    var tr = document.createElement('tr');
    tr.style.cursor = 'pointer';
    tr.title = 'คลิกซ้าย = Meas Seeds | คลิกขวา = RSSI Seeds';
    tr.innerHTML =
      '<td><span class="floor-pill '+(fp[fn]||'')+'">'+log.floor+'</span></td>'+
      '<td style="color:var(--purple);font-weight:600">'+log.id+'</td>'+
      '<td>'+(knnOkM
        ? '<span class="badge badge-ok">'+log.knnFloorM+'</span>'
        : '<span class="badge badge-err" title="KNN ทำนาย'+log.knnFloorM+' ≠ จริง">'+log.knnFloorM+' ✗</span>')+
      '</td>'+
      '<td style="color:var(--muted);font-size:10px">'+log.gxM+', '+log.gyM+'</td>'+
      '<td style="color:'+ecM+';font-weight:600">'+log.xyErrM.toFixed(2)+'</td>'+
      '<td style="color:'+seedColorM+';font-size:10px;cursor:pointer;text-decoration:underline dotted;" title="คลิกเพื่อดู Seeds (สีเหลือง=มีการตัดออก)">'+seedLabelM+'</td>'+
      '<td>'+(knnOkR
        ? '<span class="badge badge-ok">'+log.knnFloorR+'</span>'
        : '<span class="badge badge-err" title="KNN ทำนาย'+log.knnFloorR+' ≠ จริง">'+log.knnFloorR+' ✗</span>')+
      '</td>'+
      '<td style="color:var(--muted);font-size:10px">'+log.gxR+', '+log.gyR+'</td>'+
      '<td style="color:'+ecR+';font-weight:600">'+log.xyErrR.toFixed(2)+'</td>'+
      '<td style="color:'+seedColorR+';font-size:10px;cursor:pointer;text-decoration:underline dotted;" title="คลิกเพื่อดู Seeds (สีเหลือง=มีการตัดออก)">'+seedLabelR+'</td>';

    // คลิกซ้ายครึ่ง = Meas seeds, คลิกขวาครึ่ง = RSSI seeds
    tr.addEventListener('click', function(e){
      var rect = tr.getBoundingClientRect();
      var isLeft = (e.clientX - rect.left) < rect.width/2;
      var key = isLeft ? keyM : keyR;
      var d = window._seedData[key];
      if(d) window.showSeedPopup(d.title, d.seeds, d.result);
    });
    tbody.appendChild(tr);
  });

  document.getElementById('pso-results-section').classList.remove('hidden');
  // อัพเดทตาราง KNN+PSO ด้วย (ให้ผลตรงกัน)
  renderKNNTable();
  document.getElementById('tbl-empty').classList.add('hidden');
  document.getElementById('tbl-content').classList.remove('hidden');
  renderMaps();
}

// ── Seed Popup: แสดง PR ที่นำไป PSO พร้อม gbest result ──
// expose to window so onclick attributes can call them
window.showSeedPopup = function showSeedPopup(title, seeds, result){
  document.getElementById('dev-popup-title').textContent = title;
  var fp = {'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
  var errColor = result.err<=3?'var(--ok)':result.err<=6?'var(--amber)':'var(--err)';
  var fc = result.filteredCount||0;
  var html =
    '<div style="background:var(--bg4);border-radius:var(--rs);padding:10px 14px;margin-bottom:12px;font-size:12px;line-height:1.9;">'+
    '<span style="color:var(--muted)">Mobile: </span><b style="color:var(--text)">'+result.id+'</b>'+
    '<span style="color:var(--muted);margin-left:14px;">Floor จริง: </span><b style="color:var(--text)">'+result.floor+'</b>'+
    '<br><span style="color:var(--muted)">gbest: </span>'+
    '<span style="font-family:var(--mono);color:var(--blue)">('+result.gx+', '+result.gy+')</span>'+
    '<span style="color:var(--muted);margin-left:14px;">XY Error: </span>'+
    '<span style="font-family:var(--mono);color:'+errColor+'">'+result.err+' m</span>'+
    '</div>'+
    '<div style="font-size:11px;color:var(--muted);margin-bottom:8px;">'+
    'PR ที่นำไปเป็น Seed ใน PSO <b style="color:var(--text)">'+seeds.length+' ตัว</b>'+
    (fc>0?' &nbsp;<span style="color:var(--amber);font-size:10px;">(ตัดออก '+fc+' ตัว คลาดเคลื่อน>2ชั้น)</span>':
          ' &nbsp;<span style="color:var(--ok);font-size:10px;">(ผ่านทุกตัว)</span>')+
    '</div>'+
    '<table style="width:100%;font-size:12px;border-collapse:collapse;">'+
    '<thead><tr style="background:var(--bg4);">'+
    '<th style="padding:5px 10px;color:var(--muted2);text-align:center;">#</th>'+
    '<th style="padding:5px 10px;color:var(--purple);text-align:left;">Particle ID</th>'+
    '<th style="padding:5px 10px;color:var(--muted);">Floor</th>'+
    '<th style="padding:5px 10px;color:var(--blue);">X (m)</th>'+
    '<th style="padding:5px 10px;color:var(--blue);">Y (m)</th>'+
    '</tr></thead><tbody>'+
    seeds.map(function(s,i){
      var fn=(s.floor||'').replace('Floor ','');
      return '<tr style="border-bottom:0.5px solid var(--border);">'+
        '<td style="padding:5px 10px;font-family:var(--mono);color:var(--muted2);text-align:center;">'+(i+1)+'</td>'+
        '<td style="padding:5px 10px;font-family:var(--mono);color:var(--purple)">'+s.id+'</td>'+
        '<td style="padding:5px 10px;"><span class="floor-pill '+(fp[fn]||'')+'">'+s.floor+'</span></td>'+
        '<td style="padding:5px 10px;font-family:var(--mono);color:var(--blue)">'+s.x+'</td>'+
        '<td style="padding:5px 10px;font-family:var(--mono);color:var(--blue)">'+s.y+'</td>'+
        '</tr>';
    }).join('')+
    '</tbody></table>';
  document.getElementById('dev-popup-body').innerHTML = html;
  document.getElementById('dev-popup-overlay').style.display='block';
  document.getElementById('dev-popup').style.display='block';
}


  // 2 Convergence Charts แยก Meas / RSSI
  var cOpts = function(data, color, label){
    return {type:'line',data:{labels:data.map(function(_,i){return 'k='+(i+1);}),
      datasets:[{label:label,data:data,borderColor:color,
        backgroundColor:color.replace('rgb','rgba').replace(')',',0.08)'),
        pointRadius:2,tension:.35,borderWidth:2,fill:true}]},
      options:{responsive:true,maintainAspectRatio:false,
        plugins:{legend:{display:false}},
        scales:{y:{beginAtZero:true,grid:{color:GRID_C},ticks:{color:TICK_C}},
                x:{grid:{display:false},ticks:{color:TICK_C,maxTicksLimit:10}}}}};
  };
  mkChart('psoConvergeChartM', cOpts(psoIterAvgM,'#a855f7','Meas. Fitness'));
  mkChart('psoConvergeChart',  cOpts(psoIterAvg, '#10b981','RSSI Fitness'));

window.showDevPopup = function showDevPopup(title, prList){
  document.getElementById('dev-popup-title').textContent = title;
  var fp = {'1':'fp1','2':'fp2','3':'fp3','4':'fp4'};
  var html = prList.length===0
    ? '<p style="color:var(--ok);font-size:13px;">✅ ไม่มีที่หลงชั้น</p>'
    : '<table style="width:100%;font-size:12px;border-collapse:collapse;">'+
      '<thead><tr style="background:var(--bg4);"><th style="padding:6px 10px;color:var(--muted);text-align:left;">Particle</th>'+
      '<th style="padding:6px 10px;color:var(--muted);">Floor จริง</th>'+
      '<th style="padding:6px 10px;color:var(--err);">ทำนายได้</th>'+
      '<th style="padding:6px 10px;color:var(--amber);">ห่าง</th></tr></thead><tbody>'+
      prList.map(function(item){
        var fn=item.actualFloor.replace('Floor ','');
        return '<tr style="border-bottom:0.5px solid var(--border);">'+
          '<td style="padding:6px 10px;font-family:var(--mono);color:var(--text)">'+item.id+'</td>'+
          '<td style="padding:6px 10px;"><span class="floor-pill '+(fp[fn]||'')+'">'+item.actualFloor+'</span></td>'+
          '<td style="padding:6px 10px;font-family:var(--mono);color:var(--err)">'+item.predicted+'</td>'+
          '<td style="padding:6px 10px;font-family:var(--mono);color:var(--amber)">'+item.gap+' ชั้น</td></tr>';
      }).join('')+'</tbody></table>';
  document.getElementById('dev-popup-body').innerHTML = html;
  document.getElementById('dev-popup-overlay').style.display='block';
  document.getElementById('dev-popup').style.display='block';
}


// ── FLOOR MAPS ──
var mapColors={
  'Floor 1':'#f59e0b','Floor 2':'#3b82f6','Floor 3':'#10b981','Floor 4':'#a855f7'
};
var floorCanvases={'Floor 1':'map-f1','Floor 2':'map-f2','Floor 3':'map-f3','Floor 4':'map-f4'};

function renderMaps(){
  FLOORS.forEach(function(fl){
    var cid = floorCanvases[fl];
    var el  = document.getElementById(cid);
    if(!el) return;
    var wrap = el.parentElement;
    var W    = wrap.clientWidth || 400;
    var H    = Math.round(W * (G.MH/G.MW));
    var dpr  = window.devicePixelRatio || 1;
    el.width  = W*dpr; el.height = H*dpr;
    el.style.width = W+"px"; el.style.height = H+"px";
    var ctx = el.getContext("2d");
    ctx.setTransform(1,0,0,1,0,0);
    ctx.scale(dpr,dpr);

    var PAD=20, MAP_W=G.MW, MAP_H=G.MH;
    var dW=W-PAD*2, dH=H-PAD*2;
    // x0,y0 = มุมขวาล่าง (x→ซ้าย, y↑)
    function cx(rx){ return PAD+(MAP_W-rx)/MAP_W*dW; }
    function cy(ry){ return PAD+(MAP_H-ry)/MAP_H*dH; }

    ctx.fillStyle="#080c14";
    ctx.fillRect(0,0,W,H);

    drawFloorPlan(ctx,fl,cx,cy,W,H,PAD,dW,dH);
    drawFloorPRs(ctx,fl,cx,cy);

    // Floor label
    ctx.fillStyle="rgba(0,0,0,0.6)";
    ctx.fillRect(PAD,PAD+2,90,15);
    ctx.fillStyle=mapColors[fl]||"#fff";
    ctx.font="bold 9px Noto Sans Thai,monospace";
    ctx.textAlign="left";
    var flPRs=PR_DB.filter(function(p){return p.floor===fl;});
    ctx.fillText(fl+" ("+flPRs.length+" PR)",PAD+4,PAD+13);
  });
}

// ── วาด Floor Plan ด้วย Canvas (ไม่ใช้ภาพ) ──
// geometry อิงจากแปลนจริง x0,y0=มุมขวาล่าง
// ══════════════════════════════════════════════════════
// FLOOR PLAN GEOMETRY — จาก PDF แบบชั้น 4
// Coordinate: x0,y0 = มุมขวาล่าง
//   X เพิ่มไปซ้าย  (0=ขวาสุด → 36.08=ซ้ายสุดอาคาร)
//   Y เพิ่มขึ้นบน   (0=ล่างสุด → ~25m=บนสุด)
//
// Layout แนวตั้ง (Y):
//   ห้องล่าง:    y = 0     – 14.28
//   Setback:     y = 14.28 – 16.51  (เว้า 2.23m)
//   ห้องบน:      y = 16.51 – 24.77  (8.26m)
//   (ระเบียงบน:  y = 24.77 – 26.66)
//
// Layout แนวนอน (X) ← อ้างอิงจาก x=0=ขวา:
//   16414/16413-15: x = 0     – 16.25  (ขวา)
//   Stairs/Corridor:x = 16.25 – 19.83  (3.58m)
//   Toilet:         x = 19.83 – 25.14  (5.31m)
//   Teacher2:       x = 25.14 – 30.45  (5.31m)
//   Teacher1:       x = 30.45 – 36.08  (5.63m)
//   ระเบียง:        x = 36.08 – 37.97  (1.89m นอกอาคาร)
// ══════════════════════════════════════════════════════

var G = {
  MW: 36.08,   // Map Width (x) from AutoCAD
  MH: 26.07,   // Map Height (y) = 26.07 confirmed from AutoCAD dim
  X: {
    mid:    16.25,  // wall: 16414 | stairs
    stair:  19.83,  // wall: stairs | toilet
    toilet: 25.14,  // wall: toilet | teacher2
    tchr2:  30.45,  // wall: teacher2 | teacher1
    tchr1:  36.08,  // left wall of main building
    balc:   37.97,  // balcony outer wall
  },
  Y: {
    lower:   14.28,  // top of lower rooms (16412/16414)
    corrTop: 17.81,  // top of Central Corridor = 14.28+3.53
    setTop:  20.04,  // top of setback void    = 17.81+2.23
    upper:   26.07,  // top of upper rooms     = 17.81+8.26
    balcH:    7.24,  // balcony lower height
  }
};

var FLOOR_PLAN_GEO = (function(){
  // Geometry from AutoCAD Floor_4 (confirmed coordinates)
  // Y: lower=14.28, corrTop=17.81(+3.53), setTop=20.04(+2.23), upper=26.07(+8.26)
  // Setback: Teacher2+Toilet+Stairs (14.20m) bottom=setTop=20.04
  // Teacher1+16413-15 protrude down to corrTop=17.81
  function makeFloor(nameLL, nameLR, nameUL){
    var x=G.X, y=G.Y;
    return {
      walls:[
        // vertical lower
        [x.mid,   0,         x.mid,   y.lower  ],
        [x.stair, 0,         x.stair, y.lower  ],
        [x.tchr1, 0,         x.tchr1, y.lower  ],
        // vertical upper
        [x.mid,   y.setTop,  x.mid,   y.upper  ],
        [x.stair, y.setTop,  x.stair, y.upper  ],
        [x.toilet,y.setTop,  x.toilet,y.upper  ],
        [x.tchr2, y.setTop,  x.tchr2, y.upper  ],
        [x.tchr1, y.setTop,  x.tchr1, y.upper  ],
        [x.balc,  0,         x.balc,  y.upper  ],
        // horizontal lower top
        [0,       y.lower,   x.mid,   y.lower  ],
        [x.stair, y.lower,   x.tchr1, y.lower  ],
        // horizontal upper top
        [0,       y.upper,   x.balc,  y.upper  ],
        // setback step: Teacher1+16413-15 protrude to corrTop
        [0,       y.corrTop, x.mid,   y.corrTop],  // 16413-15 bottom
        [x.tchr2, y.corrTop, x.tchr1, y.corrTop],  // Teacher1 bottom
        // setback vertical steps
        [x.mid,   y.corrTop, x.mid,   y.setTop ],
        [x.tchr2, y.corrTop, x.tchr2, y.setTop ],
        // setback floor (Teacher2+Toilet+Stairs bottom)
        [x.mid,   y.setTop,  x.tchr2, y.setTop ],
        // left wall full
        [0,       0,         0,       y.upper  ],
        // balcony lower
        [x.tchr1, y.balcH,   x.balc,  y.balcH  ],
        [x.tchr1, y.balcH,   x.tchr1, y.lower  ],
        [x.tchr1, y.setTop,  x.balc,  y.setTop ],
      ],
      rooms:[
        [0,       0,         x.mid,   y.lower,  nameLL,        'rgba(18,36,80,0.92)'  ],
        [x.stair, 0,         x.tchr1, y.lower,  nameLR,        'rgba(8,46,30,0.90)'   ],
        [x.mid,   0,         x.stair, y.lower,  '',            'rgba(6,12,35,0.95)'   ],
        [0,       y.lower,   x.tchr1, y.corrTop,'',            'rgba(4,8,20,0.98)'    ],
        [0,       y.corrTop, x.mid,   y.upper,  nameUL,        'rgba(28,12,62,0.88)'  ],
        [x.mid,   y.setTop,  x.stair, y.upper,  'Stairs',      'rgba(10,22,50,0.90)'  ],
        [x.stair, y.setTop,  x.toilet,y.upper,  'Toilet',      'rgba(18,26,46,0.88)'  ],
        [x.toilet,y.setTop,  x.tchr2, y.upper,  'Teacher Rm',  'rgba(32,22,8,0.80)'   ],
        [x.tchr2, y.corrTop, x.tchr1, y.upper,  'Teacher Rm',  'rgba(32,22,6,0.78)'   ],
        [x.mid,   y.corrTop, x.tchr2, y.setTop, '',            'rgba(2,4,12,0.99)'    ],
        [x.tchr1, 0,         x.balc,  y.balcH,  '',            'rgba(14,14,18,0.80)'  ],
        [x.tchr1, y.setTop,  x.balc,  y.upper,  '',            'rgba(14,14,18,0.80)'  ],
      ],
      stairs:{ x1:x.mid, y1:y.setTop, x2:x.stair, y2:y.upper, steps:6 },
      balcony:[
        {x1:x.tchr1,y1:0,       x2:x.balc,y2:y.balcH},
        {x1:x.tchr1,y1:y.setTop,x2:x.balc,y2:y.upper},
      ],
      corridor:{ y1:y.lower, y2:y.corrTop },
    };
  }
  return {
    'Floor 4': makeFloor('16414','16412','16413-15'),
    'Floor 3': makeFloor('16310','16308','16312'),
    'Floor 2': makeFloor('',    '16212',''),
    'Floor 1': makeFloor('16112','16110','16111'),
  };
})();

function drawFloorPlan(ctx, fl, cx, cy, W, H, PAD, dW, dH){
  var geo = FLOOR_PLAN_GEO[fl];
  if(!geo) return;
  var y=G.Y, x=G.X;

  // ── Background
  ctx.fillStyle='rgba(6,9,18,0.97)';
  ctx.fillRect(cx(x.balc)-2, cy(y.upper)-2, cx(-1)-cx(x.balc)+4, cy(-0.5)-cy(y.upper)+4);

  // ── Room fills
  geo.rooms.forEach(function(r){
    var rx1=cx(r[2]),ry1=cy(r[3]),rx2=cx(r[0]),ry2=cy(r[1]);
    ctx.fillStyle=r[5];
    ctx.fillRect(rx1,ry1,rx2-rx1,ry2-ry1);
  });

  // ── Hatched balcony
  geo.balcony.forEach(function(b){
    var bx1=cx(b.x2),by1=cy(b.y2),bx2=cx(b.x1),by2=cy(b.y1);
    ctx.save();
    ctx.beginPath(); ctx.rect(bx1,by1,bx2-bx1,by2-by1); ctx.clip();
    ctx.strokeStyle='rgba(120,130,155,0.22)'; ctx.lineWidth=0.6;
    for(var d=bx1-(by2-by1)-30;d<bx2+40;d+=6){
      ctx.beginPath(); ctx.moveTo(d,by1); ctx.lineTo(d+by2-by1,by2); ctx.stroke();
    }
    ctx.restore();
  });

  // ── Stairs hatching
  var st=geo.stairs;
  var sx1=cx(st.x2),sy1=cy(st.y2),sx2=cx(st.x1),sy2=cy(st.y1);
  ctx.strokeStyle='rgba(80,110,185,0.28)'; ctx.lineWidth=0.6;
  var sh=(sy2-sy1)/st.steps;
  for(var si=0;si<=st.steps;si++){
    ctx.beginPath(); ctx.moveTo(sx1,sy1+si*sh); ctx.lineTo(sx2,sy1+si*sh); ctx.stroke();
  }

  // ── Inner partition walls
  ctx.strokeStyle='rgba(185,205,238,0.58)'; ctx.lineWidth=0.9;
  geo.walls.forEach(function(w){
    ctx.beginPath(); ctx.moveTo(cx(w[0]),cy(w[1])); ctx.lineTo(cx(w[2]),cy(w[3])); ctx.stroke();
  });

  // ── Outer boundary (bold)
  ctx.strokeStyle='rgba(220,235,255,0.90)'; ctx.lineWidth=1.8;
  // Bottom
  ctx.beginPath(); ctx.moveTo(cx(0),cy(0)); ctx.lineTo(cx(x.tchr1),cy(0)); ctx.stroke();
  // Right wall
  ctx.beginPath(); ctx.moveTo(cx(0),cy(0)); ctx.lineTo(cx(0),cy(y.upper)); ctx.stroke();
  // Top
  ctx.beginPath(); ctx.moveTo(cx(0),cy(y.upper)); ctx.lineTo(cx(x.balc),cy(y.upper)); ctx.stroke();
  // Balcony outer
  ctx.beginPath(); ctx.moveTo(cx(x.balc),cy(0)); ctx.lineTo(cx(x.balc),cy(y.upper)); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(cx(x.tchr1),cy(0)); ctx.lineTo(cx(x.balc),cy(0)); ctx.stroke();

  // ── Setback outline (THE KEY SHAPE)
  ctx.strokeStyle='rgba(220,235,255,0.90)'; ctx.lineWidth=1.8;
  // 16413-15 bottom (protrudes to corrTop)
  ctx.beginPath(); ctx.moveTo(cx(0),cy(y.corrTop)); ctx.lineTo(cx(x.mid),cy(y.corrTop)); ctx.stroke();
  // Teacher1 bottom (protrudes to corrTop)
  ctx.beginPath(); ctx.moveTo(cx(x.tchr2),cy(y.corrTop)); ctx.lineTo(cx(x.tchr1),cy(y.corrTop)); ctx.stroke();
  // Setback step right
  ctx.beginPath(); ctx.moveTo(cx(x.mid),cy(y.corrTop)); ctx.lineTo(cx(x.mid),cy(y.setTop)); ctx.stroke();
  // Setback step left
  ctx.beginPath(); ctx.moveTo(cx(x.tchr2),cy(y.corrTop)); ctx.lineTo(cx(x.tchr2),cy(y.setTop)); ctx.stroke();
  // Setback floor (Teacher2+Toilet+Stairs bottom)
  ctx.beginPath(); ctx.moveTo(cx(x.mid),cy(y.setTop)); ctx.lineTo(cx(x.tchr2),cy(y.setTop)); ctx.stroke();

  // ── Corridor dashed boundary
  ctx.setLineDash([3,3]);
  ctx.strokeStyle='rgba(70,130,200,0.38)'; ctx.lineWidth=0.7;
  ctx.beginPath(); ctx.moveTo(cx(0),cy(y.lower)); ctx.lineTo(cx(x.mid),cy(y.lower)); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(cx(x.stair),cy(y.lower)); ctx.lineTo(cx(x.tchr1),cy(y.lower)); ctx.stroke();
  ctx.setLineDash([]);

  // ── Room labels
  ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.font='bold 7.5px IBM Plex Mono,monospace';
  geo.rooms.forEach(function(r){
    if(!r[4]) return;
    var lx=(cx(r[0])+cx(r[2]))/2;
    var ly=(cy(r[1])+cy(r[3]))/2;
    ctx.fillStyle='rgba(56,195,175,0.70)';
    ctx.fillText(r[4],lx,ly);
  });

  // ── Corridor label
  var corrY=(cy(y.lower)+cy(y.corrTop))/2;
  ctx.font='6.5px IBM Plex Mono,monospace';
  ctx.fillStyle='rgba(70,140,210,0.52)';
  ctx.fillText('Central Corridor', cx(x.r16415/2||8), corrY);
}

function drawFloorPRs(ctx,fl,cx,cy){
  var flPRs=PR_DB.filter(function(p){return p.floor===fl;});
  var isF4=fl==="Floor 4";

  // Floor 4: สามเหลี่ยม AN + เส้น
  if(isF4){
    // เส้นเชื่อม AN
    ctx.strokeStyle="rgba(168,85,247,0.4)";
    ctx.lineWidth=0.8; ctx.setLineDash([4,3]);
    [[0,1],[1,2],[2,0]].forEach(function(p){
      ctx.beginPath();
      ctx.moveTo(cx(ANCHORS[p[0]].x),cy(ANCHORS[p[0]].y));
      ctx.lineTo(cx(ANCHORS[p[1]].x),cy(ANCHORS[p[1]].y));
      ctx.stroke();
    });
    ctx.setLineDash([]);
  }

  // PR dots
  flPRs.forEach(function(pr){
    var px=cx(pr.x),py=cy(pr.y);
    var knnLog=knnLogs.find(function(l){return l.id===pr.id;});
    var psoLog=psoLogs.find(function(l){return l.id===pr.id;});
    var dotC="rgba(255,255,255,0.35)";
    if(knnLog){
      dotC=(knnLog.mAvg==="ถูก"&&knnLog.rAvg==="ถูก")?"#22c55e":"#ef4444";
    }
    var isOpen=OPEN_ATRIUM_IDS.has(pr.id);
    var r=isOpen?4.5:3.5;

    ctx.shadowColor=dotC; ctx.shadowBlur=5;
    ctx.beginPath();ctx.arc(px,py,r,0,Math.PI*2);
    ctx.fillStyle=dotC;ctx.fill();
    ctx.shadowBlur=0;

    if(isOpen){
      ctx.strokeStyle="#f59e0b";ctx.lineWidth=1;ctx.stroke();
    }

    ctx.shadowColor="rgba(0,0,0,0.9)";ctx.shadowBlur=3;
    ctx.fillStyle="rgba(255,255,255,0.9)";
    ctx.font="bold 6.5px IBM Plex Mono,monospace";
    ctx.textAlign="center";
    ctx.fillText(pr.id,px,py-r-2);
    ctx.shadowBlur=0;

    // PSO gbest ★
    if(psoLog){
      var gx=cx(psoLog.gxR),gy=cy(psoLog.gyR);
      ctx.shadowColor="#facc15";ctx.shadowBlur=8;
      ctx.beginPath();ctx.arc(gx,gy,4,0,Math.PI*2);
      ctx.fillStyle="#facc15";ctx.fill();
      ctx.shadowBlur=0;
      ctx.beginPath();ctx.moveTo(px,py);ctx.lineTo(gx,gy);
      ctx.strokeStyle="rgba(250,204,21,0.3)";ctx.lineWidth=0.7;
      ctx.setLineDash([2,2]);ctx.stroke();ctx.setLineDash([]);
    }
  });

  // AN Nodes สามเหลี่ยมสีม่วง (F4 เท่านั้น)
  if(isF4){
    ANCHORS.forEach(function(an){
      var ax=cx(an.x),ay=cy(an.y),s=6;
      ctx.shadowColor="#a855f7";ctx.shadowBlur=10;
      ctx.beginPath();ctx.moveTo(ax,ay-s);ctx.lineTo(ax+s,ay+s);ctx.lineTo(ax-s,ay+s);
      ctx.closePath();ctx.fillStyle="#a855f7";ctx.fill();
      ctx.strokeStyle="#e9d5ff";ctx.lineWidth=0.8;ctx.stroke();
      ctx.shadowBlur=0;
      ctx.fillStyle="rgba(255,255,255,0.95)";
      ctx.font="bold 7px IBM Plex Mono,monospace";
      ctx.textAlign="center";
      ctx.fillText(an.id,ax,ay-s-3);
    });
  }
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
    var knnRes  = knnPredictFloor(mRSSI, mobile.id);
    var fl      = knnRes.floor;
    var seeds   = knnRes.seeds;
    var pz      = getZ(fl);

    // Build initial particles
    var particles = [];
    seeds.forEach(function(s){
      particles.push({x:s.x+rand(-2,2),y:s.y+rand(-2,2),z:pz,
        vx:0,vy:0,bx:s.x,by:s.y,bc:Infinity});
    });
    while(particles.length<n){
      particles.push({x:rand(0,G.MW),y:rand(0,G.MH),z:pz,
        vx:0,vy:0,bx:rand(0,G.MW),by:rand(0,G.MH),bc:Infinity});
    }

    // Init fitness ใช้ predictRSSI ตาม floor (ไม่มี FAF แยก)
    particles.forEach(function(p){
      p.bc=fitness(p,mRSSI,fl); p.bx=p.x; p.by=p.y;
    });
    var gbest=getBest(particles);
    animState.particles = particles;
    animState.gbest = gbest;
    animState.history = [];
    animState.totalK = K;
    animState.currentK = 0;
    animState.floor = fl;    // ใช้สำหรับ fitness ใน step
    animState.mRSSI = mRSSI;
    animState.seeds = seeds;
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
      'KNN → '+fl+' | '+mobile.id+' | mode: '+animState.rssiMode.toUpperCase()+' | seed: '+seeds.length+' PR';
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
      var cost=fitness(p,animState.mRSSI,animState.floor);
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

  var ANIM_PAD = 20;
  // ── Animator coordinate transform (same as renderMaps: x0=right, x→left, y↑)
  function acx(rx, W){ return ANIM_PAD + (G.MW - rx) / G.MW * (W - ANIM_PAD*2); }
  function acy(ry, H){ return ANIM_PAD + (G.MH - ry) / G.MH * (H - ANIM_PAD*2); }

  function clearAnimCanvas(){
    var a = getAnimCanvas();
    a.ctx.fillStyle = '#080c14';
    a.ctx.fillRect(0, 0, a.W, a.H);
  }

  function drawAnimCanvas(){
    if(!animState.mobile||!animState.particles.length) return;
    drawScene(
      animState.particles.map(function(p){return{x:p.x,y:p.y,bc:p.bc};}),
      animState.gbest,
      animState.currentK
    );
  }

  function drawFromSnapshot(snap){
    drawScene(snap.particles, snap.gbest, snap.k);
  }

  function drawScene(particles, gbest, k){
    var a   = getAnimCanvas();
    var W   = a.W, H = a.H;
    var ctx = a.ctx;
    var mob = animState.mobile;

    // local shortcuts using animator transform
    function lx(rx){ return acx(rx, W); }
    function ly(ry){ return acy(ry, H); }

    ctx.fillStyle='#060912'; ctx.fillRect(0,0,W,H);

    // ── Draw floor plan background
    var fl = animState.floor || 'Floor 4';
    drawFloorPlan(ctx, fl,
      function(rx){ return lx(rx); },
      function(ry){ return ly(ry); },
      W, H, ANIM_PAD, W-ANIM_PAD*2, H-ANIM_PAD*2
    );

    // ── AN Nodes (Floor 4 only)
    if(fl === 'Floor 4'){
      ctx.setLineDash([4,3]);
      ctx.strokeStyle='rgba(168,85,247,0.35)'; ctx.lineWidth=0.8;
      [[0,1],[1,2],[2,0]].forEach(function(p){
        ctx.beginPath();
        ctx.moveTo(lx(ANCHORS[p[0]].x),ly(ANCHORS[p[0]].y));
        ctx.lineTo(lx(ANCHORS[p[1]].x),ly(ANCHORS[p[1]].y));
        ctx.stroke();
      });
      ctx.setLineDash([]);
      ANCHORS.forEach(function(an){
        var ax=lx(an.x),ay=ly(an.y),s=5;
        ctx.shadowColor='#a855f7'; ctx.shadowBlur=8;
        ctx.beginPath(); ctx.moveTo(ax,ay-s); ctx.lineTo(ax+s,ay+s); ctx.lineTo(ax-s,ay+s);
        ctx.closePath(); ctx.fillStyle='#a855f7'; ctx.fill();
        ctx.strokeStyle='#e9d5ff'; ctx.lineWidth=0.7; ctx.stroke();
        ctx.shadowBlur=0;
        ctx.fillStyle='rgba(255,255,255,0.85)';
        ctx.font='6.5px IBM Plex Mono,monospace'; ctx.textAlign='center';
        ctx.fillText(an.id, ax, ay-s-3);
      });
    }

    // ── Seed PR dots
    var flPRs = PR_DB.filter(function(p){ return p.floor===fl; });
    flPRs.forEach(function(pr){
      if(mob && pr.id===mob.id) return;
      ctx.beginPath(); ctx.arc(lx(pr.x),ly(pr.y),2.5,0,Math.PI*2);
      ctx.fillStyle='rgba(96,165,250,0.35)'; ctx.fill();
    });

    // ── Particles (hot→red, cool→green)
    var maxCost=Math.max.apply(null,particles.map(function(p){return p.bc||1;}));
    particles.forEach(function(p){
      var ratio=maxCost>0?Math.min(1,(p.bc||0)/maxCost):0;
      var r=Math.round(239*ratio+34*(1-ratio));
      var gv=Math.round(68*ratio+197*(1-ratio));
      var bv=Math.round(68*ratio+94*(1-ratio));
      ctx.beginPath(); ctx.arc(lx(p.x),ly(p.y),3,0,Math.PI*2);
      ctx.fillStyle='rgba('+r+','+gv+','+bv+',0.85)'; ctx.fill();
    });

    // ── gbest trail
    var ts=Math.max(0,animState.history.length-6);
    for(var ti=ts; ti<animState.history.length-1; ti++){
      var h1=animState.history[ti],h2=animState.history[ti+1];
      ctx.beginPath(); ctx.moveTo(lx(h1.gbest.x),ly(h1.gbest.y));
      ctx.lineTo(lx(h2.gbest.x),ly(h2.gbest.y));
      ctx.strokeStyle='rgba(250,204,21,0.5)'; ctx.lineWidth=1.5; ctx.stroke();
    }

    // ── gbest ★
    if(gbest){
      ctx.shadowColor='#facc15'; ctx.shadowBlur=14;
      ctx.beginPath(); ctx.arc(lx(gbest.x),ly(gbest.y),5,0,Math.PI*2);
      ctx.fillStyle='#facc15'; ctx.fill(); ctx.shadowBlur=0;
      ctx.fillStyle='rgba(255,255,255,0.9)';
      ctx.font='bold 7px IBM Plex Mono,monospace'; ctx.textAlign='center';
      ctx.fillText('gbest',lx(gbest.x),ly(gbest.y)-9);
      ctx.fillText('('+gbest.x.toFixed(2)+','+gbest.y.toFixed(2)+')',lx(gbest.x),ly(gbest.y)+14);
    }

    // ── Mobile จริง
    if(mob){
      var mx=lx(mob.x),my=ly(mob.y);
      ctx.shadowColor='#22c55e'; ctx.shadowBlur=12;
      ctx.beginPath(); ctx.arc(mx,my,9,0,Math.PI*2);
      ctx.strokeStyle='rgba(34,197,94,0.35)'; ctx.lineWidth=2; ctx.stroke();
      ctx.beginPath(); ctx.arc(mx,my,5,0,Math.PI*2);
      ctx.fillStyle='#22c55e'; ctx.fill(); ctx.shadowBlur=0;
      ctx.fillStyle='rgba(255,255,255,0.9)';
      ctx.font='bold 7px IBM Plex Mono,monospace'; ctx.textAlign='center';
      ctx.fillText(mob.id,mx,my-13);
      ctx.fillText('(actual)',mx,my+17);
      if(gbest){
        var err=Math.sqrt((gbest.x-mob.x)**2+(gbest.y-mob.y)**2);
        ctx.beginPath(); ctx.moveTo(lx(gbest.x),ly(gbest.y)); ctx.lineTo(mx,my);
        ctx.strokeStyle='rgba(239,68,68,0.6)'; ctx.lineWidth=1.2;
        ctx.setLineDash([4,3]); ctx.stroke(); ctx.setLineDash([]);
        var emx=(lx(gbest.x)+mx)/2, emy=(ly(gbest.y)+my)/2;
        ctx.fillStyle='#ef4444'; ctx.font='bold 8px IBM Plex Mono,monospace';
        ctx.textAlign='center'; ctx.fillText(err.toFixed(2)+'m',emx,emy-5);
      }
    }

    // ── Iteration label
    ctx.fillStyle='rgba(0,0,0,0.55)'; ctx.fillRect(ANIM_PAD,H-ANIM_PAD-18,140,15);
    ctx.fillStyle='rgba(255,255,255,0.6)'; ctx.font='bold 8px IBM Plex Mono,monospace';
    ctx.textAlign='left';
    ctx.fillText('k='+k+' / '+animState.totalK+' | '+fl, ANIM_PAD+4, H-ANIM_PAD-6);
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
// ══════════════════════════════════════════════════════
// ══════════════════════════════════════════════════════
// FIX POSITION ENGINE — Real-world Validation Test
// Features: Multi-floor PSO, True Position, Test Log, Export
// ══════════════════════════════════════════════════════
(function(){

  var FP_ANS = [
    {id:'AN1', x:7.96,  y:9.07},
    {id:'AN3', x:28.10, y:9.57},
    {id:'AN4', x:8.77,  y:22.01},
  ];
  var ALL_FLOORS = ['Floor 4','Floor 3','Floor 2','Floor 1'];
  var fpResult   = null;
  var testLog    = [];

  // ── Floor selector toggle ─────────────────────────
  document.querySelectorAll('input[name="fp-floor-mode"]').forEach(function(r){
    r.addEventListener('change', function(){
      var manual = document.querySelector('input[name="fp-floor-mode"]:checked').value === 'manual';
      document.getElementById('fp-floor-select').style.display = manual ? 'block' : 'none';
    });
  });

  // ── Read params ───────────────────────────────────
  function getParams(){
    return {
      rssi: [
        parseFloat(document.getElementById('fp-rssi-an1').value)||0,
        parseFloat(document.getElementById('fp-rssi-an2').value)||0,
        parseFloat(document.getElementById('fp-rssi-an3').value)||0,
      ],
      pld0:  parseFloat(document.getElementById('fp-pld0').value)||-53,
      anH:   parseFloat(document.getElementById('fp-an-height').value)||10.5,
      mobH:  parseFloat(document.getElementById('fp-mob-height').value)||1.5,
      n:     parseInt(document.getElementById('fp-n').value)||60,
      K:     parseInt(document.getElementById('fp-k').value)||50,
      R:     parseInt(document.getElementById('fp-r').value)||5,
      grd:   parseFloat(document.getElementById('fp-grid').value)||2.0,
      floorMode: document.querySelector('input[name="fp-floor-mode"]:checked').value,
      manualFloor: document.getElementById('fp-floor-select').value,
    };
  }

  // ── Fitness (ใช้ per-floor η+FAF จาก model จริง) ──
  function fpFitness(px, py, rssiMeas, fl, pld0, anH, mobH){
    var dz  = Math.abs(anH - mobH);
    var eta = ETA_FLOOR[fl]  || 2.0;
    var faf = FAF_FLOOR[fl]  || 0;
    var sum = 0;
    FP_ANS.forEach(function(an, i){
      var d3   = Math.sqrt((px-an.x)**2 + (py-an.y)**2 + dz**2);
      var pred = pld0 - 10*eta*Math.log10(Math.max(d3,0.01)) - faf;
      sum += (pred - rssiMeas[i])**2;
    });
    return Math.sqrt(sum / FP_ANS.length);
  }

  // ── PSO for one floor ─────────────────────────────
  function psoOneFloor(rssiMeas, fl, p){
    var costHist = Array(p.K).fill(0);
    var allBest  = {x:G.MW/2, y:G.MH/2, cost:Infinity};

    for(var run=0; run<p.R; run++){
      var particles = [];
      for(var i=0; i<p.n; i++){
        var px=Math.random()*G.MW, py=Math.random()*G.MH;
        var cost=fpFitness(px,py,rssiMeas,fl,p.pld0,p.anH,p.mobH);
        particles.push({x:px,y:py,vx:0,vy:0,bx:px,by:py,bc:cost});
      }
      var gbest={x:particles[0].bx,y:particles[0].by,cost:particles[0].bc};
      particles.forEach(function(p2){if(p2.bc<gbest.cost){gbest={x:p2.bx,y:p2.by,cost:p2.bc};}});

      for(var k=0; k<p.K; k++){
        var w=0.9-(0.9-0.4)*(k/p.K);
        particles.forEach(function(pt){
          pt.vx=w*pt.vx+2*Math.random()*(pt.bx-pt.x)+2*Math.random()*(gbest.x-pt.x);
          pt.vy=w*pt.vy+2*Math.random()*(pt.by-pt.y)+2*Math.random()*(gbest.y-pt.y);
          pt.x=Math.max(0,Math.min(G.MW,pt.x+pt.vx));
          pt.y=Math.max(0,Math.min(G.MH,pt.y+pt.vy));
          var c=fpFitness(pt.x,pt.y,rssiMeas,fl,p.pld0,p.anH,p.mobH);
          if(c<pt.bc){pt.bc=c;pt.bx=pt.x;pt.by=pt.y;}
          if(c<gbest.cost){gbest={x:pt.x,y:pt.y,cost:c};}
        });
        costHist[k]+=gbest.cost/p.R;
      }
      if(gbest.cost<allBest.cost) allBest={x:gbest.x,y:gbest.y,cost:gbest.cost};
    }

    // Grid refinement
    var bx=allBest.x, by=allBest.y, bc=allBest.cost;
    for(var dx=-p.grd;dx<=p.grd;dx+=0.2){
      for(var dy=-p.grd;dy<=p.grd;dy+=0.2){
        var gx=Math.max(0,Math.min(G.MW,bx+dx));
        var gy=Math.max(0,Math.min(G.MH,by+dy));
        var gc=fpFitness(gx,gy,rssiMeas,fl,p.pld0,p.anH,p.mobH);
        if(gc<bc){bc=gc;bx=gx;by=gy;}
      }
    }
    return {x:bx, y:by, cost:bc, floor:fl, costHist:costHist};
  }

  // ── Main 2-Stage Run ─────────────────────────────
  function runFPSO(){
    var p   = getParams();
    var btn = document.getElementById('fp-btn-run');
    btn.textContent='\u23f3 \u0e01\u0e33\u0e25\u0e31\u0e07\u0e04\u0e33\u0e19\u0e27\u0e13...'; btn.disabled=true;

    document.getElementById('fp-stage-panel').style.display='block';
    document.getElementById('fp-result-cards').style.display='none';
    document.getElementById('fp-stage1-result').style.display='none';
    document.getElementById('fp-stage2-result').style.display='none';
    document.getElementById('fp-stage2-card').style.opacity='0.4';
    document.getElementById('fp-ambig-warn').style.display='none';
    document.getElementById('fp-stage1-status').textContent='\u23f3 \u0e01\u0e33\u0e25\u0e31\u0e07 detect \u0e0a\u0e31\u0e49\u0e19...';
    document.getElementById('fp-stage2-status').textContent='\u0e23\u0e2d Stage 1...';
    document.getElementById('fp-stage1-costs').innerHTML='';

    setTimeout(function(){
      var floorsToRun = (p.floorMode==='manual'&&p.manualFloor)
        ? [p.manualFloor] : ALL_FLOORS;

      // Stage 1: PSO เบาๆ ทุกชั้น
      var s1results={};
      var s1p={pld0:p.pld0,anH:p.anH,mobH:p.mobH,
               n:Math.min(30,p.n),K:Math.min(20,p.K),R:Math.min(3,p.R),grd:1.0};
      floorsToRun.forEach(function(fl){ s1results[fl]=psoOneFloor(p.rssi,fl,s1p); });

      var sorted=floorsToRun.slice().sort(function(a,b){
        return s1results[a].cost-s1results[b].cost;
      });
      var bestFl   = sorted[0];
      var bestCost = s1results[bestFl].cost;
      var secCost  = sorted.length>1 ? s1results[sorted[1]].cost : bestCost*2;
      var margin     = (secCost-bestCost)/Math.max(bestCost,0.001);
      var confidence = Math.min(100,Math.round(margin*200));
      var isAmbig    = confidence<20;

      // Stage 1 UI
      var costsHtml=sorted.map(function(fl){
        var r=s1results[fl],ib=fl===bestFl;
        return '<div style="display:flex;justify-content:space-between;padding:3px 0;'+
          'color:'+(ib?'#facc15':'#6b7a99')+';">'+
          '<span>'+(ib?'\u2605 ':'')+fl+'</span>'+
          '<span style="font-family:var(--mono);">'+r.cost.toFixed(3)+'</span></div>';
      }).join('');
      document.getElementById('fp-stage1-costs').innerHTML=costsHtml;
      document.getElementById('fp-stage1-floor').textContent=bestFl;
      document.getElementById('fp-stage1-status').textContent='\u2705 \u0e15\u0e23\u0e27\u0e08\u0e1e\u0e1a\u0e0a\u0e31\u0e49\u0e19';
      document.getElementById('fp-stage1-result').style.display='block';
      document.getElementById('fp-conf-bar').style.width=confidence+'%';
      document.getElementById('fp-conf-bar').style.background=
        confidence>=60?'#22c55e':confidence>=30?'#f59e0b':'#ef4444';
      document.getElementById('fp-conf-pct').textContent=confidence+'%';
      document.getElementById('fp-conf-pct').style.color=
        confidence>=60?'#22c55e':confidence>=30?'#f59e0b':'#ef4444';
      if(isAmbig){
        document.getElementById('fp-ambig-warn').style.display='block';
        document.getElementById('fp-ambig-warn').innerHTML=
          '\u26a0\ufe0f \u0e44\u0e21\u0e48\u0e41\u0e19\u0e48\u0e43\u0e08 ('+confidence+'%) \u2014 \u0e2d\u0e32\u0e08\u0e40\u0e1b\u0e47\u0e19 '+sorted[0]+' \u0e2b\u0e23\u0e37\u0e2d '+(sorted[1]||'?')+
          '<br><span style="font-size:9px;">cost \u0e15\u0e48\u0e32\u0e07\u0e01\u0e31\u0e19\u0e19\u0e49\u0e2d\u0e22 \u0e25\u0e2d\u0e07 measure \u0e2d\u0e35\u0e01\u0e04\u0e23\u0e31\u0e49\u0e07</span>';
      }

      document.getElementById('fp-stage2-card').style.opacity='1';
      document.getElementById('fp-stage2-status').textContent='\u23f3 PSO \u0e01\u0e33\u0e25\u0e31\u0e07\u0e2b\u0e32\u0e15\u0e33\u0e41\u0e2b\u0e19\u0e48\u0e07...';

      setTimeout(function(){
        var s2result=psoOneFloor(p.rssi,bestFl,p);
        document.getElementById('fp-stage2-xy').textContent=
          'X='+s2result.x.toFixed(2)+'m  Y='+s2result.y.toFixed(2)+'m';
        document.getElementById('fp-stage2-cost').textContent='RMSE: '+s2result.cost.toFixed(3)+' dBm';
        document.getElementById('fp-stage2-result').style.display='block';
        document.getElementById('fp-stage2-status').textContent='\u2705 \u0e2b\u0e32\u0e15\u0e33\u0e41\u0e2b\u0e19\u0e48\u0e07\u0e2a\u0e33\u0e40\u0e23\u0e47\u0e08';

        fpResult=s2result;
        fpResult.floorResults=s1results;
        fpResult.confidence=confidence;
        fpResult.isAmbig=isAmbig;
        fpResult.rssiMeas=p.rssi;
        fpResult.pld0=p.pld0;
        fpResult.anH=p.anH;
        fpResult.mobH=p.mobH;

        document.getElementById('fp-res-x').textContent=s2result.x.toFixed(2)+' m';
        document.getElementById('fp-res-y').textContent=s2result.y.toFixed(2)+' m';
        document.getElementById('fp-res-floor').textContent=s2result.floor;
        document.getElementById('fp-res-cost').textContent=s2result.cost.toFixed(3);
        document.getElementById('fp-result-cards').style.display='block';
        document.getElementById('fp-map-label').textContent=
          '\u0e41\u0e1c\u0e19\u0e1c\u0e31\u0e07 '+s2result.floor+' \u2014 conf.'+confidence+'%';

        if(p.floorMode==='auto'){
          var ch2=ALL_FLOORS.map(function(fl){
            var r=s1results[fl]; if(!r) return '';
            var ib=fl===bestFl;
            return '<span style="margin-right:12px;color:'+(ib?'#facc15':'#6b7a99')+';">'+
              fl+': <b>'+r.cost.toFixed(3)+'</b>'+(ib?' \u2605':'')+'</span>';
          }).join('');
          document.getElementById('fp-floor-costs').innerHTML='\ud83d\udcca Cost: '+ch2;
          document.getElementById('fp-floor-costs').style.display='block';
        }

        var dz=Math.abs(p.anH-p.mobH);
        var eta=ETA_FLOOR[bestFl]||2.0,faf=FAF_FLOOR[bestFl]||0;
        var dlines=FP_ANS.map(function(an,i){
          var d3=Math.sqrt((s2result.x-an.x)**2+(s2result.y-an.y)**2+dz**2);
          var pred=p.pld0-10*eta*Math.log10(Math.max(d3,0.01))-faf;
          return an.id+' pred='+pred.toFixed(1)+' meas='+p.rssi[i]+' \u0394='+(p.rssi[i]-pred).toFixed(1);
        }).join(' | ');
        document.getElementById('fp-dist-info').textContent='\ud83d\udcd0 '+dlines;

        var trueX=parseFloat(document.getElementById('fp-true-x').value);
        var trueY=parseFloat(document.getElementById('fp-true-y').value);
        var trueFl=document.getElementById('fp-true-floor').value;
        fpResult.trueX=trueX; fpResult.trueY=trueY; fpResult.trueFl=trueFl;
        var ec_el=document.getElementById('fp-error-card');
        var ed_el=document.getElementById('fp-error-detail');
        if(!isNaN(trueX)&&trueX!==''&&!isNaN(trueY)&&trueY!==''){
          var xyErr=Math.sqrt((s2result.x-trueX)**2+(s2result.y-trueY)**2);
          var flOk=trueFl?(s2result.floor===trueFl):null;
          document.getElementById('fp-res-error').textContent=xyErr.toFixed(2)+' m';
          ec_el.style.display='block';
          var ec=xyErr<=3?'#22c55e':xyErr<=6?'#f59e0b':'#ef4444';
          var fs=flOk===null?'':flOk?'  \u2705 \u0e0a\u0e31\u0e49\u0e19\u0e16\u0e39\u0e01':'  \u274c \u0e0a\u0e31\u0e49\u0e19\u0e1c\u0e34\u0e14 (\u0e08\u0e23\u0e34\u0e07='+trueFl+')';
          ed_el.innerHTML='\ud83d\udccd True: ('+trueX.toFixed(2)+', '+trueY.toFixed(2)+') '+trueFl+'<br>'+
            '\ud83c\udfaf Est: ('+s2result.x.toFixed(2)+', '+s2result.y.toFixed(2)+') '+s2result.floor+'<br>'+
            '\ud83d\udccf XY Error: <b style="color:'+ec+'">'+xyErr.toFixed(2)+'m</b>'+fs+
            ' | Conf: <b style="color:'+(confidence>=60?'#22c55e':confidence>=30?'#f59e0b':'#ef4444')+'">'+confidence+'%</b>';
          ed_el.style.display='block';
          fpResult.xyErr=xyErr; fpResult.flOk=flOk;
        } else {
          ec_el.style.display='none'; ed_el.style.display='none'; fpResult.xyErr=null;
        }

        mkChart('fp-conv-chart',{type:'line',
          data:{labels:s2result.costHist.map(function(_,i){return i+1;}),
            datasets:[
              {label:'Stage1 detect ('+bestFl+')',data:s1results[bestFl].costHist,
               borderColor:'rgba(168,85,247,0.55)',backgroundColor:'transparent',
               pointRadius:0,tension:0.3,borderWidth:1,borderDash:[4,3]},
              {label:'Stage2 position ('+bestFl+')',data:s2result.costHist,
               borderColor:'rgba(250,204,21,0.9)',backgroundColor:'rgba(250,204,21,0.06)',
               pointRadius:0,tension:0.3,borderWidth:2,fill:true}
            ]},
          options:{responsive:true,maintainAspectRatio:false,
            plugins:{legend:{display:true,labels:{color:'#6b7a99',font:{size:9}}}},
            scales:{y:{grid:{color:'rgba(255,255,255,0.04)'},ticks:{color:'#6b7a99',font:{size:9}}},
              x:{grid:{display:false},ticks:{color:'#6b7a99',font:{size:9},maxTicksLimit:10}}}}
        });

        drawFPMap(s2result.x,s2result.y,s2result.floor,
          isNaN(trueX)?null:trueX,isNaN(trueY)?null:trueY);
        updatePreview();
        document.getElementById('fp-btn-save').style.display='block';
        btn.textContent='\u26a1 Run 2-Stage PSO \u2014 \u0e15\u0e23\u0e27\u0e08\u0e0a\u0e31\u0e49\u0e19 \u2192 \u0e2b\u0e32\u0e15\u0e33\u0e41\u0e2b\u0e19\u0e48\u0e07';
        btn.disabled=false;
      },30);
    },30);
  }


  // ── Draw map ──────────────────────────────────────
  function drawFPMap(bx, by, fl, trueX, trueY){
    fl = fl || 'Floor 4';
    var el  = document.getElementById('fp-map-canvas');
    var W   = el.parentElement.clientWidth || 500;
    var H   = Math.round(W * G.MH/G.MW);
    var dpr = window.devicePixelRatio||1;
    el.width=W*dpr; el.height=H*dpr; el.style.height=H+'px';
    var ctx = el.getContext('2d');
    ctx.scale(dpr,dpr);
    var PAD=20, dW=W-PAD*2, dH=H-PAD*2;
    function cx(rx){return PAD+(G.MW-rx)/G.MW*dW;}
    function cy(ry){return PAD+(G.MH-ry)/G.MH*dH;}

    drawFloorPlan(ctx, fl, cx, cy, W, H, PAD, dW, dH);

    // AN nodes + range circles
    var p    = getParams();
    var dz   = Math.abs(p.anH-p.mobH);
    var eta  = ETA_FLOOR[fl]||2.0;
    var colors=['rgba(59,130,246,0.5)','rgba(16,185,129,0.5)','rgba(245,158,11,0.5)'];
    FP_ANS.forEach(function(an,i){
      // range circle
      var d3 = Math.pow(10,(p.pld0-p.rssi[i]-FAF_FLOOR[fl])/(10*eta));
      var d2 = Math.sqrt(Math.max(0,d3*d3-dz*dz));
      ctx.beginPath(); ctx.arc(cx(an.x),cy(an.y), d2/G.MW*dW, 0,Math.PI*2);
      ctx.strokeStyle=colors[i]; ctx.lineWidth=0.9;
      ctx.setLineDash([3,3]); ctx.stroke(); ctx.setLineDash([]);
      // triangle
      var ax=cx(an.x),ay=cy(an.y),s=5;
      ctx.shadowColor='#a855f7'; ctx.shadowBlur=8;
      ctx.beginPath(); ctx.moveTo(ax,ay-s); ctx.lineTo(ax+s,ay+s); ctx.lineTo(ax-s,ay+s);
      ctx.closePath(); ctx.fillStyle='#a855f7'; ctx.fill(); ctx.shadowBlur=0;
      ctx.fillStyle='rgba(240,200,255,0.9)';
      ctx.font='6.5px IBM Plex Mono,monospace'; ctx.textAlign='center';
      ctx.fillText(an.id, ax, ay-s-3);
    });

    // True position (green circle)
    if(trueX!==null && trueY!==null && !isNaN(trueX) && !isNaN(trueY)){
      var tx=cx(trueX), ty=cy(trueY);
      ctx.shadowColor='#22c55e'; ctx.shadowBlur=14;
      ctx.beginPath(); ctx.arc(tx,ty,8,0,Math.PI*2);
      ctx.strokeStyle='#22c55e'; ctx.lineWidth=2; ctx.stroke();
      ctx.beginPath(); ctx.arc(tx,ty,3,0,Math.PI*2);
      ctx.fillStyle='#22c55e'; ctx.fill(); ctx.shadowBlur=0;
      ctx.fillStyle='rgba(255,255,255,0.9)';
      ctx.font='bold 7px IBM Plex Mono,monospace'; ctx.textAlign='center';
      ctx.fillText('True', tx, ty-13);
      // error line
      var ex=cx(bx), ey=cy(by);
      ctx.beginPath(); ctx.moveTo(tx,ty); ctx.lineTo(ex,ey);
      ctx.strokeStyle='rgba(239,68,68,0.55)'; ctx.lineWidth=1.2;
      ctx.setLineDash([4,3]); ctx.stroke(); ctx.setLineDash([]);
    }

    // Estimated position (star)
    var px=cx(bx), py=cy(by);
    ctx.shadowColor='#facc15'; ctx.shadowBlur=16;
    ctx.beginPath(); ctx.arc(px,py,7,0,Math.PI*2);
    ctx.fillStyle='#facc15'; ctx.fill(); ctx.shadowBlur=0;
    ctx.strokeStyle='#fff'; ctx.lineWidth=1.2; ctx.stroke();
    ctx.fillStyle='rgba(255,255,255,0.95)';
    ctx.font='bold 7px IBM Plex Mono,monospace'; ctx.textAlign='center';
    ctx.fillText('★ Est.', px, py-12);
    ctx.fillText('('+bx.toFixed(2)+','+by.toFixed(2)+')', px, py+16);
  }

  // ── Realtime preview ──────────────────────────────
  function updatePreview(){
    var p    = getParams();
    var fl   = (p.floorMode==='manual'&&p.manualFloor) ? p.manualFloor : 'Floor 4';
    var dz   = Math.abs(p.anH-p.mobH);
    var eta  = ETA_FLOOR[fl]||2.0;
    var faf  = FAF_FLOOR[fl]||0;
    var refX = fpResult ? fpResult.x : FP_ANS[0].x;
    var refY = fpResult ? fpResult.y : FP_ANS[0].y;
    var lines = FP_ANS.map(function(an,i){
      var d3   = Math.sqrt((refX-an.x)**2+(refY-an.y)**2+dz**2);
      var pred = p.pld0-10*eta*Math.log10(Math.max(d3,0.01))-faf;
      var diff = (p.rssi[i]-pred);
      var dc   = Math.abs(diff)<3?'#22c55e':Math.abs(diff)<6?'#f59e0b':'#ef4444';
      return an.id+': Pred=<span style="color:var(--blue)">'+pred.toFixed(1)+'</span>'+
        ' | Meas=<span style="color:var(--amber)">'+p.rssi[i].toFixed(1)+'</span>'+
        ' | Δ=<span style="color:'+dc+'">'+diff.toFixed(1)+'</span>';
    });
    document.getElementById('fp-predict-preview').innerHTML = lines.join('<br>');
  }

  // ── Test Log ──────────────────────────────────────
  function saveTestCase(){
    if(!fpResult) return;
    var n = testLog.length+1;
    var entry = {
      n:n,
      estX:fpResult.x, estY:fpResult.y, estFloor:fpResult.floor,
      trueX:fpResult.trueX, trueY:fpResult.trueY, trueFl:fpResult.trueFl,
      xyErr:fpResult.xyErr, flOk:fpResult.flOk, cost:fpResult.cost,
    };
    testLog.push(entry);
    renderLog();
  }

  function renderLog(){
    if(!testLog.length) return;
    document.getElementById('fp-test-log-empty').style.display='none';
    document.getElementById('fp-test-log').style.display='block';
    var tbody = document.getElementById('fp-log-body');
    tbody.innerHTML = testLog.map(function(e){
      var errStr = e.xyErr!==null&&e.xyErr!==undefined ?
        '<span style="color:'+(e.xyErr<=3?'#22c55e':e.xyErr<=6?'#f59e0b':'#ef4444')+';">'+e.xyErr.toFixed(2)+'m</span>' : '—';
      var truePos = (e.trueX&&e.trueY) ? e.trueX.toFixed(2)+','+e.trueY.toFixed(2) : '—';
      var flOkStr = e.flOk===null||e.flOk===undefined ? '—' : (e.flOk?'✅':'❌');
      return '<tr style="border-bottom:1px solid rgba(255,255,255,0.04);">'+
        '<td style="padding:4px 8px;color:#6b7a99;">'+e.n+'</td>'+
        '<td style="padding:4px 8px;color:#f59e0b;">'+e.estFloor+'</td>'+
        '<td style="padding:4px 8px;font-family:var(--mono);">'+e.estX.toFixed(2)+','+e.estY.toFixed(2)+'</td>'+
        '<td style="padding:4px 8px;color:#22c55e;">'+(e.trueFl||'—')+'</td>'+
        '<td style="padding:4px 8px;font-family:var(--mono);">'+truePos+'</td>'+
        '<td style="padding:4px 8px;">'+errStr+'</td>'+
        '<td style="padding:4px 8px;">'+flOkStr+'</td>'+
        '<td style="padding:4px 8px;font-family:var(--mono);">'+e.cost.toFixed(3)+'</td>'+
        '</tr>';
    }).join('');

    // Summary
    var withErr = testLog.filter(function(e){return e.xyErr!==null&&e.xyErr!==undefined;});
    var withFl  = testLog.filter(function(e){return e.flOk!==null&&e.flOk!==undefined;});
    var avgErr  = withErr.length ? (withErr.reduce(function(s,e){return s+e.xyErr;},0)/withErr.length).toFixed(2) : '—';
    var flAcc   = withFl.length  ? ((withFl.filter(function(e){return e.flOk;}).length/withFl.length)*100).toFixed(0)+'%' : '—';
    var le3     = withErr.filter(function(e){return e.xyErr<=3;}).length;
    var le6     = withErr.filter(function(e){return e.xyErr<=6;}).length;
    document.getElementById('fp-log-summary').innerHTML =
      'Tests: '+testLog.length+' | Avg Error: <b style="color:#facc15;">'+avgErr+'m</b>'+
      ' | Floor Acc: <b style="color:#22c55e;">'+flAcc+'</b>'+
      (withErr.length?' | ≤3m: '+le3+'/'+withErr.length+' | ≤6m: '+le6+'/'+withErr.length:'');
  }

  function clearLog(){
    testLog=[];
    document.getElementById('fp-test-log-empty').style.display='block';
    document.getElementById('fp-test-log').style.display='none';
    document.getElementById('fp-log-body').innerHTML='';
    document.getElementById('fp-log-summary').innerHTML='';
  }

  // ── Event bindings ────────────────────────────────
  document.getElementById('fp-btn-run').addEventListener('click', runFPSO);
  document.getElementById('fp-btn-save').addEventListener('click', saveTestCase);
  document.getElementById('fp-btn-clear-log').addEventListener('click', clearLog);

  ['fp-rssi-an1','fp-rssi-an2','fp-rssi-an3','fp-pld0','fp-an-height','fp-mob-height']
  .forEach(function(id){
    document.getElementById(id).addEventListener('input', function(){
      updatePreview();
      if(fpResult) drawFPMap(fpResult.x,fpResult.y,fpResult.floor,fpResult.trueX,fpResult.trueY);
    });
  });

  updatePreview();

})(); // end Fix Position engine

})();


})();
</script>
</body>
</html>
