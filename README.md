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
                
                // โหลดเป็นตารางแถวตรง ๆ
                const rawRows = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                processLiveKNN(rawRows);
            };
            reader.readAsArrayBuffer(file);
        }

        let accChart = null;
        let errChart = null;

        function processLiveKNN(rawRows) {
            let cleanData = [];
            
            // วนลูปอ่านข้อมูลข้าม 2 แถวแรกที่เป็นหัวตาราง และเลือกเฉพาะแถวที่มีข้อมูลจริง
            for (let i = 0; i < rawRows.length; i++) {
                let row = rawRows[i];
                if (row && row[0] && String(row[0]).trim().toLowerCase().startsWith('floor')) {
                    // ป้องกันแถวที่เป็นข้อความ "Floor" ที่เป็นหัวข้อคอลัมน์ A แถวที่ 1 
                    if (String(row[1]).trim().toLowerCase().startsWith('particle')) continue;

                    let floorStr = String(row[0]).trim();
                    let partStr = row[1] ? String(row[1]).trim() : '';
                    
                    // แปลงค่าความแรงสัญญาณเป็นตัวเลขให้ปลอดภัย
                    let m1 = parseFloat(row[2]);
                    let m2 = parseFloat(row[3]);
                    let m3 = parseFloat(row[4]);
                    
                    let r1 = parseFloat(row[5]);
                    let r2 = parseFloat(row[6]);
                    let r3 = parseFloat(row[7]);

                    // ดักจับ: ถ้าแถวไหนมีค่าความแรงสัญญาณหลุดเป็นตัวอักษร ให้ข้ามทันทีป้องกันระบบค้าง
                    if (isNaN(m1) || isNaN(m2) || isNaN(m3)) continue;

                    cleanData.push({
                        actualFloor: floorStr,
                        particleId: partStr,
                        meas: [m1, m2, m3],
                        rssi:
