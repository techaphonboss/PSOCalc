<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PSOCalc - KNN Floor Prediction Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-50 text-gray-800 font-sans min-h-screen">

    <div class="container mx-auto px-4 py-8">
        <header class="mb-8 text-center bg-white p-6 rounded-2xl shadow-sm border border-gray-100">
            <h1 class="text-3xl font-extrabold text-blue-600 mb-2">📊 PSOCalc AI Dashboard</h1>
            <p class="text-gray-500 text-sm md:text-base">ระบบคำนวณสมการ KNN วิเคราะห์และเปรียบเทียบชั้นอัตโนมัติ (Live Math Engine)</p>
        </header>

        <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 mb-8">
            <h2 class="text-lg font-bold text-gray-700 mb-4 flex items-center gap-2">
                📂 อัปโหลดไฟล์ข้อมูลดิบของคุณ (.xlsx หรือ .csv)
            </h2>
            <div class="flex flex-col items-center justify-center border-2 border-dashed border-gray-300 rounded-xl p-8 bg-gray-50 hover:bg-gray-100 transition cursor-pointer" id="drop-zone">
                <input type="file" id="file-input" accept=".xlsx, .xls, .csv" class="hidden">
                <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                <p class="text-sm text-gray-600 font-medium">คลิกเพื่อเลือกไฟล์ หรือลากไฟล์มาวางที่นี่</p>
                <p class="text-xs text-gray-400 mt-1">โยนไฟล์ Excel ที่มีสัญญาณดิบเข้ามาได้เลย ระบบจะคำนวณหาคำตอบให้เองทั้งหมด</p>
            </div>
            <div id="file-info" class="mt-3 text-sm text-blue-600 font-medium hidden text-center"></div>
        </div>

        <div id="analysis-section" class="hidden space-y-8">
            
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-blue-500 uppercase">Meas. (Top 3 Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-m-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">อัตราการทำนายชั้นถูกต้อง</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-indigo-500 uppercase">Meas. (Grand Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-m-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">อัตราการทำนายชั้นถูกต้อง</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-emerald-500 uppercase">RSSI Pred (Top 3 Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-r-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">อัตราการทำนายชั้นถูกต้อง</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-teal-500 uppercase">RSSI Pred (Grand Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-r-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">อัตราการทำนายชั้นถูกต้อง</p>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm">
                    <h3 class="text-base font-bold text-gray-700 mb-4">🏆 สรุปเปรียบเทียบความแม่นยำของอัลกอริทึม (%)</h3>
                    <div class="h-64 relative">
                        <canvas id="accuracyChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm">
                    <h3 class="text-base font-bold text-gray-700 mb-4">📍 จำนวนจุดที่ระบบประเมินผลผิดพลาดแยกตามชั้นจริง</h3>
                    <div class="h-64 relative">
                        <canvas id="errorFloorChart"></canvas>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-2xl border border-gray-100 shadow-sm overflow-hidden">
                <div class="p-6 border-b border-gray-100 bg-gray-50/50 flex justify-between items-center">
                    <h3 class="text-base font-bold text-gray-700">📋 รายงานตารางคำนวณสดรายจุด (Live Calculation Logs)</h3>
                    <span class="text-xs bg-blue-100 text-blue-800 px-3 py-1 rounded-full font-medium" id="total-records">0 Particles</span>
                </div>
                <div class="overflow-x-auto max-h-96">
                    <table class="w-full text-left border-collapse text-xs md:text-sm">
                        <thead>
                            <tr class="bg-gray-100 text-gray-600 sticky top-0 uppercase font-semibold">
                                <th class="p-3 border-b">Floor จริง</th>
                                <th class="p-3 border-b">Particle ID</th>
                                <th class="p-3 bg-blue-50 text-blue-700 border-b">Meas (3 ค่าน้อยสุด)</th>
                                <th class="p-3 bg-indigo-50 text-indigo-700 border-b">Meas (Avg รวมชั้น)</th>
                                <th class="p-3 bg-emerald-50 text-emerald-700 border-b">RSSI Pred (3 ค่าน้อยสุด)</th>
                                <th class="p-3 bg-teal-50 text-teal-700 border-b">RSSI Pred (Avg รวมชั้น)</th>
                            </tr>
                        </thead>
                        <tbody id="table-output" class="divide-y divide-gray-100">
                            </tbody>
                    </table>
                </div>
            </div>

        </div>
    </div>

    <script>
        const dropZone = document.getElementById('drop-zone');
        const fileInput = document.getElementById('file-input');
        const fileInfo = document.getElementById('file-info');
        const analysisSection = document.getElementById('analysis-section');

        dropZone.addEventListener('click', () => fileInput.click());
        dropZone.addEventListener('dragover', (e) => { e.preventDefault(); dropZone.classList.add('bg-blue-50', 'border-blue-400'); });
        dropZone.addEventListener('dragleave', () => dropZone.classList.remove('bg-blue-50', 'border-blue-400'));
        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.classList.remove('bg-blue-50', 'border-blue-400');
            if (e.dataTransfer.files.length) handleFile(e.dataTransfer.files[0]);
        });
        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length) handleFile(e.target.files[0]);
        });

        function handleFile(file) {
            fileInfo.textContent = `ระบบ AI กำลังวิเคราะห์และคำนวณสมการคณิตศาสตร์รายจุด...`;
            fileInfo.classList.remove('hidden');
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const firstSheetName = workbook.SheetNames[0];
                const worksheet = workbook.Sheets[firstSheetName];
                
                // แปลงไฟล์เป็น Array ดิบแบบ Row-by-Row เพื่อความยืดหยุ่นสูงสุดในการสแกนข้อมูล
                const rawRows = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                processLiveKNN(rawRows);
            };
            reader.readAsArrayBuffer(file);
        }

        let accChart = null;
        let errChart = null;

        function processLiveKNN(rawRows) {
            // ระบบสแกนอัจฉริยะ: กรองเฉพาะแถวที่มีคอลัมน์แรกระบุชั้น "floor" โดยไม่สนใจแถวหัวตาราง
            let cleanData = [];
            rawRows.forEach(row => {
                if (row && row[0] && String(row[0]).trim().toLowerCase().startsWith('floor')) {
                    // ล้างค่าช่องว่าง (Trim) ข้อมูลพิกัดชั้นและไอดีประจำจุด
                    let floorStr = String(row[0]).trim();
                    let partStr = row[1] ? String(row[1]).trim() : '';
                    
                    // ดึงค่าความแรงสัญญาณ AN1, AN2, AN3 แปลงเป็นตัวเลขทศนิยม
                    let m1 = parseFloat(row[2]) || 0;
                    let m2 = parseFloat(row[3]) || 0;
                    let m3 = parseFloat(row[4]) || 0;
                    let r1 = parseFloat(row[5]) || 0;
                    let r2 = parseFloat(row[6]) || 0;
                    let r3 = parseFloat(row[7]) || 0;

                    cleanData.push({
                        actualFloor: floorStr,
                        particleId: partStr,
                        meas: [m1, m2, m3],
                        rssi: [r1, r2, r3]
                    });
                }
            });

            if (cleanData.length === 0) {
                fileInfo.textContent = "❌ ไม่พบข้อมูลโครงสร้างชั้นภายในไฟล์ กรุณาตรวจสอบคอลัมน์แรก";
                return;
            }

            let logs = [];
            let counts = { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0, total: cleanData.length };
            let floorErrors = { 'Floor 1': 0, 'Floor 2': 0, 'Floor 3': 0, 'Floor 4': 0 };

            document.getElementById('total-records').textContent = `${cleanData.length} Particles`;

            // ลูปคำนวณสมการรายจุดแบบจำลอง Mobile Unit
            cleanData.forEach((mobile) => {
                let actualFloor = mobile.actualFloor;
                
                let mDistances = { 'Floor 1': [], 'Floor 2': [], 'Floor 3': [], 'Floor 4': [] };
                let rDistances = { 'Floor 1': [], 'Floor 2': [], 'Floor 3': [], 'Floor 4': [] };

                // คำนวณ Euclidean Distance เทียบกับพิกัดอ้างอิงทั้งหมดในตาราง
                cleanData.forEach((target) => {
                    let targetFloor = target.actualFloor;

                    // สูตรขั้นตอนที่ 1: คำนวณระยะห่างทางเรขาคณิต (Euclidean Distance)
                    let distM = Math.sqrt(
                        Math.pow(mobile.meas[0] - target.meas[0], 2) +
                        Math.pow(mobile.meas[1] - target.meas[1], 2) +
                        Math.pow(mobile.meas[2] - target.meas[2], 2)
                    );

                    let distR = Math.sqrt(
                        Math.pow(mobile.rssi[0] - target.rssi[0], 2) +
                        Math.pow(mobile.rssi[1] - target.rssi[1], 2) +
                        Math.pow(mobile.rssi[2] - target.rssi[2], 2)
                    );

                    mDistances[targetFloor].push(distM);
                    rDistances[targetFloor].push(distR);
                });

                // ฟังก์ชันประมวลผลคำตอบ (ทำนายชั้น)
                function runPrediction(distanceMap, isTop3Mode) {
                    let finalFloorScores = {};
                    
                    for (let fl in distanceMap) {
                        let dList = distanceMap[fl];
                        if (isTop3Mode) {
                            // สูตรขั้นตอนที่ 2: กรองเอาค่าศูนย์ออก (ตัดตัวมันเอง) แล้วเรียงค่าน้อยสุด 3 ค่ามาหาค่าเฉลี่ย
                            let filtered = dList.filter(v => v > 0).sort((a, b) => a - b);
                            finalFloorScores[fl] = filtered.length >= 3 ? (filtered[0] + filtered[1] + filtered[2]) / 3 : (filtered[0] || 9999);
                        } else {
                            // สูตรขั้นตอนที่ 3: หาค่าเฉลี่ยรวมระยะทางของทั้งชั้น
                            let totalSum = dList.reduce((a, b) => a + b, 0);
                            finalFloorScores[fl] = dList.length > 0 ? (totalSum / dList.length) : 9999;
                        }
                    }

                    // คัดเลือกชั้นที่ได้คะแนนเฉลี่ยระยะทางต่ำสุด (ใกล้เคียงที่สุด)
                    let predicted = Object.keys(finalFloorScores).reduce((a, b) => finalFloorScores[a] < finalFloorScores[b] ? a : b);
                    
                    // ปรับแต่งการแสดงผล: ถ้าตรงกับชั้นจริงให้ตอบ "ถูก" ถ้าไม่ตรงให้แสดงเป็น "ชั้น X" ตามที่คุณต้องการ
                    return predicted === actualFloor ? "ถูก" : `ชั้น ${predicted.replace('Floor ', '')}`;
                }

                let p_m_top3 = runPrediction(mDistances, true);
                let p_m_avg  = runPrediction(mDistances, false);
                let p_r_top3 = runPrediction(rDistances, true);
                let p_r_avg  = runPrediction(rDistances, false);

                if (p_m_top3 === "ถูก") counts.mTop3++;
                if (p_m_avg === "ถูก") counts.mAvg++;
                if (p_r_top3 === "ถูก") counts.rTop3++;
                if (p_r_avg === "ถูก") counts.rAvg++;

                // บันทึกสถิติจุดทำนายพลาดของโมเดลหลัก (Measurement Top 3) ไปแจกแจงบนกราฟรายงานผล
                if (p_m_top3 !== "ถูก") {
                    if (actualFloor.includes('1')) floorErrors['Floor 1']++;
                    else if (actualFloor.includes('2')) floorErrors['Floor 2']++;
                    else if (actualFloor.includes('3')) floorErrors['Floor 3']++;
                    else if (actualFloor.includes('4')) floorErrors['Floor 4']++;
                }

                logs.push({
                    floor: actualFloor,
                    id: mobile.particleId,
                    mTop3: p_m_top3,
                    mAvg: p_m_avg,
                    rTop3: p_r_top3,
                    rAvg: p_r_avg
                });
            });

            // คำนวณสรุปเปอร์เซ็นต์ความถูกต้องของแต่ละอัลกอริทึม
            const calcPct = (v, t) => ((v / t) * 100).toFixed(2) + "%";
            document.getElementById('acc-m-top3').textContent = calcPct(counts.mTop3, counts.total);
            document.getElementById('acc-m-avg').textContent = calcPct(counts.mAvg, counts.total);
            document.getElementById('acc-r-top3').textContent = calcPct(counts.rTop3, counts.total);
            document.getElementById('acc-r-avg').textContent = calcPct(counts.rAvg, counts.total);

            // ฉีดแถวข้อมูลผลลัพธ์ ถูก/ผิด ลงตารางรายงานบนหน้าแดชบอร์ดเว็บ
            const tbody = document.getElementById('table-output');
            tbody.innerHTML = "";
            logs.forEach(log => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-gray-700 border-b">${log.floor}</td>
                    <td class="p-3 text-gray-500 border-b">${log.id}</td>
                    <td class="p-3 border-b ${log.mTop3 === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50 font-semibold'}">${log.mTop3}</td>
                    <td class="p-3 border-b ${log.mAvg === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50 font-semibold'}">${log.mAvg}</td>
                    <td class="p-3 border-b ${log.rTop3 === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50 font-semibold'}">${log.rTop3}</td>
                    <td class="p-3 border-b ${log.rAvg === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50 font-semibold'}">${log.rAvg}</td>
                `;
                tbody.appendChild(tr);
            });

            // สั่งอัปเดตแผนภูมิแท่งรายงานผลสถิติแบบ Dynamic
            renderCharts(counts, floorErrors);
            analysisSection.classList.remove('hidden');
            fileInfo.textContent = "✅ ประมวลผลสมการ KNN และรายงานสถิติสำเร็จเสร็จสิ้น!";
        }

        function renderCharts(counts, errors) {
            const ctx1 = document.getElementById('accuracyChart').getContext('2d');
            const ctx2 = document.getElementById('errorFloorChart').getContext('2d');

            if(accChart) accChart.destroy();
            if(errChart) errChart.destroy();

            // 1. กราฟแท่งแสดงอัตราความแม่นยำรวม (%) ของแต่ละวิธีคำนวณ
            accChart = new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: ['Meas (Top 3)', 'Meas (Avg รวมชั้น)', 'RSSI (Top 3)', 'RSSI (Avg รวมชั้น)'],
                    datasets: [{
                        label: 'ความถูกต้องแม่นยำ (%)',
                        data: [
                            ((counts.mTop3 / counts.total) * 100),
                            ((counts.mAvg / counts.total) * 100),
                            ((counts.rTop3 / counts.total) * 100),
                            ((counts.rAvg / counts.total) * 100)
                        ],
                        backgroundColor: ['#3b82f6', '#6366f1', '#10b981', '#14b8a6']
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: { y: { min: 0, max: 100 } }
                }
            });

            // 2. กราฟแท่งจำแนกปริมาณจุดคลาดเคลื่อนรายชั้น
            errChart = new Chart(ctx2, {
                type: 'bar',
                data: {
                    labels: Object.keys(errors),
                    datasets: [{
                        label: 'จำนวนจุดที่ทำนายสลับชั้น (จุด)',
                        data: Object.values(errors),
                        backgroundColor: ['#ef4444', '#f97316', '#f59e0b', '#84cc16']
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
                }
            });
        }
    </script>
</body>
</html>
