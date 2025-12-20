<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Sync Reminder</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline'; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com; font-src 'self' data:;">
    <script type="text/javascript" src="cordova.js"></script>
    
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    
    <style>
        /* ==================== 
           CSS: STYLING DAN RESPONSIVITAS 
           ==================== */
        :root {
            --color-white: #ffffff;
            --color-light-bg: #f7f9ff;
            --color-primary-blue: #4A90E2;
            --color-accent-pink: #FF69B4;
            --color-text-dark: #333333;
            --color-warning: #FFC0CB;
            --color-alert: #DC3545;
            --color-syncing: #FFA500;
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            min-height: 100vh;
            margin: 0; 
            padding: 20px 10px;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            transition: background-color 0.2s; 
            color: var(--color-text-dark);
        }

        .main-container {
            background-color: var(--color-white);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center;
            width: 100%;
            max-width: 850px;
            box-sizing: border-box; 
        }

        h1 {
            color: var(--color-primary-blue);
            font-weight: 700;
            margin-bottom: 5px;
        }

        p {
            color: #666;
            margin-bottom: 25px;
            font-size: 0.9em;
        }

        .timer-list {
            display: grid;
            grid-template-columns: repeat(2, 1fr); 
            gap: 20px;
            margin-top: 20px;
        }

        .timer-card {
            border: 1px solid #e0e0e0;
            padding: 20px;
            border-radius: 12px;
            background-color: var(--color-white);
            transition: all 0.3s;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
        }

        /* Style untuk membedakan timer yang sedang berjalan */
        .timer-card.running-mode {
            border-left: 5px solid var(--color-primary-blue); 
            background-color: #f0f7ff; 
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1); 
        }

        .timer-card.running-mode h2 {
            color: var(--color-primary-blue); 
        }
        
        /* Mengubah warna countdown saat mode berjalan menjadi pink */
        .timer-card.running-mode .countdown-display {
            color: var(--color-accent-pink); 
            animation: none !important; 
        }


        .timer-card h2 {
            color: var(--color-accent-pink);
            padding-bottom: 5px;
            font-size: 1.3em;
            font-weight: 600;
        }

        .countdown-display {
            font-size: 2.5em;
            margin: 15px 0 5px 0; 
            font-weight: 700;
            color: var(--color-primary-blue);
        }
        
        /* STYLE UNTUK WAKTU SELESAI */
        .end-time-display {
            font-size: 0.85em;
            color: #999;
            margin-bottom: 10px;
        }

        .alarm-message {
            color: var(--color-alert);
            font-weight: 600;
            margin-top: 10px;
            font-size: 0.95em;
        }

        .timer-controls {
            display: flex;
            gap: 10px;
            margin-top: 15px;
            align-items: center;
            justify-content: center;
        }

        .timer-controls label {
            font-weight: 400;
            white-space: nowrap;
        }
        
        .timer-controls input {
            padding: 10px 5px;
            max-width: 60px; 
            text-align: center; 
            border: 1px solid #ddd;
            border-radius: 8px;
            font-family: 'Poppins', sans-serif;
            font-size: 1em;
            transition: border-color 0.2s;
        }
        
        .timer-controls input:focus {
            border-color: var(--color-primary-blue);
            outline: none;
        }

        .timer-controls button {
            padding: 10px 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            flex-grow: 1; 
            transition: background-color 0.2s, transform 0.1s;
        }
        .timer-controls button:active { transform: scale(0.98); }

        .start-btn.syncing {
            background-color: var(--color-syncing) !important;
            color: white;
            animation: pulse-sync 1s infinite alternate;
        }
        
        .start-btn { 
            background-color: var(--color-primary-blue); 
            color: white; 
        }
        .start-btn:hover:not(:disabled):not(.syncing) { background-color: #387ad1; }
        
        .reset-btn { 
            background-color: #6c757d; 
            color: white; 
        }
        .reset-btn:hover:not(:disabled) { background-color: #5a6268; }

        /* Style untuk Tombol STOP/DISMISS saat WAKTU HABIS */
        .stop-alarm-btn {
            background-color: var(--color-alert) !important; 
            color: white;
            font-size: 1.1em;
            padding: 10px 15px;
            animation: pulse-stop 0.5s infinite alternate; 
        }

        .timer-controls button:disabled {
            background-color: #999;
            cursor: not-allowed;
            color: #ccc;
        }

        /* --- STATE ALERT CSS --- */
        .timer-card.warning {
            border-color: var(--color-warning);
            box-shadow: 0 0 10px rgba(255, 192, 203, 0.5);
            background-color: #fffafa;
        }
        .timer-card.warning .countdown-display {
            color: var(--color-primary-blue); 
            animation: pulse-warn 1s infinite alternate; 
        }

        .timer-card.alert {
            border-color: var(--color-alert);
            background-color: #ffeff3;
            box-shadow: 0 0 10px rgba(255, 105, 180, 0.7);
        }
        .timer-card.alert .countdown-display {
            color: var(--color-alert); 
            font-size: 2.2em;
            animation: none;
        }
        
        .flash-alarm-red {
            background-color: #ff4d6d !important;
            transition: background-color 0.2s; 
        }

        /* ANIMASI */
        @keyframes pulse-warn {
            from { transform: scale(1); opacity: 1; }
            to { transform: scale(1.01); opacity: 0.9; }
        }
        @keyframes pulse-sync {
            from { background-color: var(--color-syncing); }
            to { background-color: #ff7f00; }
        }
        @keyframes pulse-stop {
            from { opacity: 1; box-shadow: 0 0 10px rgba(220, 53, 69, 0.7); }
            to { opacity: 0.8; box-shadow: none; }
        }

        /* MEDIA QUERY */
        @media (max-width: 650px) {
            body { padding: 10px 0; }
            .main-container { padding: 15px; border-radius: 0; box-shadow: none; max-width: 100%;}
            .timer-list { grid-template-columns: 1fr; gap: 15px; } 
            .timer-card { padding: 15px; }
            .countdown-display { font-size: 2em; margin: 10px 0 5px 0; }
            .timer-controls { flex-wrap: wrap; gap: 8px; justify-content: space-between; }
            .timer-controls label { flex-basis: 100%; text-align: left; font-size: 0.9em; }
            .timer-controls input { max-width: 80px; flex-grow: 0; }
            .timer-controls button { padding: 10px; font-size: 0.9em; flex-basis: calc(50% - 5px); }
            .stop-alarm-btn { flex-basis: 100%; } 
        }
    </style>
</head>
<body>
    <div class="main-container">
        <h1>Gacoan Timer Thawing 🧊</h1>
        <p>Malang Jakarta V.1.3 (Alarm Debounced)</p>
        
        <div class="timer-list" id="timer-list">
        </div>
    </div>

    <script>
       // ===================================
        // FIREBASE CONFIGURATION (JANGAN DIUBAH)
        // ===================================
        const firebaseConfig = {
          apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
          authDomain: "thawing-ec583.firebaseapp.com",
          databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
          projectId: "thawing-ec583",
          storageBucket: "thawing-ec583.firebasestorage.app", 
          messagingSenderId: "1043079332713",
          appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8",
          measurementId: "G-WLBFJ6V7QT"            
        };


        // Initialize Firebase
        try {
            if (!firebase.apps.length) {
                 firebase.initializeApp(firebaseConfig);
            }
        } catch (e) {
            console.error("Firebase Initialization Failed. Check your config.", e);
        }
        
        const dbRef = firebase.database().ref('thawingTimers');
        
        /* ==================== 
           JAVASCRIPT LOGIC 
           ==================== */
        
        // --- KONFIGURASI APLIKASI ---
        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 120 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
            { id: 7, name: "KULIT PANGSIT", defaultTimeMinutes: 120 },
        ];
        
        const WARNING_TIME_SECONDS = 15 * 60; 
        let activeIntervals = {}; 
        let notificationPermission = Notification.permission;
        
        let titleInterval = null;
        let flashInterval = null;
        const originalTitle = document.title;
        const WARNING_TITLE_PREFIX = '🚨 HABIS! - ';
        const FLASH_COLOR_CLASS = 'flash-alarm-red'; 

        // --- BARU: STATUS GLOBAL UNTUK MENGHINDARI TABRAKAN SUARA ---
        let isSpeaking = false; 
        const speechQueue = []; 
        
        // Fungsi Format Waktu
        function formatTime(totalSeconds) {
            totalSeconds = Math.max(0, totalSeconds); 
            const hours = Math.floor(totalSeconds / 3600);
            const minutes = Math.floor((totalSeconds % 3600) / 60);
            const seconds = totalSeconds % 60;

            const h = String(hours).padStart(2, '0');
            const m = String(minutes).padStart(2, '0');
            const s = String(seconds).padStart(2, '0');

            return `${h}:${m}:${s}`;
        }
        
        // =================================================
        // 🚨 FUNGSI ALARM AGRESIF DAN DEBOUNCING AUDIO
        // =================================================
        
        function resumeAudioContext() {
             if (!window.audioCtx) {
                 window.audioCtx = new (window.AudioContext || window.webkitAudioContext)();
             }
             if (window.audioCtx.state === 'suspended') {
                 window.audioCtx.resume().catch(e => console.error("Failed to resume AudioContext:", e));
             }
        }
        
        // Fungsi untuk memproses pesan suara di antrian
        function processQueue() {
            if (isSpeaking || speechQueue.length === 0) {
                return; 
            }

            isSpeaking = true;
            const message = speechQueue.shift(); // Ambil pesan pertama dari antrian

            window.speechSynthesis.cancel(); 
            
            const utterance = new SpeechSynthesisUtterance(message);
            
            const voices = window.speechSynthesis.getVoices();
            const indoVoice = voices.find(v => v.lang.startsWith('id'));
            
            if (indoVoice) {
                utterance.voice = indoVoice;
            } else {
                utterance.lang = 'id-ID';
            }
            
            utterance.rate = 1.0; 
            utterance.volume = 1.0; 
            
            utterance.onend = () => {
                isSpeaking = false; 
                processQueue(); // Setelah selesai, panggil lagi untuk memproses pesan berikutnya
            };
            utterance.onerror = () => {
                isSpeaking = false; 
                processQueue(); // Jika gagal, tetap coba proses antrian berikutnya
            };
            
            window.speechSynthesis.speak(utterance);
        }

        // Fungsi yang menambahkan pesan ke antrian suara (PENGGANTI speakMessage lama)
        function speakMessage(message) {
            if ('speechSynthesis' in window) {
                // Hindari duplikasi pesan di antrian (opsional, tapi disarankan)
                if (!speechQueue.includes(message)) {
                    speechQueue.push(message); 
                }
                processQueue(); 
            } else {
                console.warn("Web Speech API tidak didukung.");
            }
        }

        function sendNotification(itemName) {
            if (notificationPermission === 'granted') {
                new Notification("⏰ WAKTU THAWING HABIS!", {
                    body: `🚨 Segera ambil bahan: ${itemName}.`,
                    tag: 'thawing-alarm',
                    renotify: true, 
                    requireInteraction: true 
                }).onclick = function() {
                    window.focus(); 
                    this.close();
                    stopAggressiveAlarm(); 
                };
            }
        }

        function startVibrationAlert() {
            if ('vibrate' in navigator) {
                const pattern = [1000, 500, 1000]; 
                navigator.vibrate(pattern);
            }
        }

        function startFlashAlarm() {
            if (flashInterval) return;
            let isFlashing = false;
            flashInterval = setInterval(() => {
                isFlashing = !isFlashing;
                document.body.classList.toggle(FLASH_COLOR_CLASS, isFlashing);
            }, 200);
        }

        function startTitleAlert(itemName) {
            if (titleInterval) return;
            let isAlertState = false;
            titleInterval = setInterval(() => {
                isAlertState = !isAlertState;
                document.title = isAlertState 
                    ? WARNING_TITLE_PREFIX + itemName 
                    : originalTitle;
            }, 800);
        }

        // FUNGSI UNTUK MEMATIKAN SEMUA ALARM AGRESIF (Dipanggil oleh Tombol STOP)
        function stopAggressiveAlarm() {
            if (titleInterval) {
                clearInterval(titleInterval);
                titleInterval = null;
            }
            if (flashInterval) {
                clearInterval(flashInterval);
                flashInterval = null;
            }
            document.title = originalTitle;
            document.body.classList.remove(FLASH_COLOR_CLASS);
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel(); 
            }
            if ('vibrate' in navigator) {
                 navigator.vibrate(0); 
            }
            // --- Reset Status Audio dan Antrian ---
            isSpeaking = false; 
            speechQueue.length = 0; 
            // -------------------------------------
        }
        
        // --- LOGIKA UTAMA: FUNGSI TICK (Update tampilan berdasarkan data Firebase) ---
        
        function tick(itemId, endTimeMs, inputMinutes, timerState) {
            const timerCard = document.getElementById(`card-${itemId}`);
            if (!timerCard) return;

            // Tambahkan class 'running-mode' jika belum ada (sinkronisasi)
            if (!timerCard.classList.contains('running-mode')) {
                timerCard.classList.add('running-mode');
            }

            const display = document.getElementById(`display-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const endTimeDisplay = document.getElementById(`end-time-${itemId}`);
            const item = THAWING_ITEMS.find(i => i.id === itemId);
            const itemName = item ? item.name : 'Bahan';
            
            clearTimeout(activeIntervals[itemId]);

            const now = Date.now();
            let duration = Math.floor((endTimeMs - now) / 1000); 
            
            // --- UPDATE TAMPILAN KONTROL (Mode Running) ---
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            if (inputTime) inputTime.value = inputMinutes; 
            if (inputTime) inputTime.readOnly = true;
            if (startButton) startButton.style.display = 'none';
            if (resetButton) {
                resetButton.style.display = 'block';
                // Pastikan tombol reset dalam mode normal saat Running
                resetButton.textContent = 'RESET'; 
                resetButton.classList.remove('stop-alarm-btn');
            }
            
            // Tampilkan waktu selesai (End Time)
            const formattedEndTime = new Date(endTimeMs).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
            if (endTimeDisplay) endTimeDisplay.textContent = `Selesai: ${formattedEndTime}`;

            // 1. Update tampilan waktu
            if (duration >= 0) {
                 display.textContent = formatTime(duration);
            }
           
            // 2. Transisi ke mode WARNING (15 Menit)
            if (duration <= WARNING_TIME_SECONDS && duration > 0) {
                if (!timerCard.classList.contains('warning')) {
                    timerCard.classList.add('warning');
                    timerCard.classList.remove('alert'); 
                    const remainingMinutes = Math.ceil(duration / 60);
                    alarmMessage.textContent = `🔔 PERINGATAN! ${itemName} tersisa ${remainingMinutes} menit.`;
                    alarmMessage.style.display = 'block';
                }
            } else if (duration > WARNING_TIME_SECONDS) {
                 timerCard.classList.remove('warning');
                 alarmMessage.style.display = 'none';
            }

            // 3. Bunyikan alarm di mode WARNING (setiap 30 detik)
            if (duration <= WARNING_TIME_SECONDS && duration > 0 && duration % 30 === 0) {
                const remainingMinutes = Math.ceil(duration / 60);
                // Menggunakan fungsi speakMessage baru yang mendukung antrian
                speakMessage(`Perhatian! Waktu thawing ${itemName} kurang ${remainingMinutes} menit.`); 
            }
            
            // 4. Kondisi Waktu Habis: (Alarm & Hapus dari database)
            if (duration <= 0) {
                clearTimeout(activeIntervals[itemId]);
                delete activeIntervals[itemId];
                
                // Hapus entry dari Firebase (PENTING untuk sinkronisasi RESET)
                if (timerState) { 
                    dbRef.child(itemId).remove().catch(e => console.log('Hapus item gagal.'));
                }
                
                display.textContent = "WAKTU HABIS!";
                timerCard.classList.remove('warning');
                timerCard.classList.add('alert'); 
                alarmMessage.textContent = `✅ SELESAI! Bahan ${itemName} butuh penanganan.`;
                alarmMessage.style.display = 'block';
                
                // Tombol berubah menjadi STOP ALARM
                if (resetButton) {
                    resetButton.textContent = 'STOP ALARM & AMBIL'; 
                    resetButton.classList.add('stop-alarm-btn'); 
                    resetButton.style.display = 'block'; 
                }
                
                // --- AKTIVASI ALARM AGRESIF (Lokal di perangkat ini) ---
                sendNotification(itemName);
                startVibrationAlert(); 
                startTitleAlert(itemName);
                startFlashAlarm();
                // Menggunakan fungsi speakMessage baru yang mendukung antrian
                speakMessage(`PERINGATAN KERAS! Waktu thawing ${itemName} telah habis! Segera ambil bahan!`); 
                
                return; 
            }
            
            // 5. Jadwalkan tick berikutnya (Lokal)
            activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs, inputMinutes, timerState), 1000);
        }

        // --- FUNGSI PUBLIK (Dipanggil oleh User) ---
        
        function startCountdown(itemId) {
            resumeAudioContext(); 
            Notification.requestPermission().then(permission => {
                notificationPermission = permission; 
            });
            
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const durationMinutes = parseInt(inputTime.value);

            if (isNaN(durationMinutes) || durationMinutes <= 0) {
                alert(`Mohon masukkan waktu thawing yang valid.`);
                return;
            }
            
            // Tambahkan class 'running-mode' saat memulai
            const timerCard = document.getElementById(`card-${itemId}`);
            if (timerCard) timerCard.classList.add('running-mode');

            // Visual Feedback Instan
            startButton.textContent = "SYNCING...";
            startButton.classList.add('syncing');
            startButton.disabled = true; 
            
            const durationMs = durationMinutes * 60 * 1000;
            const endTimeMs = Date.now() + durationMs; 
            
            dbRef.child(itemId).set({ 
                endTime: endTimeMs, 
                inputMinutes: durationMinutes 
            })
            .then(() => {
                console.log(`Timer ${itemId} started and synced.`);
            })
            .catch(error => {
                alert("Gagal memulai timer. Periksa koneksi atau aturan Firebase.");
                console.error(error);
                localResetUI(itemId, durationMinutes); 
            });
        }

        // FUNGSI INI MENGHAPUS DATA DARI FIREBASE / MENGHENTIKAN ALARM
        function resetTimer(itemId) {
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            // SEMBUNYIKAN TOMBOL SEGERA SAAT DIKLIK
            if (resetButton) {
                resetButton.style.display = 'none'; 
            }
            
            stopAggressiveAlarm(); 
            
            // Panggil .remove() untuk memastikan sinkronisasi ke semua perangkat
            dbRef.child(itemId).remove()
            .then(() => {
                console.log(`Timer ${itemId} reset/stopped and synced.`);
            })
            .catch(error => {
                alert("Gagal mereset timer. Periksa koneksi atau aturan Firebase.");
                console.error(error);
                // Jika gagal, tampilkan kembali tombol agar user bisa coba lagi
                if (resetButton) {
                    resetButton.style.display = 'block'; 
                }
            });
        }

        // Mereset UI lokal berdasarkan item default
        function localResetUI(itemId, inputMinutes = null) {
            const item = THAWING_ITEMS.find(i => i.id === itemId);
            if (!item) return;

            const finalInput = inputMinutes !== null ? inputMinutes : item.defaultTimeMinutes;
            
            clearTimeout(activeIntervals[itemId]);
            delete activeIntervals[itemId];

            const timerCard = document.getElementById(`card-${itemId}`);
            const inputTime = document.getElementById(`time-input-${itemId}`);
            const display = document.getElementById(`display-${itemId}`);
            const endTimeDisplay = document.getElementById(`end-time-${itemId}`);
            const alarmMessage = document.getElementById(`msg-${itemId}`);
            const startButton = document.getElementById(`start-btn-${itemId}`);
            const resetButton = document.getElementById(`reset-btn-${itemId}`);
            
            if (!timerCard || !inputTime || !display || !startButton || !resetButton) return; 

            // Hapus semua status dan kembalikan ke tampilan default
            timerCard.classList.remove('alert', 'warning');
            // Hapus class 'running-mode'
            timerCard.classList.remove('running-mode');

            startButton.classList.remove('syncing'); 
            inputTime.readOnly = false;
            startButton.style.display = 'block';
            startButton.textContent = 'START';
            startButton.disabled = false;
            
            // Reset Tombol RESET/STOP
            resetButton.style.display = 'none';
            resetButton.textContent = 'RESET'; 
            resetButton.classList.remove('stop-alarm-btn'); 

            alarmMessage.style.display = 'none';

            inputTime.value = finalInput;
            display.textContent = formatTime(finalInput * 60);
            if (endTimeDisplay) endTimeDisplay.textContent = 'Durasi default';

            // Fokus pada input
            if (!timerCard.classList.contains('alert')) {
                setTimeout(() => {
                    inputTime.focus();
                }, 100); 
            }
        }

        // Fungsi untuk membuat elemen HTML timer
        function createTimerCard(item) {
            const timerListContainer = document.getElementById('timer-list');
            
            const card = document.createElement('div');
            card.className = 'timer-card';
            card.id = `card-${item.id}`;
            
            card.innerHTML = `
                <h2>${item.name}</h2>
                <div id="display-${item.id}" class="countdown-display">
                    ${formatTime(item.defaultTimeMinutes * 60)}
                </div>
                
                <div id="end-time-${item.id}" class="end-time-display">Durasi default</div> 

                <div id="msg-${item.id}" class="alarm-message" style="display: none;"></div>

                <div class="timer-controls">
                    <label for="time-input-${item.id}">Durasi (mnt):</label>
                    <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}" min="1" max="180">
                    <button id="start-btn-${item.id}" class="start-btn">START</button>
                    <button id="reset-btn-${item.id}" class="reset-btn" style="display: none;">RESET</button>
                </div>
            `;
            
            timerListContainer.appendChild(card);
            
            document.getElementById(`start-btn-${item.id}`).addEventListener('click', () => startCountdown(item.id));
            document.getElementById(`reset-btn-${item.id}`).addEventListener('click', () => resetTimer(item.id));

            document.getElementById(`time-input-${item.id}`).addEventListener('input', (event) => {
                const minutes = parseInt(event.target.value) || 0;
                const display = document.getElementById(`display-${item.id}`);
                const endTimeDisplay = document.getElementById(`end-time-${item.id}`);
                
                if (!document.getElementById(`time-input-${item.id}`).readOnly) {
                    display.textContent = formatTime(minutes * 60);
                    if (endTimeDisplay) endTimeDisplay.textContent = 'Durasi custom';
                }
            });
            
            localResetUI(item.id, item.defaultTimeMinutes);
        }

        // ===================================
        // 🚨 LOGIKA SINKRONISASI REAL-TIME
        // ===================================
        
        document.addEventListener('DOMContentLoaded', () => {
            THAWING_ITEMS.forEach(item => {
                createTimerCard(item);
            });
            notificationPermission = Notification.permission;
            
            dbRef.on('value', (snapshot) => {
                const timersData = snapshot.val() || {}; 

                THAWING_ITEMS.forEach(item => {
                    const itemId = item.id;
                    const timerState = timersData[itemId];
                    
                    if (timerState && timerState.endTime) {
                        const endTime = timerState.endTime;
                        const inputMinutes = timerState.inputMinutes || item.defaultTimeMinutes;
                        
                        tick(itemId, endTime, inputMinutes, timerState);
                        
                        const startButton = document.getElementById(`start-btn-${itemId}`);
                         if (startButton) {
                             startButton.classList.remove('syncing');
                         }

                    } else {
                        clearTimeout(activeIntervals[itemId]);
                        delete activeIntervals[itemId];

                        localResetUI(itemId, item.defaultTimeMinutes); 
                    }
                });
            });
        });
        
        // PWA SERVICE WORKER REGISTRATION
        if ('serviceWorker' in navigator) {
          window.addEventListener('load', () => {
            navigator.serviceWorker.register('/service-worker.js')
              .then(registration => console.log('ServiceWorker registered'))
              .catch(error => console.error('ServiceWorker registration failed:', error));
          });
        }
    </script>
</body>
</html>
