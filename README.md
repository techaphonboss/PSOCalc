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
            <h1 class="text-3xl font-extrabold text-blue-600 mb-2">📊 PSOCalc Dashboard</h1>
            <p class="text-gray-500 text-sm md:text-base">ระบบเปรียบเทียบอัลกอริทึมคำนวณชั้นภายในอาคาร (Measurement vs RSSI Predict)</p>
        </header>

        <div class="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 mb-8">
            <h2 class="text-lg font-bold text-gray-700 mb-4 flex items-center gap-2">
                📂 อัปโหลดไฟล์ข้อมูลของคุณ (.xlsx หรือ .csv)
            </h2>
            <div class="flex flex-col items-center justify-center border-2 border-dashed border-gray-300 rounded-xl p-8 bg-gray-50 hover:bg-gray-100 transition cursor-pointer" id="drop-zone">
                <input type="file" id="file-input" accept=".xlsx, .xls, .csv" class="hidden">
                <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                <p class="text-sm text-gray-600 font-medium">คลิกเพื่อเลือกไฟล์ หรือลากไฟล์มาวางที่นี่</p>
                <p class="text-xs text-gray-400 mt-1">รองรับไฟล์ตารางประวัติ Particle ที่มีคอลัมน์ผลทำนาย ถูก/ผิด</p>
            </div>
            <div id="file-info" class="mt-3 text-sm text-green-600 font-medium hidden text-center"></div>
        </div>

        <div id="analysis-section" class="hidden space-y-8">
            
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-blue-500 uppercase">Meas. (Top 3 Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-m-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-indigo-500 uppercase">Meas. (Grand Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-m-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-emerald-500 uppercase">RSSI Pred (Top 3 Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-r-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-sm">
                    <span class="text-xs font-bold text-teal-500 uppercase">RSSI Pred (Grand Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-r-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm">
                    <h3 class="text-base font-bold text-gray-700 mb-4">🏆 สรุปเปรียบเทียบความแม่นยำรวม (%)</h3>
                    <div class="h-64 relative">
                        <canvas id="accuracyChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm">
                    <h3 class="text-base font-bold text-gray-700 mb-4">📍 อัตราการทำนายผิดพลาดจำแนกตามชั้นจริง (อิงตาม Meas. Top 3)</h3>
                    <div class="h-64 relative">
                        <canvas id="errorFloorChart"></canvas>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-2xl border border-gray-100 shadow-sm overflow-hidden">
                <div class="p-6 border-b border-gray-100 bg-gray-50/50 flex justify-between items-center">
                    <h3 class="text-base font-bold text-gray-700">📋 ตารางดึงข้อมูลวิเคราะห์ผลการทำนายรายจุดจาก Excel</h3>
                    <span class="text-xs bg-blue-100 text-blue-800 px-3 py-1 rounded-full font-medium" id="total-records">0 จุด</span>
                </div>
                <div class="overflow-x-auto max-h-96">
                    <table class="w-full text-left border-collapse text-xs md:text-sm">
                        <thead>
                            <tr class="bg-gray-100 text-gray-600 sticky top-0 uppercase font-semibold">
                                <th class="p-3 border-b">Floor จริง</th>
                                <th class="p-3 border-b">Particle ID</th>
                                <th class="p-3 bg-blue-50 text-blue-700 border-b">Meas (3 ค่าหาร 3)</th>
                                <th class="p-3 bg-indigo-50 text-indigo-700 border-b">Meas (Avg. รวม)</th>
                                <th class="p-3 bg-emerald-50 text-emerald-700 border-b">RSSI Pred (3 ค่าหาร 3)</th>
                                <th class="p-3 bg-teal-50 text-teal-700 border-b">RSSI Pred (Avg. รวม)</th>
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
            fileInfo.textContent = `กำลังโหลดและดึงข้อมูลสรุป: ${file.name}`;
            fileInfo.classList.remove('hidden');
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const firstSheetName = workbook.SheetNames[0];
                const worksheet = workbook.Sheets[firstSheetName];
                const jsonData = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                
                parseExcelData(jsonData);
            };
            reader.readAsArrayBuffer(file);
        }

        let accChart = null;
        let errChart = null;

        function parseExcelData(rows) {
            // กรองเอาเฉพาะแถวข้อมูลที่มีคำว่า Floor อยู่ในคอลัมน์แรก (ข้ามหัวตารางแถวที่ 1-2)
            let dataRows = rows.filter(r => r[0] && String(r[0]).trim().startsWith('Floor'));

            let logs = [];
            let counts = { mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0, total: dataRows.length };
            let floorErrors = { 'Floor 1': 0, 'Floor 2': 0, 'Floor 3': 0, 'Floor 4': 0 };

            document.getElementById('total-records').textContent = `${dataRows.length} Particles`;

            dataRows.forEach(row => {
                let actualFloor = String(row[0]).trim(); // คอลัมน์ A: Floor จริง
                let particleId = row[1];                // คอลัมน์ B: Particle ID

                // แมปปิ้งดัชนีคอลัมน์ให้ตรงตามภาพจริงใน Excel (เริ่มนับจากคอลัมน์ A = 0)
                // J = 9 (Meas 3 ค่า), K = 10 (Meas Avg), L = 11 (RSSI 3 ค่า), M = 12 (RSSI Avg)
                let predMeasTop3 = row[10] ? String(row[10]).trim() : "ไม่ระบุ"; 
                let predMeasAvg  = row[11] ? String(row[11]).trim() : "ไม่ระบุ"; 
                let predRSSITop3 = row[12] ? String(row[12]).trim() : "ไม่ระบุ"; 
                let predRSSIAvg  = row[13] ? String(row[13]).trim() : "ไม่ระบุ"; 

                // ตรวจสอบสถานะความถูกต้องเพื่อนำไปคำนวณสถิติเป็น %
                let isMeasTop3True = predMeasTop3 === "ถูก";
                let isMeasAvgTrue  = predMeasAvg === "ถูก";
                let isRSSITop3True = predRSSITop3 === "ถูก";
                let isRSSIAvgTrue  = predRSSIAvg === "ถูก";

                if(isMeasTop3True) counts.mTop3++;
                if(isMeasAvgTrue)  counts.mAvg++;
                if(isRSSITop3True) counts.rTop3++;
                if(isRSSIAvgTrue)  counts.rAvg++;

                // บันทึกสถิติจุดที่ผิดชั้นเพื่อพลอตกราฟวงกลมวิเคราะห์ปัญหา
                if(!isMeasTop3True) {
                    floorErrors[actualFloor] = (floorErrors[actualFloor] || 0) + 1;
                }

                logs.push({
                    floor: actualFloor,
                    id: particleId,
                    mTop3: predMeasTop3,
                    mAvg: predMeasAvg,
                    rTop3: predRSSITop3,
                    rAvg: predRSSIAvg,
                    flags: [isMeasTop3True, isMeasAvgTrue, isRSSITop3True, isRSSIAvgTrue]
                });
            });

            // อัปเดตตัวเลขเปอร์เซ็นต์บนหน้าเว็บการ์ด
            const calcPct = (val, total) => total > 0 ? ((val / total) * 100).toFixed(2) + "%" : "0%";
            document.getElementById('acc-m-top3').textContent = calcPct(counts.mTop3, counts.total);
            document.getElementById('acc-m-avg').textContent = calcPct(counts.mAvg, counts.total);
            document.getElementById('acc-r-top3').textContent = calcPct(counts.rTop3, counts.total);
            document.getElementById('acc-r-avg').textContent = calcPct(counts.rAvg, counts.total);

            // เจนเนอเรทข้อมูลลงตารางจำลองบนหน้าเว็บ
            const tbody = document.getElementById('table-output');
            tbody.innerHTML = "";
            logs.forEach(log => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-gray-700 border-b">${log.floor}</td>
                    <td class="p-3 text-gray-500 border-b">${log.id}</td>
                    <td class="p-3 border-b ${log.flags[0] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.mTop3}</td>
                    <td class="p-3 border-b ${log.flags[1] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.mAvg}</td>
                    <td class="p-3 border-b ${log.flags[2] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.rTop3}</td>
                    <td class="p-3 border-b ${log.flags[3] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.rAvg}</td>
                `;
                tbody.appendChild(tr);
            });

            // วาดและแสดงผลกราฟแดชบอร์ด
            renderCharts(counts, floorErrors);
            analysisSection.classList.remove('hidden');
        }

        function renderCharts(counts, errors) {
            const ctx1 = document.getElementById('accuracyChart').getContext('2d');
            const ctx2 = document.getElementById('errorFloorChart').getContext('2d');

            if(accChart) accChart.destroy();
            if(errChart) errChart.destroy();

            // กราฟแท่งเปรียบเทียบ % ความถูกต้องรวมของทั้ง 4 วิธีคำนวณ
            accChart = new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: ['Meas (Top 3)', 'Meas (Avg รวม)', 'RSSI (Top 3)', 'RSSI (Avg รวม)'],
                    datasets: [{
                        label: 'เปอร์เซ็นต์ความถูกต้องแม่นยำ (%)',
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

            // กราฟวงกลมวิเคราะห์อัตราการทายชั้นผิดพลาดจำแนกตามชั้นจริง
            errChart = new Chart(ctx2, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(errors),
                    datasets: [{
                        data: Object.values(errors),
                        backgroundColor: ['#ef4444', '#f97316', '#f59e0b', '#84cc16']
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });
        }
    </script>
</body>
</html>
