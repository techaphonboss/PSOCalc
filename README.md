<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PSOCalc - Professional KNN IPS Live Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&family=Noto+Sans+Thai:wght@300;400;500;700;900&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    
    <style>
        body { font-family: 'Noto Sans Thai', 'JetBrains Mono', sans-serif; }
        .code-font { font-family: 'JetBrains Mono', monospace; }
    </style>
</head>
<body class="bg-[#0b0f19] text-[#f3f4f6] min-h-screen relative overflow-x-hidden">

    <div class="absolute top-0 left-1/4 w-96 h-96 bg-blue-500/10 rounded-full blur-3xl pointer-events-none"></div>
    <div class="absolute top-0 right-1/4 w-96 h-96 bg-indigo-500/10 rounded-full blur-3xl pointer-events-none"></div>

    <div class="container mx-auto px-6 py-10 relative z-10">
        
        <header class="mb-10 flex flex-col md:flex-row md:items-center md:justify-between border-b border-gray-800 pb-6">
            <div>
                <div class="flex items-center gap-3 mb-2">
                    <span class="bg-blue-500/10 text-blue-400 text-xs font-bold uppercase tracking-wider px-3 py-1 rounded-md border border-blue-500/20">LIVE ENGINE v3.5</span>
                </div>
                <h1 class="text-3xl font-black tracking-tight text-white">
                    <span class="bg-gradient-to-r from-blue-500 to-indigo-400 bg-clip-text text-transparent">PSOCalc</span> Dashboard
                </h1>
                <p class="text-gray-400 text-sm mt-1">ระบบวิเคราะห์ ประมวลผล และส่งออกข้อมูลอัลกอริทึม KNN ค้นหาพิกัดระดับชั้นภายในอาคาร (IPS)</p>
            </div>
            <div class="mt-4 md:mt-0">
                <span class="text-xs text-gray-500 block uppercase font-bold tracking-widest">Engine Status</span>
                <span id="engine-status" class="inline-flex items-center gap-2 text-xs font-medium bg-amber-500/10 text-amber-400 px-3 py-1 rounded-full border border-amber-500/20 mt-1">
                    <span class="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span> Ready for Analytics
                </span>
            </div>
        </header>

        <div class="mb-10">
            <div id="drop-zone" class="relative group border-2 border-dashed border-gray-700 bg-[#111827]/50 backdrop-blur-md rounded-2xl p-10 text-center hover:border-blue-500 transition-all duration-300 cursor-pointer shadow-xl">
                <input type="file" id="file-input" accept=".xlsx, .xls, .csv" class="hidden">
                <div>
                    <div class="w-16 h-16 bg-gray-800/80 rounded-xl flex items-center justify-center mx-auto mb-4 border border-gray-700 group-hover:bg-blue-500/10 group-hover:border-blue-500/50 transition-all duration-300">
                        <svg class="w-8 h-8 text-gray-400 group-hover:text-blue-400 transition-colors" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                    </div>
                    <h3 class="text-lg font-bold text-white mb-1 group-hover:text-blue-400 transition-colors">ลากและหย่อนไฟล์ข้อมูลดิบที่นี่เพื่อคำนวณสด</h3>
                    <p class="text-gray-400 text-sm max-w-md mx-auto mb-2">รองรับไฟล์แผนผัง <span class="text-blue-400 font-semibold">.xlsx / .csv</span> ตัวสรุปค่าสัญญาณดิบ</p>
                </div>
            </div>
        </div>

        <div id="analysis-section" class="hidden space-y-10">
            
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-5">
                <div class="bg-[#111827]/70 backdrop-blur-md p-6 rounded-xl border border-gray-800 shadow-lg relative overflow-hidden">
                    <div class="absolute top-0 left-0 w-1 h-full bg-[#2563eb]"></div>
                    <span class="text-xs font-bold text-blue-400 uppercase block mb-1">Meas. (Top 3 Avg)</span>
                    <h3 class="text-3xl font-black text-white code-font" id="acc-m-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">คำนวณแบบ 3 ค่าน้อยสุดหาร 3</p>
                </div>
                <div class="bg-[#111827]/70 backdrop-blur-md p-6 rounded-xl border border-gray-800 shadow-lg relative overflow-hidden">
                    <div class="absolute top-0 left-0 w-1 h-full bg-[#ea580c]"></div>
                    <span class="text-xs font-bold text-orange-400 uppercase block mb-1">Meas. (Grand Avg)</span>
                    <h3 class="text-3xl font-black text-white code-font" id="acc-m-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">คำนวณแบบค่าเฉลี่ยรวมทั้งชั้น</p>
                </div>
                <div class="bg-[#111827]/70 backdrop-blur-md p-6 rounded-xl border border-gray-800 shadow-lg relative overflow-hidden">
                    <div class="absolute top-0 left-0 w-1 h-full bg-[#10b981]"></div>
                    <span class="text-xs font-bold text-emerald-400 uppercase block mb-1">RSSI (Top 3 Avg)</span>
                    <h3 class="text-3xl font-black text-white code-font" id="acc-r-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">คำนวณแบบ 3 ค่าน้อยสุดหาร 3</p>
                </div>
                <div class="bg-[#111827]/70 backdrop-blur-md p-6 rounded-xl border border-gray-800 shadow-lg relative overflow-hidden">
                    <div class="absolute top-0 left-0 w-1 h-full bg-[#db2777]"></div>
                    <span class="text-xs font-bold text-pink-400 uppercase block mb-1">RSSI (Grand Avg)</span>
                    <h3 class="text-3xl font-black text-white code-font" id="acc-r-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">คำนวณแบบค่าเฉลี่ยรวมทั้งชั้น</p>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="bg-[#111827]/70 backdrop-blur-md p-6 rounded-2xl border border-gray-800 shadow-xl">
                    <h3 class="text-sm font-bold text-gray-300 uppercase tracking-wider mb-4">🏆 อัตราความถูกต้องรวมในแต่ละโมเดล (%)</h3>
                    <div class="h-80 relative"><canvas id="accuracyChart"></canvas></div>
                </div>
                <div class="bg-[#111827]/70 backdrop-blur-md p-6 rounded-2xl border border-gray-800 shadow-xl">
                    <h3 class="text-sm font-bold text-gray-300 uppercase tracking-wider mb-4">📍 ปริมาณความผิดพลาดคลาดเคลื่อนแยกตามชั้นและวิธีคำนวณ (จุด)</h3>
                    <div class="h-80 relative"><canvas id="errorFloorChart"></canvas></div>
                </div>
            </div>

            <div class="bg-[#111827]/70 backdrop-blur-md rounded-2xl border border-gray-800 shadow-xl overflow-hidden">
                <div class="p-5 border-b border-gray-800 bg-gray-900/30 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                    <div>
                        <h3 class="text-base font-bold text-white">📋 ตารางผลการคำนวณระดับชั้นจากสมการ KNN ในเบราว์เซอร์</h3>
                        <p class="text-xs text-gray-400 mt-0.5">กดปุ่มเพื่อส่งออกตารางสรุปผลลัพธ์นี้ไปเป็นไฟล์สเปรดชีต Excel</p>
                    </div>
                    <div class="flex items-center gap-3">
                        <button id="btn-export" class="bg-gradient-to-r from-green-600 to-emerald-500 hover:from-green-500 hover:to-emerald-400 text-white font-bold text-xs px-4 py-2.5 rounded-lg border border-emerald-600/30 flex items-center gap-2 shadow-lg transition-all duration-200">
                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                            EXPORT TO EXCEL (.xlsx)
                        </button>
                        <span class="text-xs bg-blue-500/10 text-blue-400 px-3 py-2 rounded-lg border border-blue-500/20 font-bold code-font" id="total-records">0 PARTICLES</span>
                    </div>
                </div>
                <div class="overflow-x-auto max-h-[500px]">
                    <table class="w-full text-left border-collapse text-xs md:text-sm">
                        <thead>
                            <tr class="bg-[#1f2937]/50 text-gray-400 uppercase font-bold sticky top-0 border-b border-gray-800 backdrop-blur-md">
                                <th class="p-4 w-32">Floor จริง</th>
                                <th class="p-4">Particle ID</th>
                                <th class="p-4 bg-blue-950/20 text-blue-400">Meas (Top 3)</th>
                                <th class="p-4 bg-orange-950/10 text-orange-400">Meas (Avg รวม)</th>
                                <th class="p-4 bg-emerald-950/20 text-emerald-400">RSSI (Top 3)</th>
                                <th class="p-4 bg-pink-950/10 text-pink-400">RSSI (Avg รวม)</th>
                            </tr>
                        </thead>
                        <tbody id="table-output" class="divide-y divide-gray-800/60 code-font"></tbody>
                    </table>
                </div>
            </div>

        </div>
    </div>

    <script>
        const dropZone = document.getElementById('drop-zone');
        const fileInput = document.getElementById('file-input');
        const engineStatus = document.getElementById('engine-status');
        const analysisSection = document.getElementById('analysis-section');
        const btnExport = document.getElementById('btn-export');

        // ประกาศตัวแปรเก็บล็อกแบบ Global ไว้นอกขอบเขตฟังก์ชันเพื่อไม่ให้ค่าสูญหาย
        let globalLogs = []; 

        dropZone.addEventListener('click', () => fileInput.click());
        dropZone.addEventListener('dragover', (e) => { e.preventDefault(); dropZone.classList.add('border-blue-500', 'bg-blue-500/5'); });
        dropZone.addEventListener('dragleave', () => dropZone.classList.remove('border-blue-500', 'bg-blue-500/5'));
        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.classList.remove('border-blue-500', 'bg-blue-500/5');
            if (e.dataTransfer.files.length) handleLiveFile(e.dataTransfer.files[0]);
        });
        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length) handleLiveFile(e.target.files[0]);
        });

        function handleLiveFile(file) {
            engineStatus.className = "inline-flex items-center gap-2 text-xs font-medium bg-blue-500/10 text-blue-400 px-3 py-1 rounded-full border border-blue-500/20 mt-1";
            engineStatus.innerHTML = `<span class="w-2 h-2 rounded-full bg-blue-400 animate-spin"></span> Processing Live Engine...`;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const firstSheetName = workbook.SheetNames[0];
                    const worksheet = workbook.Sheets[firstSheetName];
                    const rawRows = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                    
                    executeKnnEngine(rawRows);
                } catch (err) {
                    engineStatus.className = "inline-flex items-center gap-2 text-xs font-medium bg-red-500/10 text-red-400 px-3 py-1 rounded-full border border-red-500/20 mt-1";
                    engineStatus.innerHTML = `❌ Error Parsing Data`;
                }
            };
            reader.readAsArrayBuffer(file);
        }

        let accChart = null;
        let errChart = null;

        function executeKnnEngine(rawRows) {
            let cleanData = [];
            
            for (let i = 0; i < rawRows.length; i++) {
                let row = rawRows[i];
                if (row && row[0] && String(row[0]).trim().startsWith('Floor')) {
                    let floorStr = String(row[0]).trim();
                    let partStr = row[1] ? String(row[1]).trim() : '';
                    
                    if (partStr.toLowerCase().startsWith('particle')) continue;

                    let m1 = parseFloat(row[2]); let m2 = parseFloat(row[3]); let m3 = parseFloat(row[4]);
                    let r1 = parseFloat(row[5]); let r2 = parseFloat(row[6]); let r3 = parseFloat(row[7]);

                    if (isNaN(m1) || isNaN(m2) || isNaN(m3)) continue;

                    cleanData.push({
                        actualFloor: floorStr,
                        particleId: partStr,
                        meas: [m1, m2, m3],
                        rssi: [isNaN(r1) ? 0 : r1, isNaN(r2) ? 0 : r2, isNaN(r3) ? 0 : r3]
                    });
                }
            }

            if (cleanData.length === 0) {
                engineStatus.innerHTML = `❌ No valid data rows found`;
                return;
            }

            // รีเซ็ตข้อมูลเก็บล็อกระดับ Global ทุกครั้งที่มีการรันข้อมูลใหม่
            globalLogs = [];
            let counts = { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0, total: cleanData.length };
            
            let detailErrors = {
                'Floor 1': { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0 },
                'Floor 2': { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0 },
                'Floor 3': { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0 },
                'Floor 4': { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0 }
            };

            document.getElementById('total-records').textContent = `${cleanData.length} PARTICLES`;

            cleanData.forEach((mobile) => {
                let actualFloor = mobile.actualFloor;
                let mDistances = { 'Floor 1': [], 'Floor 2': [], 'Floor 3': [], 'Floor 4': [] };
                let rDistances = { 'Floor 1': [], 'Floor 2': [], 'Floor 3': [], 'Floor 4': [] };

                cleanData.forEach((target) => {
                    let targetFloor = target.actualFloor;

                    let distM = Math.sqrt(Math.pow(mobile.meas[0]-target.meas[0],2) + Math.pow(mobile.meas[1]-target.meas[1],2) + Math.pow(mobile.meas[2]-target.meas[2],2));
                    let distR = Math.sqrt(Math.pow(mobile.rssi[0]-target.rssi[0],2) + Math.pow(mobile.rssi[1]-target.rssi[1],2) + Math.pow(mobile.rssi[2]-target.rssi[2],2));

                    mDistances[targetFloor].push(distM);
                    rDistances[targetFloor].push(distR);
                });

                function runPredictionLive(distanceMap, isTop3Mode) {
                    let finalFloorScores = {};
                    
                    for (let fl in distanceMap) {
                        let dList = distanceMap[fl];
                        if (isTop3Mode) {
                            let sorted = dList.slice().sort((a, b) => a - b);
                            let filtered = sorted.filter(v => v > 0);
                            if (filtered.length < 3) filtered = sorted;
                            finalFloorScores[fl] = filtered.length >= 3 ? (filtered[0] + filtered[1] + filtered[2]) / 3 : (filtered[0] || 999);
                        } else {
                            let totalSum = dList.reduce((a, b) => a + b, 0);
                            finalFloorScores[fl] = dList.length > 0 ? (totalSum / dList.length) : 999;
                        }
                    }

                    let predicted = Object.keys(finalFloorScores).reduce((a, b) => finalFloorScores[a] < finalFloorScores[b] ? a : b);
                    return predicted === actualFloor ? "ถูก" : `ชั้น ${predicted.replace('Floor ', '')}`;
                }

                let p_m_top3 = runPredictionLive(mDistances, true);
                let p_m_avg  = runPredictionLive(mDistances, false);
                let p_r_top3 = runPredictionLive(rDistances, true);
                let p_r_avg  = runPredictionLive(rDistances, false);

                if (p_m_top3 === "ถูก") counts.mTop3++; else detailErrors[actualFloor].mTop3++;
                if (p_m_avg === "ถูก") counts.mAvg++; else detailErrors[actualFloor].mAvg++;
                if (p_r_top3 === "ถูก") counts.rTop3++; else detailErrors[actualFloor].rTop3++;
                if (p_r_avg === "ถูก") counts.rAvg++; else detailErrors[actualFloor].rAvg++;

                // ผลักออบเจกต์คำตอบเข้าสู่ตัวแปรภายนอกโดยตรง
                globalLogs.push({
                    floor: actualFloor, id: mobile.particleId,
                    mTop3: p_m_top3, mAvg: p_m_avg, rTop3: p_r_top3, rAvg: p_r_avg
                });
            });

            const pct = (v, t) => ((v / t) * 100).toFixed(2) + "%";
            document.getElementById('acc-m-top3').textContent = pct(counts.mTop3, counts.total);
            document.getElementById('acc-m-avg').textContent = pct(counts.mAvg, counts.total);
            document.getElementById('acc-r-top3').textContent = pct(counts.rTop3, counts.total);
            document.getElementById('acc-r-avg').textContent = pct(counts.rAvg, counts.total);

            const tbody = document.getElementById('table-output');
            tbody.innerHTML = "";
            globalLogs.forEach(log => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-gray-800/40 transition-colors border-b border-gray-800/50 text-xs sm:text-sm";
                
                const badgeStyle = (val) => val === 'ถูก' 
                    ? 'text-emerald-400 bg-emerald-500/10 border border-emerald-500/20 px-2 py-0.5 rounded-md font-bold' 
                    : 'text-red-400 bg-red-500/10 border border-red-500/20 px-2 py-0.5 rounded-md font-semibold';

                // ฟังก์ชันแยกสีของ Badge คอลัมน์ Floor จริง
                let floorBadgeClass = "bg-gray-800 text-gray-300 border border-gray-700";
                if (log.floor.includes('1')) floorBadgeClass = "bg-red-500/10 text-red-400 border border-red-500/30";
                else if (log.floor.includes('2')) floorBadgeClass = "bg-orange-500/10 text-orange-400 border border-orange-500/30";
                else if (log.floor.includes('3')) floorBadgeClass = "bg-blue-500/10 text-blue-400 border border-blue-500/30";
                else if (log.floor.includes('4')) floorBadgeClass = "bg-purple-500/10 text-purple-400 border border-purple-500/30";

                tr.innerHTML = `
                    <td class="p-4 font-bold"><span class="px-2.5 py-1 rounded-md text-xs tracking-wide block text-center ${floorBadgeClass}">${log.floor}</span></td>
                    <td class="p-4 text-gray-400">${log.id}</td>
                    <td class="p-4 bg-blue-950/5"><span class="${badgeStyle(log.mTop3)}">${log.mTop3}</span></td>
                    <td class="p-4 bg-orange-950/5"><span class="${badgeStyle(log.mAvg)}">${log.mAvg}</span></td>
                    <td class="p-4 bg-emerald-950/5"><span class="${badgeStyle(log.rTop3)}">${log.rTop3}</span></td>
                    <td class="p-4 bg-pink-950/5"><span class="${badgeStyle(log.rAvg)}">${log.rAvg}</span></td>
                `;
                tbody.appendChild(tr);
            });

            renderHighContrastCharts(counts, detailErrors);
            analysisSection.classList.remove('hidden');
            
            engineStatus.className = "inline-flex items-center gap-2 text-xs font-medium bg-emerald-500/10 text-emerald-400 px-3 py-1 rounded-full border border-emerald-500/20 mt-1";
            engineStatus.innerHTML = `<span class="w-2 h-2 rounded-full bg-emerald-400"></span> Analysis Complete`;
        }

        function renderHighContrastCharts(counts, detailErrors) {
            const ctx1 = document.getElementById('accuracyChart').getContext('2d');
            const ctx2 = document.getElementById('errorFloorChart').getContext('2d');

            if(accChart) accChart.destroy();
            if(errChart) errChart.destroy();

            // ชุดสี High Contrast ที่ต่างกันสุดขั้ว: น้ำเงิน / ส้ม / เขียว / ชมพู
            const themeColors = ['#2563eb', '#ea580c', '#10b981', '#db2777'];

            accChart = new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: ['Meas (Top 3)', 'Meas (Avg)', 'RSSI (Top 3)', 'RSSI (Avg)'],
                    datasets: [{
                        data: [
                            ((counts.mTop3 / counts.total) * 100),
                            ((counts.mAvg / counts.total) * 100),
                            ((counts.rTop3 / counts.total) * 100),
                            ((counts.rAvg / counts.total) * 100)
                        ],
                        backgroundColor: themeColors,
                        borderRadius: 5
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: {
                        y: { min: 0, max: 100, grid: { color: '#1f2937' }, ticks: { color: '#9ca3af' } },
                        x: { grid: { display: false }, ticks: { color: '#9ca3af' } }
                    }
                }
            });

            const floorLabels = ['Floor 1', 'Floor 2', 'Floor 3', 'Floor 4'];
            errChart = new Chart(ctx2, {
                type: 'bar',
                data: {
                    labels: floorLabels,
                    datasets: [
                        {
                            label: 'Meas (Top 3)',
                            data: floorLabels.map(f => detailErrors[f].mTop3),
                            backgroundColor: themeColors[0],
                            borderRadius: 4
                        },
                        {
                            label: 'Meas (Avg)',
                            data: floorLabels.map(f => detailErrors[f].mAvg),
                            backgroundColor: themeColors[1],
                            borderRadius: 4
                        },
                        {
                            label: 'RSSI (Top 3)',
                            data: floorLabels.map(f => detailErrors[f].rTop3),
                            backgroundColor: themeColors[2],
                            borderRadius: 4
                        },
                        {
                            label: 'RSSI (Avg)',
                            data: floorLabels.map(f => detailErrors[f].rAvg),
                            backgroundColor: themeColors[3],
                            borderRadius: 4
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { 
                        legend: { 
                            display: true,
                            labels: { color: '#9ca3af', font: { size: 11 } }
                        } 
                    },
                    scales: {
                        y: { 
                            beginAtZero: true, 
                            grid: { color: '#1f2937' }, 
                            ticks: { color: '#9ca3af', stepSize: 1 } 
                        },
                        x: { grid: { display: false }, ticks: { color: '#9ca3af' } }
                    }
                }
            });
        }

        // ตัวสคริปต์แก้ไขจุดบกพร่อง ดึงข้อมูล Global สั่งแปลงสเปรดชีต Excel ทันที
        btnExport.addEventListener('click', () => {
            if (!globalLogs || globalLogs.length === 0) {
                alert("ไม่พบข้อมูลที่จะส่งออก กรุณาลากไฟล์ข้อมูลดิบลงในระบบก่อนครับ");
                return;
            }
            
            const excelRows = globalLogs.map(item => ({
                'Floor จริง': item.floor,
                'Particle ID': item.id,
                'ผลทำนาย Meas (3 ค่าน้อยสุด)': item.mTop3,
                'ผลทำนาย Meas (Avg รวมทั้งชั้น)': item.mAvg,
                'ผลทำนาย RSSI Predict (3 ค่าน้อยสุด)': item.rTop3,
                'ผลทำนาย RSSI Predict (Avg รวมทั้งชั้น)': item.rAvg
            }));

            const worksheet = XLSX.utils.json_to_sheet(excelRows);
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "KNN_Prediction_Data");

            const maxProps = [18, 15, 28, 28, 28, 28];
            worksheet['!cols'] = maxProps.map(w => ({ w: w }));

            XLSX.writeFile(workbook, "KNN_Predict_Dashboard_Report.xlsx");
        });
    </script>
</body>
</html>
