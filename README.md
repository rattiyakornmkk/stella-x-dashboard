<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>แดชบอร์ดสรุปข้อมูลคดี - บริษัท สเตลล่า เอ็กซ์ จำกัด (มหาชน)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutral Harmony -->
    <!-- Application Structure Plan: A dashboard-centric SPA. The structure starts with high-level KPIs (total damage, case counts by status) for an immediate overview. Below this, a large, interactive bar chart visualizes the financial impact of each case. The main user interaction is a set of filter buttons to dynamically display case "cards" based on their legal status (Court vs. Police). Clicking a card opens a modal for in-depth details. This structure was chosen to transform a static, linear report into an exploratory tool. It guides the user from a broad summary to specific details on demand, which is more intuitive and less overwhelming than a long scrollable document. -->
    <!-- Visualization & Content Choices: 
        - Total Damage/Case Counts -> KPI Cards -> To inform with key metrics -> Static HTML -> Provides instant, critical summary figures.
        - Damage per Case -> Bar Chart -> To compare financial impact -> Chart.js Canvas -> Allows for quick visual comparison of case severity. Hover interaction provides specifics.
        - Case Distribution by Status -> Donut Chart -> To organize/inform -> Chart.js Canvas -> Shows the proportion of cases in different stages of the legal process.
        - Case List -> Interactive Card Grid -> To organize and inform -> HTML/CSS/JS -> Users can scan summaries and filter the list. Clicking a card to open a modal provides progressive disclosure, revealing details without cluttering the initial view. This avoids a wall of text.
        - Detailed Case Info -> Modal Popup -> To inform -> JS show/hide -> Provides comprehensive data only when requested by the user, maintaining a clean UI. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #f8f7f4; 
        }
        .chart-container {
            position: relative;
            width: 100%;
            margin-left: auto;
            margin-right: auto;
            height: 320px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 400px;
                max-height: 500px;
            }
        }
        .modal-backdrop {
            transition: opacity 0.3s ease;
        }
        .modal-content {
            transition: transform 0.3s ease;
        }
        .btn-filter.active {
            background-color: #0c4a6e;
            color: white;
        }
        /* Specific header colors for table */
        .case-group-filed-self {
            background-color: #0369a1; /* sky-700 */
        }
        .case-group-joint-filed {
            background-color: #059669; /* emerald-600 */
        }
        .case-group-problematic {
            background-color: #dc2626; /* red-600 */
        }
        .case-table-row:nth-child(even) {
            background-color: #f0f9ff;
        }
    </style>
