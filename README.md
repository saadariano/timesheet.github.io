<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cartellino Presenze</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- jsPDF and jsPDF-AutoTable libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            color: #1f2937;
        }
        .modal-overlay {
            background-color: rgba(0, 0, 0, 0.5);
        }
        .calendar-grid {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            gap: 0.5rem;
        }
        .day-cell {
            min-height: 80px;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .day-cell:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .message-box {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 100;
            background-color: white;
            padding: 24px;
            border-radius: 12px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
            text-align: center;
        }
    </style>
</head>
<body class="p-4 sm:p-8">
    <!-- Message Box -->
    <div id="messageBox" class="hidden message-box">
        <p id="messageText" class="text-lg font-semibold text-gray-800 mb-4"></p>
        <button id="messageBoxCloseBtn" class="bg-blue-600 text-white px-6 py-2 rounded-lg font-bold hover:bg-blue-700 transition-colors duration-200">OK</button>
    </div>

    <div id="main-app" class="max-w-4xl mx-auto bg-white rounded-xl shadow-lg p-6 sm:p-8">
        <header class="mb-6 border-b pb-4">
            <h1 class="text-3xl font-bold text-center text-gray-900">Cartellino Presenze di Saad Mounir</h1>
        </header>

        <main class="space-y-6">
            <div class="flex flex-col sm:flex-row items-center justify-between space-y-4 sm:space-y-0 sm:space-x-4">
                <div class="flex items-center space-x-2">
                    <button id="prevMonthBtn" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-2 px-4 rounded-lg transition-colors duration-200">
                        &lt;
                    </button>
                    <h2 id="currentMonthYear" class="text-2xl font-bold text-gray-800 w-40 text-center"></h2>
                    <button id="nextMonthBtn" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-2 px-4 rounded-lg transition-colors duration-200">
                        &gt;
                    </button>
                </div>
                <div class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-2 w-full sm:w-auto">
                    <select id="monthSelect" class="block w-full sm:w-auto px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors duration-200"></select>
                    <input type="number" id="yearInput" class="block w-full sm:w-auto px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors duration-200" min="2000" max="2100">
                </div>
            </div>

            <!-- New Buttons Section -->
            <div class="flex flex-col sm:flex-row justify-center items-center space-y-4 sm:space-y-0 sm:space-x-4 mt-6">
                <button id="createPdfBtn" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-2 px-4 rounded-lg transition-colors duration-200 flex items-center space-x-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-9.586a1 1 0 011.414 0L9 8.586V3a1 1 0 012 0v5.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
                    </svg>
                    <span>Crea PDF</span>
                </button>
                <button id="exportBtn" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-2 px-4 rounded-lg transition-colors duration-200 flex items-center space-x-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-9.586a1 1 0 011.414 0L9 8.586V3a1 1 0 012 0v5.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
                    </svg>
                    <span>Esporta</span>
                </button>
                <label for="importFile" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold py-2 px-4 rounded-lg cursor-pointer transition-colors duration-200 flex items-center space-x-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM6.293 6.707a1 1 0 010-1.414l3-3a1 1 0 011.414 0l3 3a1 1 0 01-1.414 1.414L11 5.414V13a1 1 0 11-2 0V5.414L6.707 6.707a1 1 0 01-1.414 0z" clip-rule="evenodd" />
                    </svg>
                    <span>Importa</span>
                </label>
                <input type="file" id="importFile" class="hidden" accept=".json">
            </div>

            <div id="loading" class="text-center text-gray-500 font-medium hidden">Caricamento...</div>
            <div id="calendar" class="calendar-grid"></div>

            <div class="bg-gray-100 p-6 rounded-xl shadow-inner mt-6">
                <h3 class="text-xl font-bold mb-4">Riepilogo Mensile</h3>
                <div id="summary" class="grid grid-cols-2 sm:grid-cols-4 gap-4 text-center">
                    <div class="p-3 bg-green-100 border border-green-200 rounded-lg shadow-md">
                        <p class="text-sm text-green-800 font-medium">Ore Ordinarie</p>
                        <p id="summaryRegular" class="text-lg font-bold text-green-700">0</p>
                    </div>
                    <div class="p-3 bg-yellow-100 border border-yellow-200 rounded-lg shadow-md">
                        <p class="text-sm text-yellow-800 font-medium">Ore di Straordinario</p>
                        <p id="summaryOvertime" class="text-lg font-bold text-yellow-700">0</p>
                    </div>
                    <div class="p-3 bg-red-100 border border-red-200 rounded-lg shadow-md">
                        <p class="text-sm text-red-800 font-medium">Ore Mancanti</p>
                        <p id="summaryMissing" class="text-lg font-bold text-red-700">0</p>
                    </div>
                    <div class="p-3 bg-red-100 border border-red-200 rounded-lg shadow-md">
                        <p class="text-sm text-red-800 font-medium">Malattia</p>
                        <p id="summarySickness" class="text-lg font-bold text-red-700">0</p>
                    </div>
                    <div class="p-3 bg-blue-100 border border-blue-200 rounded-lg shadow-md">
                        <p class="text-sm text-blue-800 font-medium">Ferie</p>
                        <p id="summaryHolidays" class="text-lg font-bold text-blue-700">0</p>
                    </div>
                    <div class="p-3 bg-red-100 border border-red-200 rounded-lg shadow-md">
                        <p class="text-sm text-red-800 font-medium">Assenze</p>
                        <p id="summaryAbsence" class="text-lg font-bold text-red-700">0</p>
                    </div>
                    <div class="p-3 bg-indigo-100 border border-indigo-200 rounded-lg shadow-md">
                        <p class="text-sm text-indigo-800 font-medium">Permesso</p>
                        <p id="summaryPermesso" class="text-lg font-bold text-indigo-700">0</p>
                    </div>
                    <div class="p-3 bg-pink-100 border border-pink-200 rounded-lg shadow-md">
                        <p class="text-sm text-pink-800 font-medium">Ritardi</p>
                        <p id="summaryLates" class="text-lg font-bold text-pink-700">0</p>
                    </div>
                </div>
                <!-- Total Working Hours moved to its own centered container -->
                <div class="flex justify-center mt-4">
                    <div class="p-3 bg-blue-100 border border-blue-200 rounded-lg shadow-md w-full sm:w-1/2 md:w-1/3 text-center">
                        <p class="text-sm text-blue-800 font-medium">Totale Ore Lavorate</p>
                        <p id="summaryTotal" class="text-lg font-bold text-blue-700">0</p>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <!-- Entry Modal -->
    <div id="entryModal" class="fixed inset-0 z-50 hidden flex items-center justify-center p-4 modal-overlay">
        <div class="bg-white rounded-xl shadow-2xl p-6 w-full max-w-lg space-y-4 transform transition-transform duration-300 scale-95">
            <h3 class="text-2xl font-bold text-gray-800 text-center" id="modalDateTitle"></h3>
            <form id="entryForm" class="space-y-4">
                <div class="space-y-2">
                    <label class="block text-sm font-medium text-gray-700">Turni</label>
                    <div class="grid grid-cols-1 sm:grid-cols-3 gap-2">
                        <button type="button" data-shift="morning" class="shift-btn bg-gray-200 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-300 transition-colors duration-200">Mattina (06:00 - 14:00)</button>
                        <button type="button" data-shift="afternoon" class="shift-btn bg-gray-200 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-300 transition-colors duration-200">Pomeriggio (14:00 - 22:00)</button>
                        <button type="button" data-shift="night" class="shift-btn bg-gray-200 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-300 transition-colors duration-200">Notte (22:00 - 06:00)</button>
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                        <label for="clockIn" class="block text-sm font-medium text-gray-700">Ingresso</label>
                        <input type="time" id="clockIn" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                    </div>
                    <div>
                        <label for="clockOut" class="block text-sm font-medium text-gray-700">Uscita</label>
                        <input type="time" id="clockOut" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                        <input type="checkbox" id="overtime" class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500">
                        <label for="overtime" class="text-sm font-medium text-gray-700 ml-2">Straordinario</label>
                    </div>
                    <div>
                        <input type="checkbox" id="lates" class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500">
                        <label for="lates" class="text-sm font-medium text-gray-700 ml-2">Ritardi</label>
                    </div>
                </div>
                
                <div class="space-y-2">
                    <label class="block text-sm font-medium text-gray-700">Opzioni Assenza</label>
                    <div class="flex flex-wrap gap-2">
                        <label class="flex items-center bg-gray-100 rounded-full px-4 py-2 text-sm font-medium text-gray-700 cursor-pointer hover:bg-gray-200 transition-colors duration-200">
                            <input type="checkbox" name="leave" value="sickness" class="h-4 w-4 text-red-600 border-gray-300 rounded focus:ring-red-500 mr-2">
                            Malattia
                        </label>
                        <label class="flex items-center bg-gray-100 rounded-full px-4 py-2 text-sm font-medium text-gray-700 cursor-pointer hover:bg-gray-200 transition-colors duration-200">
                            <input type="checkbox" name="leave" value="absence" class="h-4 w-4 text-purple-600 border-gray-300 rounded focus:ring-purple-500 mr-2">
                            Assenza
                        </label>
                        <label class="flex items-center bg-gray-100 rounded-full px-4 py-2 text-sm font-medium text-gray-700 cursor-pointer hover:bg-gray-200 transition-colors duration-200">
                            <input type="checkbox" name="leave" value="holiday" class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500 mr-2">
                            Ferie
                        </label>
                        <label class="flex items-center bg-gray-100 rounded-full px-4 py-2 text-sm font-medium text-gray-700 cursor-pointer hover:bg-gray-200 transition-colors duration-200">
                            <input type="checkbox" name="leave" value="permesso" class="h-4 w-4 text-indigo-600 border-gray-300 rounded focus:ring-indigo-500 mr-2">
                            Permesso
                        </label>
                    </div>
                </div>

                <div class="flex justify-end space-x-2">
                    <button type="button" id="deleteEntryBtn" class="bg-red-500 text-white px-6 py-2 rounded-lg font-bold hover:bg-red-600 transition-colors duration-200">Elimina</button>
                    <button type="button" id="closeModalBtn" class="bg-gray-300 text-gray-800 px-6 py-2 rounded-lg font-bold hover:bg-gray-400 transition-colors duration-200">Annulla</button>
                    <button type="submit" class="bg-blue-600 text-white px-6 py-2 rounded-lg font-bold hover:bg-blue-700 transition-colors duration-200">Salva</button>
                </div>
            </form>
        </div>
    </div>
    
    <!-- PDF Selection Modal -->
    <div id="pdfSelectionModal" class="fixed inset-0 z-50 hidden flex items-center justify-center p-4 modal-overlay">
        <div class="bg-white rounded-xl shadow-2xl p-6 w-full max-w-sm space-y-4 transform transition-transform duration-300 scale-95">
            <h3 class="text-2xl font-bold text-gray-800 text-center">Seleziona Mese e Anno</h3>
            <div class="space-y-4">
                <div class="flex flex-col space-y-2">
                    <label for="pdfMonthSelect" class="text-sm font-medium text-gray-700">Mese</label>
                    <select id="pdfMonthSelect" class="block w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors duration-200"></select>
                </div>
                <div class="flex flex-col space-y-2">
                    <label for="pdfYearInput" class="text-sm font-medium text-gray-700">Anno</label>
                    <input type="number" id="pdfYearInput" class="block w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors duration-200" min="2000" max="2100">
                </div>
            </div>
            <div class="flex justify-end space-x-2">
                <button type="button" id="closePdfModalBtn" class="bg-gray-300 text-gray-800 px-6 py-2 rounded-lg font-bold hover:bg-gray-400 transition-colors duration-200">Annulla</button>
                <button type="button" id="generatePdfBtn" class="bg-blue-600 text-white px-6 py-2 rounded-lg font-bold hover:bg-blue-700 transition-colors duration-200">Genera PDF</button>
            </div>
        </div>
    </div>

    <script>
        window.onload = function() {
            let currentYear = new Date().getFullYear();
            let currentMonth = new Date().getMonth();

            const calendarEl = document.getElementById('calendar');
            const summaryEl = document.getElementById('summary');
            const monthSelectEl = document.getElementById('monthSelect');
            const yearInputEl = document.getElementById('yearInput');
            const currentMonthYearEl = document.getElementById('currentMonthYear');
            const prevMonthBtn = document.getElementById('prevMonthBtn');
            const nextMonthBtn = document.getElementById('nextMonthBtn');

            const entryModalEl = document.getElementById('entryModal');
            const entryFormEl = document.getElementById('entryForm');
            const modalDateTitleEl = document.getElementById('modalDateTitle');
            const closeModalBtn = document.getElementById('closeModalBtn');
            const deleteEntryBtn = document.getElementById('deleteEntryBtn');
            const shiftButtons = document.querySelectorAll('.shift-btn');
            const clockInEl = document.getElementById('clockIn');
            const clockOutEl = document.getElementById('clockOut');
            const overtimeEl = document.getElementById('overtime');
            const latesEl = document.getElementById('lates');
            
            const createPdfBtn = document.getElementById('createPdfBtn');
            const exportBtn = document.getElementById('exportBtn');
            const importFile = document.getElementById('importFile');

            const messageBox = document.getElementById('messageBox');
            const messageText = document.getElementById('messageText');
            const messageBoxCloseBtn = document.getElementById('messageBoxCloseBtn');

            // New PDF modal elements
            const pdfSelectionModal = document.getElementById('pdfSelectionModal');
            const pdfMonthSelect = document.getElementById('pdfMonthSelect');
            const pdfYearInput = document.getElementById('pdfYearInput');
            const closePdfModalBtn = document.getElementById('closePdfModalBtn');
            const generatePdfBtn = document.getElementById('generatePdfBtn');

            const MONTH_NAMES = ["Gennaio", "Febbraio", "Marzo", "Aprile", "Maggio", "Giugno", "Luglio", "Agosto", "Settembre", "Ottobre", "Novembre", "Dicembre"];
            const SHIFTS = {
                'morning': { clockIn: '06:00', clockOut: '14:00', hours: 8, label: '06:00 - 14:00' },
                'afternoon': { clockIn: '14:00', clockOut: '22:00', hours: 8, label: '14:00 - 22:00' },
                'night': { clockIn: '22:00', clockOut: '06:00', hours: 8, label: '22:00 - 06:00' },
            };

            let activeDayData = null;
            let activeDate = null;
            const storageKeyPrefix = 'timecard-';

            function showMessage(message) {
                messageText.textContent = message;
                messageBox.classList.remove('hidden');
            }

            function hideMessage() {
                messageBox.classList.add('hidden');
            }

            function initApp() {
                populateMonthYearSelectors();
                yearInputEl.value = currentYear;
                monthSelectEl.value = currentMonth;
                
                yearInputEl.addEventListener('change', updateCalendar);
                monthSelectEl.addEventListener('change', updateCalendar);
                prevMonthBtn.addEventListener('click', () => changeMonth(-1));
                nextMonthBtn.addEventListener('click', () => changeMonth(1));
                closeModalBtn.addEventListener('click', closeModal);
                deleteEntryBtn.addEventListener('click', deleteEntry);
                entryFormEl.addEventListener('submit', handleSaveEntry);
                messageBoxCloseBtn.addEventListener('click', hideMessage);
                createPdfBtn.addEventListener('click', openPdfModal);
                exportBtn.addEventListener('click', exportData);
                importFile.addEventListener('change', importData);

                closePdfModalBtn.addEventListener('click', () => pdfSelectionModal.classList.add('hidden'));
                generatePdfBtn.addEventListener('click', handleGeneratePdf);

                shiftButtons.forEach(btn => {
                    btn.addEventListener('click', (e) => {
                        const shift = e.target.dataset.shift;
                        const { clockIn, clockOut } = SHIFTS[shift];
                        clockInEl.value = clockIn;
                        clockOutEl.value = clockOut;
                        // Uncheck leave options when a shift is selected
                        document.querySelectorAll('input[name="leave"]').forEach(checkbox => checkbox.checked = false);
                    });
                });

                document.querySelectorAll('input[name="leave"]').forEach(checkbox => {
                    checkbox.addEventListener('change', (e) => {
                        if (e.target.checked) {
                             // Clear clock in/out times and checkboxes when a leave option is selected
                            clockInEl.value = '';
                            clockOutEl.value = '';
                            overtimeEl.checked = false;
                            latesEl.checked = false;
                        }
                    });
                });

                loadDataAndRender();
            }

            function populateMonthYearSelectors() {
                const monthsHtml = MONTH_NAMES.map((month, index) => `<option value="${index}">${month}</option>`).join('');
                monthSelectEl.innerHTML = monthsHtml;
                pdfMonthSelect.innerHTML = monthsHtml;
                yearInputEl.value = currentYear;
                pdfYearInput.value = currentYear;
            }

            function updateCalendar() {
                currentMonth = parseInt(monthSelectEl.value);
                currentYear = parseInt(yearInputEl.value);
                loadDataAndRender();
            }

            function changeMonth(delta) {
                currentMonth += delta;
                if (currentMonth < 0) {
                    currentMonth = 11;
                    currentYear--;
                } else if (currentMonth > 11) {
                    currentMonth = 0;
                    currentYear++;
                }
                monthSelectEl.value = currentMonth;
                yearInputEl.value = currentYear;
                updateCalendar();
            }

            function loadDataAndRender() {
                const key = `${storageKeyPrefix}${currentYear}-${currentMonth + 1}`;
                const storedData = localStorage.getItem(key);
                activeDayData = storedData ? JSON.parse(storedData) : {};
                renderCalendar();
                renderSummary();
            }

            function saveData() {
                const key = `${storageKeyPrefix}${currentYear}-${currentMonth + 1}`;
                localStorage.setItem(key, JSON.stringify(activeDayData));
                loadDataAndRender();
            }

            function renderCalendar() {
                calendarEl.innerHTML = '';
                currentMonthYearEl.textContent = `${MONTH_NAMES[currentMonth]} ${currentYear}`;
                
                const firstDayOfMonth = new Date(currentYear, currentMonth, 1).getDay();
                const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate();
                
                const weekdays = ["Dom", "Lun", "Mar", "Mer", "Gio", "Ven", "Sab"];
                weekdays.forEach(day => {
                    const dayHeader = document.createElement('div');
                    dayHeader.className = 'text-center font-bold text-gray-700';
                    dayHeader.textContent = day;
                    calendarEl.appendChild(dayHeader);
                });

                for (let i = 0; i < firstDayOfMonth; i++) {
                    const emptyCell = document.createElement('div');
                    calendarEl.appendChild(emptyCell);
                }

                for (let day = 1; day <= daysInMonth; day++) {
                    const dayCell = document.createElement('div');
                    dayCell.className = 'day-cell rounded-lg p-2 text-center text-sm relative border shadow-sm flex flex-col items-center justify-start';
                    dayCell.dataset.day = day;
                    
                    const dayNumber = document.createElement('p');
                    dayNumber.className = 'text-lg font-bold text-gray-800';
                    dayNumber.textContent = day;
                    dayCell.appendChild(dayNumber);

                    const data = activeDayData ? activeDayData[day] : null;
                    
                    if (data) {
                        let statusContent = '';
                        let statusClass = '';
                        
                        // Check for missing hours first to apply red background
                        if (data.regularHours < 8 && data.regularHours > 0) {
                             dayCell.classList.add('bg-red-200', 'border-red-400');
                             statusContent = `${data.clockIn} - ${data.clockOut}`;
                             statusClass = 'text-red-800';
                        } else if (data.sickness) {
                            dayCell.classList.add('bg-red-200', 'border-red-400');
                            statusContent = 'Malattia';
                            statusClass = 'text-red-800';
                        } else if (data.holiday) {
                            dayCell.classList.add('bg-blue-200', 'border-blue-400');
                            statusContent = 'Ferie';
                            statusClass = 'text-blue-800';
                        } else if (data.permesso) {
                            dayCell.classList.add('bg-purple-200', 'border-purple-400');
                            statusContent = 'Permesso';
                            statusClass = 'text-purple-800';
                        } else if (data.absence) {
                            dayCell.classList.add('bg-red-200', 'border-red-400');
                            statusContent = 'Assenza';
                            statusClass = 'text-red-800';
                        } else if (data.clockIn && data.clockOut) {
                            const inTime = data.clockIn;
                            let shiftLabel = '';
                            if (inTime >= '06:00' && inTime < '14:00') {
                                dayCell.classList.add('bg-green-200', 'border-green-400');
                                shiftLabel = SHIFTS.morning.label;
                            } else if (inTime >= '14:00' && inTime < '22:00') {
                                dayCell.classList.add('bg-orange-200', 'border-orange-400');
                                shiftLabel = SHIFTS.afternoon.label;
                            } else {
                                dayCell.classList.add('bg-blue-200', 'border-blue-400');
                                shiftLabel = SHIFTS.night.label;
                            }
                            statusContent = shiftLabel;
                            statusClass = 'text-gray-800';
                        } else {
                            dayCell.classList.add('bg-gray-50', 'border-gray-200');
                        }

                        if (statusContent) {
                            const statusText = document.createElement('p');
                            statusText.className = `text-xs mt-1 font-medium ${statusClass}`;
                            statusText.textContent = statusContent;
                            dayCell.appendChild(statusText);
                        }

                        // Add badges for specific conditions
                        if (data.overtime) {
                            const overtimeBadge = document.createElement('span');
                            overtimeBadge.className = 'text-xs font-semibold text-yellow-800 bg-yellow-300 px-2 py-0.5 rounded-full mt-1';
                            overtimeBadge.textContent = 'OT';
                            dayCell.appendChild(overtimeBadge);
                        }
                        if (data.lates) {
                            const latesBadge = document.createElement('span');
                            latesBadge.className = 'text-xs font-semibold text-pink-800 bg-pink-300 px-2 py-0.5 rounded-full mt-1';
                            latesBadge.textContent = 'Ritardo';
                            dayCell.appendChild(latesBadge);
                        }
                    } else {
                        dayCell.classList.add('bg-gray-50', 'border-gray-200');
                    }
                    
                    dayCell.addEventListener('click', () => openModal(day));
                    calendarEl.appendChild(dayCell);
                }
            }

            function renderSummary() {
                if (!activeDayData) {
                    resetSummary();
                    return;
                }

                let regularHours = 0;
                let overtimeHours = 0;
                let missingHours = 0;
                let sicknessDays = 0;
                let holidaysDays = 0;
                let absenceDays = 0;
                let permessoDays = 0;
                let latesCount = 0;

                for (const day in activeDayData) {
                    const data = activeDayData[day];
                    if (data.sickness) sicknessDays++;
                    if (data.holiday) holidaysDays++;
                    if (data.permesso) permessoDays++;
                    if (data.absence) absenceDays++;
                    if (data.lates) latesCount++;
                    if (data.regularHours) regularHours += data.regularHours;
                    if (data.regularHours < 8 && data.regularHours > 0) {
                        missingHours += (8 - data.regularHours);
                    }
                    if (data.overtimeHours) overtimeHours += data.overtimeHours;
                }
                
                const totalHours = regularHours + overtimeHours;

                document.getElementById('summaryRegular').textContent = regularHours.toFixed(1);
                document.getElementById('summaryOvertime').textContent = overtimeHours.toFixed(1);
                document.getElementById('summaryMissing').textContent = missingHours.toFixed(1);
                document.getElementById('summaryTotal').textContent = totalHours.toFixed(1);
                document.getElementById('summarySickness').textContent = sicknessDays;
                document.getElementById('summaryHolidays').textContent = holidaysDays;
                document.getElementById('summaryAbsence').textContent = absenceDays;
                document.getElementById('summaryPermesso').textContent = permessoDays;
                document.getElementById('summaryLates').textContent = latesCount;
            }

            function resetSummary() {
                document.getElementById('summaryRegular').textContent = 0;
                document.getElementById('summaryOvertime').textContent = 0;
                document.getElementById('summaryMissing').textContent = 0;
                document.getElementById('summaryTotal').textContent = 0;
                document.getElementById('summarySickness').textContent = 0;
                document.getElementById('summaryHolidays').textContent = 0;
                document.getElementById('summaryAbsence').textContent = 0;
                document.getElementById('summaryPermesso').textContent = 0;
                document.getElementById('summaryLates').textContent = 0;
            }
            
            function openModal(day) {
                activeDate = day;
                modalDateTitleEl.textContent = `${MONTH_NAMES[currentMonth]} ${day}, ${currentYear}`;
                entryFormEl.reset();
                
                const data = activeDayData[day];
                if (data) {
                    clockInEl.value = data.clockIn || '';
                    clockOutEl.value = data.clockOut || '';
                    overtimeEl.checked = data.overtime || false;
                    latesEl.checked = data.lates || false;

                    const leaveCheckboxes = document.querySelectorAll('input[name="leave"]');
                    leaveCheckboxes.forEach(checkbox => {
                        checkbox.checked = data[checkbox.value] || false;
                    });
                }
                entryModalEl.classList.remove('hidden');
            }

            function closeModal() {
                entryModalEl.classList.add('hidden');
                activeDate = null;
            }

            function handleSaveEntry(e) {
                e.preventDefault();
                if (!activeDate) return;

                const leaveCheckboxes = document.querySelectorAll('input[name="leave"]');
                const data = {
                    clockIn: clockInEl.value,
                    clockOut: clockOutEl.value,
                    overtime: overtimeEl.checked,
                    lates: latesEl.checked,
                    sickness: leaveCheckboxes[0].checked,
                    absence: leaveCheckboxes[1].checked,
                    holiday: leaveCheckboxes[2].checked,
                    permesso: leaveCheckboxes[3].checked,
                };

                let regularHours = 0;
                let overtimeHours = 0;

                if (data.clockIn && data.clockOut) {
                    const inTime = new Date(`1970-01-01T${data.clockIn}:00`);
                    let outTime = new Date(`1970-01-01T${data.clockOut}:00`);

                    if (outTime < inTime) {
                        outTime = new Date(outTime.getTime() + 24 * 60 * 60 * 1000);
                    }

                    const diffInMs = outTime - inTime;
                    const hours = diffInMs / (1000 * 60 * 60);
                    regularHours = Math.max(0, Math.min(8, hours));
                    overtimeHours = Math.max(0, hours - 8);
                }
                
                data.regularHours = regularHours;
                data.overtimeHours = overtimeHours;
                
                if (!activeDayData) {
                    activeDayData = {};
                }
                activeDayData[activeDate] = data;

                saveData();
                closeModal();
            }
            
            function deleteEntry() {
                if (!activeDate) return;

                if (activeDayData && activeDayData[activeDate]) {
                    delete activeDayData[activeDate];
                    saveData();
                    closeModal();
                }
            }

            function openPdfModal() {
                pdfMonthSelect.value = currentMonth;
                pdfYearInput.value = currentYear;
                pdfSelectionModal.classList.remove('hidden');
            }

            function handleGeneratePdf() {
                const { jsPDF } = window.jspdf;
                const pdfMonth = parseInt(pdfMonthSelect.value);
                const pdfYear = parseInt(pdfYearInput.value);

                const key = `${storageKeyPrefix}${pdfYear}-${pdfMonth + 1}`;
                const storedData = localStorage.getItem(key);
                const dataToPrint = storedData ? JSON.parse(storedData) : {};
                
                let regularHours = 0;
                let overtimeHours = 0;
                let missingHours = 0;
                let sicknessDays = 0;
                let holidaysDays = 0;
                let absenceDays = 0;
                let permessoDays = 0;
                let latesCount = 0;

                for (const day in dataToPrint) {
                    const data = dataToPrint[day];
                    if (data.sickness) sicknessDays++;
                    if (data.holiday) holidaysDays++;
                    if (data.permesso) permessoDays++;
                    if (data.absence) absenceDays++;
                    if (data.lates) latesCount++;
                    if (data.regularHours) regularHours += data.regularHours;
                    if (data.regularHours < 8 && data.regularHours > 0) {
                        missingHours += (8 - data.regularHours);
                    }
                    if (data.overtimeHours) overtimeHours += data.overtimeHours;
                }
                
                const totalHours = regularHours + overtimeHours;

                const doc = new jsPDF('p', 'pt', 'a4');
                
                doc.setFont('Helvetica', 'normal');
                
                // Add the header to the PDF
                doc.setFontSize(18);
                doc.text(`Cartellino Presenze - ${MONTH_NAMES[pdfMonth]} ${pdfYear}`, doc.internal.pageSize.getWidth() / 2, 40, { align: 'center' });

                // Define the table data
                const head = [['Giorno', 'Data', 'Ore', 'Ingresso', 'Uscita', 'Straordinario', 'Ritardi', 'Stato']];
                const body = [];
                const daysInMonth = new Date(pdfYear, pdfMonth + 1, 0).getDate();
                const weekdays = ["Dom", "Lun", "Mar", "Mer", "Gio", "Ven", "Sab"];

                for (let day = 1; day <= daysInMonth; day++) {
                    const date = new Date(pdfYear, pdfMonth, day);
                    const dayOfWeek = weekdays[date.getDay()];
                    const dayData = dataToPrint[day] || {};
                    
                    let status = [];
                    if (dayData.sickness) status.push('Malattia');
                    if (dayData.holiday) status.push('Ferie');
                    if (dayData.permesso) status.push('Permesso');
                    if (dayData.absence) status.push('Assenza');
                    if (dayData.lates) status.push('Ritardo');
                    if (dayData.overtimeHours > 0) status.push('Straordinario');
                    if (!dayData.sickness && !dayData.holiday && !dayData.permesso && !dayData.absence) {
                        if (dayData.regularHours > 0) {
                            status.push('Normale');
                        } else if (dayData.regularHours === 0 && (dayData.clockIn || dayData.clockOut)) {
                            status.push('Errore');
                        } else {
                            status.push('Non inserito');
                        }
                    }

                    body.push([
                        dayOfWeek,
                        day.toString(),
                        dayData.regularHours ? dayData.regularHours.toFixed(1) : '0',
                        dayData.clockIn || '-',
                        dayData.clockOut || '-',
                        dayData.overtimeHours ? dayData.overtimeHours.toFixed(1) : '0',
                        dayData.lates ? 'Sì' : 'No',
                        status.join(', ')
                    ]);
                }

                doc.autoTable({
                    startY: 60,
                    head: head,
                    body: body,
                    theme: 'grid',
                    styles: { fontSize: 7, cellPadding: 2, halign: 'center' },
                    headStyles: { fillColor: '#E5E7EB', textColor: '#4B5563', fontStyle: 'bold' }
                });

                // Add the summary to the PDF after the table
                const finalY = doc.autoTable.previous.finalY;
                doc.setFontSize(12);
                doc.text('Riepilogo Mensile', 40, finalY + 20);

                const summaryData = [
                    ['Ore Ordinarie', 'Ore di Straordinario', 'Ore Mancanti', 'Malattia', 'Ferie', 'Assenze', 'Permesso', 'Ritardi', 'Totale Ore Lavorate'],
                    [
                        regularHours.toFixed(1),
                        overtimeHours.toFixed(1),
                        missingHours.toFixed(1),
                        sicknessDays,
                        holidaysDays,
                        absenceDays,
                        permessoDays,
                        latesCount,
                        totalHours.toFixed(1)
                    ]
                ];

                doc.autoTable({
                    startY: finalY + 30,
                    head: [summaryData[0]],
                    body: [summaryData[1]],
                    theme: 'grid',
                    styles: { fontSize: 8, cellPadding: 5, halign: 'center' }
                });


                // Save the PDF file
                doc.save(`Cartellino-Presenze-${MONTH_NAMES[pdfMonth]}-${pdfYear}.pdf`);

                pdfSelectionModal.classList.add('hidden');
            }

            function exportData() {
                const dataToExport = {};
                try {
                    for (let i = 0; i < localStorage.length; i++) {
                        const key = localStorage.key(i);
                        if (key.startsWith(storageKeyPrefix)) {
                            dataToExport[key] = localStorage.getItem(key);
                        }
                    }
                    const dataStr = JSON.stringify(dataToExport);
                    const blob = new Blob([dataStr], {type: 'application/json'});
                    const url = URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url;
                    a.download = 'timecard-data.json';
                    document.body.appendChild(a);
                    a.click();
                    document.body.removeChild(a);
                    URL.revokeObjectURL(url);
                    showMessage('Dati esportati con successo!');
                } catch (error) {
                    showMessage('Esportazione fallita. Assicurati che il browser supporti l\'esportazione di file localmente.');
                }
            }

            function importData(event) {
                const file = event.target.files[0];
                if (!file) {
                    return;
                }

                const reader = new FileReader();
                reader.onload = (e) => {
                    try {
                        const importedData = JSON.parse(e.target.result);
                        for (const key in importedData) {
                            if (key.startsWith(storageKeyPrefix)) {
                                localStorage.setItem(key, importedData[key]);
                            }
                        }
                        loadDataAndRender();
                        showMessage('Dati importati con successo!');
                    } catch (error) {
                        showMessage('Importazione dati fallita. Assicurati che il file sia un JSON valido.');
                    }
                };
                reader.readAsText(file);
            }
            
            initApp();
        };
    </script>
</body>
</html>
