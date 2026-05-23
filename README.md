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
            <p class="text-gray-500 text-sm md:text-base">ระบบประมวลผลสมการ KNN คำนวณชั้นภายในอาคารอัตโนมัติ (Measurement vs RSSI Predict)</p>
        </header>

        <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 mb-8">
            <h2 class="text-lg font-bold text-gray-700 mb-4 flex items-center gap-2">
                📂 อัปโหลดไฟล์ข้อมูลดิบของคุณ (.xlsx หรือ .csv)
            </h2>
            <div class="flex flex-col items-center justify-center border-2 border-dashed border-gray-300 rounded-xl p-8 bg-gray-50 hover:bg-gray-100 transition cursor-pointer" id="drop-zone">
                <input type="file" id="file-input" accept=".xlsx, .xls, .csv" class="hidden">
                <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                <p class="text-sm text-gray-600 font-medium">คลิกเพื่อเลือกไฟล์ หรือลากไฟล์มาวางที่นี่</p>
                <p class="text-xs text-gray-400 mt-1">ระบบจะดึงเฉพาะสัญญาณคอลัมน์ C ถึง H ไปคำนวณสมการหาค่า ถูก/ผิด ให้เองทันที</p>
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
                    <h3 class="text-base font-bold text-gray-700">📋 ตารางคำนวณผลลัพธ์รายจุดแบบเรียลไทม์โดยระบบ AI Web-Engine</h3>
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
            fileInfo.textContent = `ระบบกำลังประมวลผลสมการคณิตศาสตร์และจำลองตำแหน่ง...`;
            fileInfo.classList.remove('hidden');
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const firstSheetName = workbook.SheetNames[0];
                const worksheet = workbook.Sheets[firstSheetName];
                const jsonData = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                
                executeLiveKNN(jsonData);
            };
            reader.readAsArrayBuffer(file);
        }

        let accChart = null;
        let errChart = null;

        function executeLiveKNN(rows) {
            // กรองเอาแถวข้อมูลจริงคัดแยกส่วนหัวตารางออก
            let dataRows = rows.filter(r => r[0] && String(r[0]).trim().toLowerCase().startsWith('floor'));

            let logs = [];
            let counts = { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0, total: dataRows.length };
            let floorErrors = { 'Floor 1': 0, 'Floor 2': 0, 'Floor 3': 0, 'Floor 4': 0 };

            document.getElementById('total-records').textContent = `${dataRows.length} Particles`;

            // วนลูปคำนวณสมการรายพิกัด โดยให้ตัวแปรแต่ละแถวเป็น Mobile Unit เพื่อทำนายชั้นแบบสดๆ
            dataRows.forEach((mobileRow) => {
                let actualFloor = String(mobileRow[0]).trim(); // ชั้นที่ถูกต้องจริง
                let particleId = String(mobileRow[1]).trim();  // รหัสไอดีประจำจุด
                
                // ดึงค่าความแรงสัญญาณดิบจากฟอร์มตาราง Excel
                let m_an1 = parseFloat(mobileRow[2]) || 0;
                let m_an2 = parseFloat(mobileRow[3]) || 0;
                let m_an3 = parseFloat(mobileRow[4]) || 0;

                let r_an1 = parseFloat(mobileRow[5]) || 0;
                let r_an2 = parseFloat(mobileRow[6]) || 0;
                let r_an3 = parseFloat(mobileRow[7]) || 0;

                // อาร์เรย์เก็บค่าระยะห่างแยกแต่ละชั้น
                let mDistances = { 'Floor 1': [], 'Floor 2': [], 'Floor 3': [], 'Floor 4': [] };
                let rDistances = { 'Floor 1': [], 'Floor 2': [], 'Floor 3': [], 'Floor 4': [] };

                // คำนวณ Euclidean Distance เทียบกับกลุ่มข้อมูลประวัติทั้งหมด 55 ตัว
                dataRows.forEach((targetRow) => {
                    let targetFloor = String(targetRow[0]).trim();
                    
                    let t_m1 = parseFloat(targetRow[2]) || 0;
                    let t_m2 = parseFloat(targetRow[3]) || 0;
                    let t_m3 = parseFloat(targetRow[4]) || 0;

                    let t_r1 = parseFloat(targetRow[5]) || 0;
                    let t_r2 = parseFloat(targetRow[6]) || 0;
                    let t_r3 = parseFloat(targetRow[7]) || 0;

                    // สูตรคณิตศาสตร์ขั้นตอนที่ 1: SQRT((AN1_PR - AN1_Mobile)^2 + ...)
                    let d_m = Math.sqrt(Math.pow(m_an1 - t_m1, 2) + Math.pow(m_an2 - t_m2, 2) + Math.pow(m_an3 - t_m3, 2));
                    let d_r = Math.sqrt(Math.pow(r_an1 - t_r1, 2) + Math.pow(r_an2 - t_r2, 2) + Math.pow(r_an3 - t_r3, 2));

                    mDistances[targetFloor].push(d_m);
                    rDistances[targetFloor].push(d_r);
                });

                // ฟังก์ชันย่อยสำหรับประเมินผลหาชั้นที่ให้ระยะห่างน้อยที่สุด
                function predict(distanceObj, isTop3) {
                    let floorScores = {};
                    for (let fl in distanceObj) {
                        let arr = distanceObj[fl];
                        if (isTop3) {
                            // สูตรขั้นตอนที่ 2: กรองค่าที่เป็น 0 ออก (ตัวมันเอง) จากนั้นเลือกมา 3 ค่าน้อยสุดเพื่อหาเฉลี่ย
                            let filtered = arr.filter(d => d > 0).sort((a, b) => a - b);
                            floorScores[fl] = filtered.length >= 3 ? (filtered[0] + filtered[1] + filtered[2]) / 3 : (filtered[0] || 999);
                        } else {
                            // สูตรขั้นตอนที่ 3: หาค่าเฉลี่ยรวมทั้งหมดในชั้นเพื่อนำมาเปรียบเทียบตัวแปร
                            let sum = arr.reduce((a, b) => a + b, 0);
                            floorScores[fl] = arr.length > 0 ? (sum / arr.length) : 999;
                        }
                    }
                    // ดึงชั้นที่คำนวณแล้วได้ระยะสั้นที่สุด (ใกล้เคียงตำแหน่งจริงที่สุด)
                    let predictedFloor = Object.keys(floorScores).reduce((a, b) => floorScores[a] < floorScores[b] ? a : b);
                    return predictedFloor === actualFloor ? "ถูก" : `ชั้น ${predictedFloor.replace('Floor ', '')}`;
                }

                let p_m_top3 = predict(mDistances, true);
                let p_m_avg  = predict(mDistances, false);
                let p_r_top3 = predict(rDistances, true);
                let p_r_avg  = predict(rDistances, false);

                if (p_m_top3 === "ถูก") counts.mTop3++;
                if (p_m_avg === "ถูก") counts.mAvg++;
                if (p_r_top3 === "ถูก") counts.rTop3++;
                if (p_r_avg === "ถูก") counts.rAvg++;

                // บันทึกสถิติจุดคลาดเคลื่อนสำหรับนำไปจัดหมวดหมู่รายงานวิจัย
                if (p_m_top3 !== "ถูก") {
                    floorErrors[actualFloor] = (floorErrors[actualFloor] || 0) + 1;
                }

                logs.push({
                    floor: actualFloor, id: particleId,
                    mTop3: p_m_top3, mAvg: p_m_avg,
                    rTop3: p_r_top3, rAvg: p_r_avg
                });
            });

            // อัปเดตข้อมูลอัตราความแม่นยำรวม (%) ลงบนหน้าเว็บแดชบอร์ด
            const pct = (v, t) => ((v / t) * 100).toFixed(2) + "%";
            document.getElementById('acc-m-top3').textContent = pct(counts.mTop3, counts.total);
            document.getElementById('acc-m-avg').textContent = pct(counts.mAvg, counts.total);
            document.getElementById('acc-r-top3').textContent = pct(counts.rTop3, counts.total);
            document.getElementById('acc-r-avg').textContent = pct(counts.rAvg, counts.total);

            // พิมพ์ผลลัพธ์ลงในตารางรายจุดให้เห็นผลชัดเจนครบถ้วนทั้ง 55 แถว
            const tbody = document.getElementById('table-output');
            tbody.innerHTML = "";
            logs.forEach(log => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-gray-700 border-b">${log.floor}</td>
                    <td class="p-3 text-gray-500 border-b">${log.id}</td>
                    <td class="p-3 border-b ${log.mTop3 === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.mTop3}</td>
                    <td class="p-3 border-b ${log.mAvg === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.mAvg}</td>
                    <td class="p-3 border-b ${log.rTop3 === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.rTop3}</td>
                    <td class="p-3 border-b ${log.rAvg === 'ถูก' ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.rAvg}</td>
                `;
                tbody.appendChild(tr);
            });

            // วาดกราฟสรุปสถิติเปรียบเทียบประสิทธิภาพแบบไดนามิก
            renderCharts(counts, floorErrors);
            analysisSection.classList.remove('hidden');
            fileInfo.textContent = "✅ คำนวณสมการและวิเคราะห์ข้อมูล IPS เสร็จสมบูรณ์!";
        }

        function renderCharts(counts, errors) {
            const ctx1 = document.getElementById('accuracyChart').getContext('2d');
            const ctx2 = document.getElementById('errorFloorChart').getContext('2d');

            if(accChart) accChart.destroy();
            if(errChart) errChart.destroy();

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

            errChart = new Chart(ctx2, {
                type: 'bar',
                data: {
                    labels: Object.keys(errors),
                    datasets: [{
                        label: 'จำนวนจุดที่ทำนายผิดพลาด (จุด)',
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
