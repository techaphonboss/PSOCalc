<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PSOCalc - KNN Floor Prediction Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-50 text-gray-800 font-sans min-h-screen">

    <div class="container mx-auto px-4 py-8">
        <header class="mb-8 text-center bg-white p-6 rounded-2xl shadow-xs border border-gray-100">
            <h1 class="text-3xl font-extrabold text-blue-600 mb-2">📊 PSOCalc Dashboard</h1>
            <p class="text-gray-500 text-sm md:text-base">ระบบเปรียบเทียบอัลกอริทึมคำนวณชั้นภายในอาคาร (Measurement vs RSSI Predict)</p>
        </header>

        <div class="bg-white p-6 rounded-2xl shadow-xs border border-gray-100 mb-8">
            <h2 class="text-lg font-bold text-gray-700 mb-4 flex items-center gap-2">
                📂 อัปโหลดไฟล์ข้อมูลของคุณ (.xlsx หรือ .csv)
            </h2>
            <div class="flex flex-col items-center justify-center border-2 border-dashed border-gray-300 rounded-xl p-8 bg-gray-50 hover:bg-gray-100 transition cursor-pointer" id="drop-zone">
                <input type="file" id="file-input" accept=".xlsx, .xls, .csv" class="hidden">
                <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                <p class="text-sm text-gray-600 font-medium">คลิกเพื่อเลือกไฟล์ หรือลากไฟล์มาวางที่นี่</p>
                <p class="text-xs text-gray-400 mt-1">รองรับโครงสร้างตารางข้อมูล Particle ทั้ง 55 ตัวแยกตามชั้น 1-4</p>
            </div>
            <div id="file-info" class="mt-3 text-sm text-green-600 font-medium hidden text-center"></div>
        </div>

        <div id="analysis-section" class="hidden space-y-8">
            
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-2xs">
                    <span class="text-xs font-bold text-blue-500 uppercase">Meas. (Top 3 Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-m-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-2xs">
                    <span class="text-xs font-bold text-indigo-500 uppercase">Meas. (Grand Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-m-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-2xs">
                    <span class="text-xs font-bold text-emerald-500 uppercase">RSSI Pred (Top 3 Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-r-top3">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
                <div class="bg-white p-5 rounded-xl border border-gray-100 shadow-2xs">
                    <span class="text-xs font-bold text-teal-500 uppercase">RSSI Pred (Grand Avg)</span>
                    <h3 class="text-2xl font-black mt-1 text-gray-800" id="acc-r-avg">0%</h3>
                    <p class="text-xs text-gray-400 mt-1">ทำนายชั้นถูกต้องแม่นยำ</p>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-xs">
                    <h3 class="text-base font-bold text-gray-700 mb-4">🏆 สรุปเปรียบเทียบความแม่นยำรวม (%)</h3>
                    <div class="h-64 relative">
                        <canvas id="accuracyChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-xs">
                    <h3 class="text-base font-bold text-gray-700 mb-4">📍 อัตราการทายชั้นผิดพลาดจำแนกตามชั้นจริง</h3>
                    <div class="h-64 relative">
                        <canvas id="errorFloorChart"></canvas>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-2xl border border-gray-100 shadow-xs overflow-hidden">
                <div class="p-6 border-b border-gray-100 bg-gray-50/50">
                    <h3 class="text-base font-bold text-gray-700">📋 ตารางวิเคราะห์ผลการจำลองรายจุด (All Particles Logs)</h3>
                </div>
                <div class="overflow-x-auto max-h-96">
                    <table class="w-full text-left border-collapse text-xs md:text-sm">
                        <thead>
                            <tr class="bg-gray-100 text-gray-600 sticky top-0 uppercase font-semibold">
                                <th class="p-3">Floor (จริง)</th>
                                <th class="p-3">Particle ID</th>
                                <th class="p-3 bg-blue-50 text-blue-700">Meas (Top 3)</th>
                                <th class="p-3 bg-indigo-50 text-indigo-700">Meas (Avg รวม)</th>
                                <th class="p-3 bg-emerald-50 text-emerald-700">RSSI Pred (Top 3)</th>
                                <th class="p-3 bg-teal-50 text-teal-700">RSSI Pred (Avg รวม)</th>
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

        // Layout interactive triggers
        dropZone.addEventListener('click', () => fileInput.click());
        dropZone.addEventListener('dragover', (e) => { e.preventDefault(); dropZone.classList.add('bg-blue-50'); });
        dropZone.addEventListener('dragleave', () => dropZone.classList.remove('bg-blue-50'));
        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.classList.remove('bg-blue-50');
            if (e.dataTransfer.files.length) handleFile(e.dataTransfer.files[0]);
        });
        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length) handleFile(e.target.files[0]);
        });

        function handleFile(file) {
            fileInfo.textContent = `กำลังโหลดและประมวลผลไฟล์: ${file.name}`;
            fileInfo.classList.remove('hidden');
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array'});
                const firstSheetName = workbook.SheetNames[0];
                const worksheet = workbook.Sheets[firstSheetName];
                const jsonData = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                
                processData(jsonData);
            };
            reader.readAsArrayBuffer(file);
        }

        // Global Charts Reference
        let accChart = null;
        let errChart = null;

        function processData(rows) {
            // ค้นหาดัชนีหัวตารางเพื่อความแม่นยำ
            let headers = rows[0];
            let dataRows = rows.slice(1).filter(r => r[0] && String(r[0]).includes('Floor'));

            let logs = [];
            let counts = {
                mTop3: 0, mAvg: 0, rTop3: 0, rAvg: 0, total: dataRows.length
            };

            let floorMissMatches = { 'Floor 1': 0, 'Floor 2': 0, 'Floor 3': 0, 'Floor 4': 0 };

            // ประมวลผลลูปจำลองแต่ละ Particle ตามเงื่อนไขทางคณิตศาสตร์ของคุณ
            dataRows.forEach(row => {
                let currentFloor = String(row[0]).trim(); // ชั้นแท้จริง เช่น "Floor 4"
                let particleId = row[1];

                // จำลองขั้นตอนที่ 1: ดึงค่าระยะห่าง Euclidean Distance จากตาราง หรือประมวลผล Logic จำลอง
                // จากแบบฟอร์มจำลอง จะทำการดึงผลสรุปการประเมินจากชุดคำสั่งคอลัมน์ Excel
                let predMeasTop3 = row[10] || "ไม่ระบุ"; 
                let predMeasAvg = row[11] || "ไม่ระบุ";
                let predRSSITop3 = row[12] || "ไม่ระบุ";
                let predRSSIAvg = row[13] || "ไม่ระบุ";

                // จัดการแปลงการแสดงผลเพื่อตรวจสอบความถูกต้อง
                let isMeasTop3True = predMeasTop3 === "ถูก";
                let isMeasAvgTrue = predMeasAvg === "ถูก";
                let isRSSITop3True = predRSSITop3 === "ถูก";
                let isRSSIAvgTrue = predRSSIAvg === "ถูก";

                if(isMeasTop3True) counts.mTop3++;
                if(isMeasAvgTrue) counts.mAvg++;
                if(isRSSITop3True) counts.rTop3++;
                if(isRSSIAvgTrue) counts.rAvg++;

                // บันทึกสถิติจุดที่ผิดชั้นสำหรับการทำกราฟวิเคราะห์ปัญหา (Error profiling)
                if(!isMeasTop3True) {
                    floorMissMatches[currentFloor] = (floorMissMatches[currentFloor] || 0) + 1;
                }

                logs.push({
                    floor: currentFloor,
                    id: particleId,
                    mTop3: predMeasTop3,
                    mAvg: predMeasAvg,
                    rTop3: predRSSITop3,
                    rAvg: predRSSIAvg,
                    flags: [isMeasTop3True, isMeasAvgTrue, isRSSITop3True, isRSSIAvgTrue]
                });
            });

            // อัปเดต UI Metrics Cards
            const calcPct = (val, total) => ((val / total) * 100).toFixed(2) + "%";
            document.getElementById('acc-m-top3').textContent = calcPct(counts.mTop3, counts.total);
            document.getElementById('acc-m-avg').textContent = calcPct(counts.mAvg, counts.total);
            document.getElementById('acc-r-top3').textContent = calcPct(counts.rTop3, counts.total);
            document.getElementById('acc-r-avg').textContent = calcPct(counts.rAvg, counts.total);

            // เจนเนอเรทตารางแสดงผลลัพธ์
            const tbody = document.getElementById('table-output');
            tbody.innerHTML = "";
            logs.forEach(log => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td class="p-3 font-semibold text-gray-700">${log.floor}</td>
                    <td class="p-3 text-gray-500">${log.id}</td>
                    <td class="p-3 ${log.flags[0] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.mTop3}</td>
                    <td class="p-3 ${log.flags[1] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.mAvg}</td>
                    <td class="p-3 ${log.flags[2] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.rTop3}</td>
                    <td class="p-3 ${log.flags[3] ? 'text-green-600 bg-green-50 font-bold' : 'text-red-500 bg-red-50'}">${log.rAvg}</td>
                `;
                tbody.appendChild(tr);
            });

            // วาดกราฟเปรียบเทียบ
            renderCharts(counts, floorMissMatches);
            analysisSection.classList.remove('hidden');
        }

        function renderCharts(counts, errors) {
            const ctx1 = document.getElementById('accuracyChart').getContext('2d');
            const ctx2 = document.getElementById('errorFloorChart').getContext('2d');

            if(accChart) accChart.destroy();
            if(errChart) errChart.destroy();

            // กราฟที่ 1: อัตราความแม่นยำรวม
            accChart = new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: ['Meas (Top3)', 'Meas (Avg รวม)', 'RSSI (Top3)', 'RSSI (Avg รวม)'],
                    datasets: [{
                        label: 'เปอร์เซ็นต์ความถูกต้อง (%)',
                        data: [
                            ((counts.mTop3/counts.total)*100),
                            ((counts.mAvg/counts.total)*100),
                            ((counts.rTop3/counts.total)*100),
                            ((counts.rAvg/counts.total)*100)
                        ],
                        backgroundColor: ['#3b82f6', '#6366f1', '#10b981', '#14b8a6']
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { min: 0, max: 100 } } }
            });

            // กราฟที่ 2: จุดผิดพลาดจำแนกตามชั้น
            errChart = new Chart(ctx2, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(errors),
                    datasets: [{
                        data: Object.values(errors),
                        backgroundColor: ['#f87171', '#fb923c', '#fbbf24', '#a3e635']
                    }]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });
        }
    </script>
</body>
</html>