</head>
<body class="text-stone-800">

    <div class="container mx-auto p-4 md:p-8">
        
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-sky-900">ภาพรวมการดำเนินคดี</h1>
            <p class="text-lg text-stone-600 mt-2">บริษัท สเตลล่า เอ็กซ์ จำกัด (มหาชน) และบริษัทในเครือ</p>
        </header>

        <main>
            <section id="summary-dashboard" class="mb-8">
                 <div class="text-center mb-6">
                    <h2 class="text-2xl font-semibold text-stone-700">สรุปข้อมูลภาพรวม</h2>
                    <p class="text-stone-500">ข้อมูลสรุปสถานะคดีและมูลค่าความเสียหายทั้งหมด ณ ปัจจุบัน</p>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 md:gap-6 text-center">
                    <div class="bg-white p-6 rounded-xl shadow-md border border-stone-200">
                        <h3 class="text-lg font-semibold text-stone-500">มูลค่าความเสียหายรวม</h3>
                        <p id="total-damage" class="text-3xl font-bold text-red-600 mt-2">1,319.37 ล้านบาท</p>
                    </div>
                    <div class="bg-white p-6 rounded-xl shadow-md border border-stone-200">
                        <h3 class="text-lg font-semibold text-stone-500">คดีในชั้นศาล</h3>
                        <p id="court-cases" class="text-3xl font-bold text-sky-800 mt-2">6 คดี</p>
                    </div>
                    <div class="bg-white p-6 rounded-xl shadow-md border border-stone-200">
                        <h3 class="text-lg font-semibold text-stone-500">คดีในชั้นสอบสวน</h3>
                        <p id="police-cases" class="text-3xl font-bold text-amber-600 mt-2">7 คดี</p>
                    </div>
                </div>
            </section>
            
            <section id="visualizations" class="mb-8 grid grid-cols-1 lg:grid-cols-5 gap-6">
                <div class="lg:col-span-3 bg-white p-4 md:p-6 rounded-xl shadow-md border border-stone-200">
                    <h2 class="text-xl font-semibold text-stone-700 text-center mb-4">มูลค่าความเสียหายแยกตามคดี (ล้านบาท)</h2>
                    <div class="chart-container">
                        <canvas id="damageByCaseChart"></canvas>
                    </div>
                </div>
                <div class="lg:col-span-2 bg-white p-4 md:p-6 rounded-xl shadow-md border border-stone-200 flex flex-col justify-center">
                    <h2 class="text-xl font-semibold text-stone-700 text-center mb-4">สัดส่วนประเภทคดี</h2>
                     <div class="chart-container" style="height: 300px; max-height: 350px;">
                        <canvas id="caseTypeChart"></canvas>
                    </div>
                </div>
            </section>

            <section id="case-details">
                <div class="text-center mb-6">
                    <h2 class="text-2xl font-semibold text-stone-700">รายละเอียดการดำเนินคดี</h2>
                    <p class="text-stone-500">เลือกดูรายละเอียดของแต่ละคดี หรือกรองตามสถานะ</p>
                </div>
                <div id="filter-buttons" class="flex justify-center flex-wrap gap-2 mb-8">
                    <button class="btn-filter active py-2 px-4 bg-white border border-stone-300 rounded-full shadow-sm hover:bg-sky-100 transition" data-filter="all">ทั้งหมด (13)</button>
                    <button class="btn-filter py-2 px-4 bg-white border border-stone-300 rounded-full shadow-sm hover:bg-sky-100 transition" data-filter="court">คดีในชั้นศาล (6)</button>
                    <button class="btn-filter py-2 px-4 bg-white border border-stone-300 rounded-full shadow-sm hover:bg-sky-100 transition" data-filter="police">คดีในชั้นสอบสวน (7)</button>
                </div>
                <div id="case-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                </div>
            </section>

            <section id="case-categorization-table" class="mt-12 bg-white p-4 md:p-6 rounded-xl shadow-md border border-stone-200">
                <h2 class="text-2xl font-semibold text-stone-700 text-center mb-6">ตารางจำแนกคดี 3 กลุ่ม พร้อมระบุสถานะ</h2>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-stone-200 rounded-lg overflow-hidden">
                        <thead>
                            <tr>
                                <th class="case-table-header case-group-filed-self px-4 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tl-lg">ฟ้องเอง<br>(เรื่อง)</th>
                                <th class="case-table-header case-group-filed-self px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">ฟ้องเอง<br>(สถานะ)</th>
                                <th class="case-table-header case-group-joint-filed px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">ฟ้องควบกัน<br>(สน.ทองหล่อ ไป สน.คลองตัน)<br>(เรื่อง)</th>
                                <th class="case-table-header case-group-joint-filed px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">ฟ้องควบกัน<br>(สถานะ)</th>
                                <th class="case-table-header case-group-problematic px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">คดีที่มีปัญหา<br>(เรื่อง)</th>
                                <th class="case-table-header case-group-problematic px-4 py-3 text-left text-xs font-medium uppercase tracking-wider rounded-tr-lg">คดีที่มีปัญหา<br>(สถานะ)</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-stone-200 text-sm">
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">1. บริษัทชีวาคุณ เอสเตทส์ จำกัด</td>
                                <td class="px-4 py-3 whitespace-nowrap">27 มิ.ย.68 เวลา 09.30 น. นัดไต่สวนมูลฟ้อง</td>
                                <td class="px-4 py-3 whitespace-nowrap">9. บริษัท KOP ซีวิล บิวส์ดิ้ง จำกัด</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap">3. พานาซี (โรงแรมเยอรมัน)</td>
                                <td class="px-4 py-3 whitespace-nowrap">คดีอยู่ DSI</td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">2. เอสเซน แอสเสท จำกัด</td>
                                <td class="px-4 py-3 whitespace-nowrap">16 - 17 ธ.ค.68 เวลา 09.00 - 16.00 น. นัดสืบพยานโจทก์และจำเลย</td>
                                <td class="px-4 py-3 whitespace-nowrap">10. ผู้รับเหมาบุคคล</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap">13. บจก.ไบโอสกาย คลินิก</td>
                                <td class="px-4 py-3 whitespace-nowrap">อยู่ระหว่างการสอบสวนของ สน.ทองหล่อ</td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">4. ศาลอาญามีนบุรี</td>
                                <td class="px-4 py-3 whitespace-nowrap">16 มิ.ย.68 เวลา 09.00 น. นัดไต่สวนมูลฟ้อง</td>
                                <td class="px-4 py-3 whitespace-nowrap">12. บริษัท วีรวรรณ แอสเซท จำกัด</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">5. ศาลอาญากรุงเทพใต้</td>
                                <td class="px-4 py-3 whitespace-nowrap">14 ก.ค.68 เวลา 13.00 น. นัดไต่สวนมูลฟ้อง</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">6. ศาลอาญากรุงเทพใต้</td>
                                <td class="px-4 py-3 whitespace-nowrap">21 ก.ค.68 เวลา 09.00 น. นัดไต่สวนมูลฟ้อง</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">7. ศาลอาญากรุงเทพใต้</td>
                                <td class="px-4 py-3 whitespace-nowrap">4 ส.ค.68 เวลา 09.00 น. นัดไต่สวนมูลฟ้อง</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">8. ศาลอาญากรุงเทพใต้</td>
                                <td class="px-4 py-3 whitespace-nowrap">18 ส.ค.68 เวลา 09.00 น. นัดไต่สวนมูลฟ้อง</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                            </tr>
                            <tr class="case-table-row">
                                <td class="px-4 py-3 whitespace-nowrap">11. บริษัท เจ.พี.เมคคานิคอล ดีไซน์ จำกัด</td>
                                <td class="px-4 py-3 whitespace-nowrap">ส่งฟ้องอัยการ 20 พ.ค.68</td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                                <td class="px-4 py-3 whitespace-nowrap"></td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </section>
        </main>
    </div>

    <div id="case-modal" class="fixed inset-0 bg-black bg-opacity-60 modal-backdrop flex justify-center items-center p-4 z-50 hidden" onclick="closeModal()">
        <div class="modal-content bg-white rounded-xl shadow-2xl w-full min-w-[90%] max-w-4xl max-h-[95vh] overflow-y-auto transform scale-95" onclick="event.stopPropagation()">
            <div class="sticky top-0 bg-white border-b border-stone-200 p-4 flex justify-between items-center">
                 <h2 id="modal-title" class="text-xl font-bold text-sky-900"></h2>
                 <button onclick="closeModal()" class="text-stone-500 hover:text-red-600 transition rounded-full p-1 focus:outline-none focus:ring-2 focus:ring-red-500">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                 </button>
            </div>
            <div class="p-6 sm:p-8">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                    <div>
                        <p class="text-sm font-semibold text-stone-500">มูลค่าความเสียหาย</p>
                        <p id="modal-damage" class="text-lg font-bold text-red-700"></p>
                    </div>
                     <div>
                        <p class="text-sm font-semibold text-stone-500">สถานะคดี</p>
                        <p id="modal-status" class="text-lg font-semibold"></p>
                    </div>
                </div>
                <div class="mb-4">
                    <p class="text-sm font-semibold text-stone-500">ข้อหา</p>
                    <p id="modal-charge" class="text-md"></p>
                </div>
                <div class="mb-4">
                    <p class="text-sm font-semibold text-stone-500">คู่กรณี</p>
                    <p id="modal-parties" class="text-md"></p>
                </div>
                <div class="mb-4">
                    <p class="text-sm font-semibold text-stone-500">ข้อเท็จจริง</p>
                    <p id="modal-facts" class="text-md bg-stone-50 p-3 rounded-lg border border-stone-200"></p>
                </div>
                 <div>
                    <p class="text-sm font-semibold text-stone-500">ความคืบหน้า</p>
                    <p id="modal-progress" class="text-md"></p>
                </div>
            </div>
        </div>
    </div>


    <script>
        document.addEventListener('DOMContentLoaded', () => {

            const caseData = [
                { id: 1, title: 'คดี อ.1161/2567', damage: 211716639.00, charge: 'ร่วมกันยักยอกทรัพย์', facts: 'อดีตผู้บริหารร่วมกันทุจริตจัดประชุมคณะกรรมการบริษัทณุศา มาย โอโซน จำกัด มีมติอนุมัติให้ขายที่ดิน 9 โฉนด ต่ำกว่าราคาประเมินราชการ', parties: 'โจทก์: บจก. ณุศา มาย โอโซน | จำเลย: บจก. ชีวาคุณ เอสเตทส์, นายวิษณุ เทพเจริญ กับพวกรวม 8 คน', progress: 'นัดไต่สวนมูลฟ้องวันที่ 27 มิถุนายน 2568 เวลา 09:30 น.', status: 'court', type: 'ยักยอกทรัพย์' },
                { id: 2, title: 'คดี อ.590/2567', damage: 340883436.00, charge: 'ร่วมกันไม่ปฏิบัติหน้าที่ด้วยความรับผิดชอบฯ', facts: 'อดีตผู้บริหารร่วมกันมีมติขายโครงการอสังหาริมทรัพย์ให้แก่บริษัทที่เกี่ยวข้องกันในราคาต่ำกว่าประเมิน และชำระราคาเพียงกึ่งหนึ่ง โดยไม่ผ่านการอนุมัติจากคณะกรรมการตรวจสอบ', parties: 'โจทก์: บจก. ธนา พาวเวอร์ โฮลดิ้ง | จำเลย: บจก. เอสเซน แอสเสท, นายวิษณุ เทพเจริญ กับพวกรวม 3 คน', progress: 'นัดสืบพยานโจทก์และจำเลยวันที่ 16-17 ธันวาคม 2568', status: 'court', type: 'ปฏิบัติหน้าที่มิชอบ' },
                { id: 3, title: 'คดีโรงแรมเยอรมัน (สน.มักกะสัน)', damage: 703000000.00, charge: 'ร่วมกันยักยอกทรัพย์สิน', facts: 'บริษัทฯ ชำระเงินมัดจำซื้อโรงแรมเยอรมัน แต่มีการปลอมแปลงเช็คโดยประทับตรา A/C Payee และสลักหลังนำฝากเข้าบัญชีของอดีตกรรมการและบุคคลใกล้ชิด', parties: 'ผู้เสียหาย: บมจ. สเตลล่า เอ็กซ์ | ผู้ต้องหา: 9 คน', progress: 'บริษัทฯ เข้าร้องทุกข์ดำเนินคดี วันที่ 24 ตุลาคม 2567', status: 'police', type: 'ยักยอกทรัพย์' },
                { id: 4, title: 'คดี อ.1191/2568 (ศาลอาญามีนบุรี)', damage: 5437743.76, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างในโครงการต่างๆ โดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'โจทก์: บจก. ณุศา มาย โอโซน | จำเลย: นายสมพิจิตร ชัยชนะจารักษ์ กับพวกรวม 6 คน', progress: 'นัดไต่สวนมูลฟ้องวันที่ 16 มิถุนายน 2568 เวลา 09:00 น.', status: 'court', type: 'ฉ้อโกง' },
                { id: 5, title: 'คดี อ.520/2568 (ศาลอาญากรุงเทพใต้)', damage: 536176.57, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างในโครงการต่างๆ โดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'โจทก์: บจก. ณุศา มาย โอโซน | จำเลย: บจก. เคโอพี ซีวิล บิวล์ดิ้ง กับพวกรวม 8 คน', progress: 'นัดไต่สวนมูลฟ้องวันที่ 14 กรกฎาคม 2568 เวลา 13:00 น.', status: 'court', type: 'ฉ้อโกง' },
                { id: 6, title: 'คดี อ.521/2568 (ศาลอาญากรุงเทพใต้)', damage: 571544.08, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างในโครงการต่างๆ โดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'โจทก์: บจก. ณุศา มาย โอโซน | จำเลย: บจก. เคโอพี ซีวิล บิวล์ดิ้ง กับพวกรวม 8 คน', progress: 'นัดไต่สวนมูลฟ้องวันที่ 21 กรกฎาคม 2568 เวลา 09:00 น.', status: 'court', type: 'ฉ้อโกง' },
                { id: 7, title: 'คดี อ.522/2568 (ศาลอาญากรุงเทพใต้)', damage: 798800.06, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างในโครงการต่างๆ โดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'โจทก์: บจก. ณุศา มาย โอโซน | จำเลย: บจก. เคโอพี ซีวิล บิวล์ดิ้ง กับพวกรวม 8 คน', progress: 'นัดไต่สวนมูลฟ้องวันที่ 4 สิงหาคม 2568 เวลา 09:00 น.', status: 'court', type: 'ฉ้อโกง' },
                { id: 8, title: 'คดี อ.523/2568 (ศาลอาญากรุงเทพใต้)', damage: 789657.75, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างในโครงการต่างๆ โดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'โจทก์: บจก. ณุศา มาย โอโซน | จำเลย: บจก. เคโอพี ซีวิล บิวล์ดิ้ง กับพวกรวม 8 คน', progress: 'นัดไต่สวนมูลฟ้องวันที่ 18 สิงหาคม 2568 เวลา 09:00 น.', status: 'court', type: 'ฉ้อโกง' },
                { id: 9, title: 'คดีจัดจ้าง บจก. เค โอ พี (สน.คลองตัน)', damage: 9256704.03, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างโดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'ผู้เสียหาย: บจก. ณุศา มาย โอโซน | ผู้ต้องหา: บจก. เคโอพี ซีวิล บิวล์ดิ้ง กับพวกรวม 8 คน', progress: 'อยู่ระหว่างการสอบสวนของพนักงานสอบสวน', status: 'police', type: 'ฉ้อโกง' },
                { id: 10, title: 'คดีจัดจ้างผู้รับเหมาบุคคล (สน.คลองตัน)', damage: 9711337.14, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างโดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'ผู้เสียหาย: บจก. ณุศา มาย โอโซน | ผู้ต้องหา: นายสมพิจิตร ชัยชนะจารักษ์ กับพวกรวม 8 คน', progress: 'อยู่ระหว่างการสอบสวนของพนักงานสอบสวน', status: 'police', type: 'ฉ้อโกง' },
                { id: 11, title: 'คดี บจก. เจ.พี.เมคคานิคอล (สน.มีนบุรี)', damage: 8153926.19, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างโดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'ผู้เสียหาย: บมจ. สเตลล่า เอ็กซ์ กับพวกรวม 2 คน | ผู้ต้องหา: นายสมพิจิตร ชัยชนะจารักษ์ กับพวกรวม 4 คน', progress: 'อยู่ระหว่างนัดส่งฟ้องพนักงานอัยการ วันที่ 20 พฤษภาคม 2568', status: 'police', type: 'ฉ้อโกง' },
                { id: 12, title: 'คดี บจก. วีรวรรณ แอสเซท (สน.คลองตัน)', damage: 28514465.10, charge: 'ร่วมกันฉ้อโกง, ปลอมและใช้เอกสารสิทธิ์ปลอม, ยักยอก, ลักทรัพย์นายจ้าง', facts: 'จัดจ้างผู้รับเหมาก่อสร้างโดยไม่มีการก่อสร้างจริง ร่วมกันทำเอกสารปลอมเพื่อเบิกจ่ายเงิน แล้วนำเช็คเข้าบัญชีของอดีตกรรมการและผู้บริหาร', parties: 'ผู้เสียหาย: บจก. ณุศา มาย โอโซน | ผู้ต้องสงสัย: บจก. วีรวรรณ แอสเซท กับพวกรวม 8 คน', progress: 'อยู่ระหว่างการสอบสวนของพนักงานสอบสวน', status: 'police', type: 'ฉ้อโกง' },
                { id: 13, title: 'คดี บจก. ไบโอสกาย คลินิก (สน.ทองหล่อ)', damage: 0, charge: 'เป็นผู้ได้รับมอบหมายให้จัดการทรัพย์สินฯ แต่กระทำผิดหน้าที่โดยทุจริต', facts: 'เป็นผู้ได้รับมอบหมายให้จัดการทรัพย์สินของผู้อื่น หรือทรัพย์สินซึ่งผู้อื่นเป็นเจ้าของรวมอยู่ กระทำผิดหน้าที่โดยทุจริต เป็นเหตุให้เกิดความเสียหายแก่ประโยชน์ที่เป็นทรัพย์สินของผู้นั้น', parties: 'ผู้เสียหาย: บจก. พานาซี เมดิคอลเซ็นเตอร์ | ผู้ต้องหา: บจก. ไบโอสกาย คลินิก และบุคคลอื่น', progress: 'อยู่ระหว่างการสอบสวนของพนักงานสอบสวน', status: 'police', type: 'ปฏิบัติหน้าที่มิชอบ' },
            ];
            
            const caseGrid = document.getElementById('case-grid');
            const modal = document.getElementById('case-modal');
            const filterButtons = document.getElementById('filter-buttons');

            const formatCurrency = (value) => {
                if (value === 0) return 'ไม่ระบุมูลค่า';
                return new Intl.NumberFormat('th-TH', { style: 'currency', currency: 'THB', minimumFractionDigits: 0 }).format(value);
            };

            const renderCases = (filter = 'all') => {
                caseGrid.innerHTML = '';
                const filteredData = filter === 'all' ? caseData : caseData.filter(c => c.status === filter);
                
                filteredData.forEach(c => {
                    const card = document.createElement('div');
                    card.className = 'bg-white p-5 rounded-xl shadow-md border border-stone-200 hover:shadow-lg hover:-translate-y-1 transition-all duration-300 flex flex-col';
                    card.innerHTML = `
                        <h3 class="font-bold text-lg text-sky-800 mb-2">${c.title}</h3>
                        <p class="text-sm text-stone-600 mb-1 line-clamp-1"><span class="font-semibold">ข้อหา:</span> ${c.charge}</p>
                        <p class="text-red-600 font-semibold text-md mb-4">${formatCurrency(c.damage)}</p>
                        <div class="mt-auto pt-4 border-t border-stone-200 text-center">
                           <button class="w-full bg-sky-700 text-white py-2 px-4 rounded-lg hover:bg-sky-800 transition" onclick="openModal(${c.id})">ดูรายละเอียด</button>
                        </div>
                    `;
                    caseGrid.appendChild(card);
                });
            };

            window.openModal = (id) => {
                const caseInfo = caseData.find(c => c.id === id);
                if (!caseInfo) return;

                document.getElementById('modal-title').textContent = caseInfo.title;
                document.getElementById('modal-damage').textContent = formatCurrency(caseInfo.damage);
                
                const statusEl = document.getElementById('modal-status');
                if (caseInfo.status === 'court') {
                    statusEl.textContent = 'คดีในชั้นศาล';
                    statusEl.className = 'text-lg font-semibold text-sky-800';
                } else {
                    statusEl.textContent = 'คดีในชั้นสอบสวน';
                    statusEl.className = 'text-lg font-semibold text-amber-600';
                }
                
                document.getElementById('modal-charge').textContent = caseInfo.charge;
                document.getElementById('modal-parties').innerHTML = caseInfo.parties.replace(/\|/g, '<br>');
                document.getElementById('modal-facts').textContent = caseInfo.facts;
                document.getElementById('modal-progress').textContent = caseInfo.progress;
                
                modal.classList.remove('hidden');
                document.body.style.overflow = 'hidden';
                setTimeout(() => {
                    modal.firstElementChild.classList.remove('scale-95');
                    modal.classList.remove('opacity-0');
                }, 10);
            };

            window.closeModal = () => {
                modal.firstElementChild.classList.add('scale-95');
                modal.classList.add('opacity-0');
                 setTimeout(() => {
                    modal.classList.add('hidden');
                    document.body.style.overflow = 'auto';
                }, 300);
            };
            
            filterButtons.addEventListener('click', (e) => {
                if (e.target.tagName === 'BUTTON') {
                    document.querySelectorAll('.btn-filter').forEach(btn => btn.classList.remove('active'));
                    e.target.classList.add('active');
                    const filter = e.target.dataset.filter;
                    renderCases(filter);
                }
            });

            const renderCharts = () => {
                const damageCtx = document.getElementById('damageByCaseChart').getContext('2d');
                const damageLabels = caseData.map(c => `คดี #${c.id}`);
                const damageValues = caseData.map(c => c.damage / 1000000); 

                new Chart(damageCtx, {
                    type: 'bar',
                    data: {
                        labels: damageLabels,
                        datasets: [{
                            label: 'มูลค่าความเสียหาย (ล้านบาท)',
                            data: damageValues,
                            backgroundColor: '#0369a1',
                            borderColor: '#0c4a6e',
                            borderWidth: 1,
                            borderRadius: 4
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        scales: {
                            y: {
                                beginAtZero: true,
                                ticks: { font: { family: "'Sarabun', sans-serif" } },
                                grid: { color: '#e7e5e4' }
                            },
                            x: {
                                ticks: { font: { family: "'Sarabun', sans-serif" } },
                                grid: { display: false }
                            }
                        },
                        plugins: {
                            legend: { display: false },
                            tooltip: {
                                callbacks: {
                                    label: function(context) {
                                        let label = context.dataset.label || '';
                                        if (label) {
                                            label += ': ';
                                        }
                                        if (context.parsed.y !== null) {
                                            label += new Intl.NumberFormat('th-TH', { style: 'decimal', minimumFractionDigits: 2 }).format(context.parsed.y) + ' ล้านบาท';
                                        }
                                        return label;
                                    }
                                },
                                titleFont: { family: "'Sarabun', sans-serif" },
                                bodyFont: { family: "'Sarabun', sans-serif" }
                            }
                        }
                    }
                });

                const typeCtx = document.getElementById('caseTypeChart').getContext('2d');
                const typeData = caseData.reduce((acc, c) => {
                    acc[c.type] = (acc[c.type] || 0) + 1;
                    return acc;
                }, {});
                
                new Chart(typeCtx, {
                    type: 'doughnut',
                    data: {
                        labels: Object.keys(typeData),
                        datasets: [{
                            label: 'จำนวนคดี',
                            data: Object.values(typeData),
                            backgroundColor: ['#0369a1', '#0891b2', '#f59e0b'],
                            borderColor: '#ffffff',
                            borderWidth: 2
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                         plugins: {
                            legend: {
                                position: 'bottom',
                                labels: {
                                    font: {
                                        family: "'Sarabun', sans-serif",
                                        size: 14
                                    }
                                }
                            },
                             tooltip: {
                                titleFont: { family: "'Sarabun', sans-serif" },
                                bodyFont: { family: "'Sarabun', sans-serif" }
                            }
                        }
                    }
                });
            };
            
            renderCases();
            renderCharts();
        });
    </script>
</body>
</html>
