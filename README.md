
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HydroSmart iOS 26 - Irigasi IoT</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    
    <!-- Google Fonts: Inter for clean iOS look -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@200;300;400;500;600;700&display=swap" rel="stylesheet">

    <style>
        /* --- CORE THEME SETUP --- */
        body {
            font-family: 'Inter', sans-serif;
            /* background: #000; <- Dihapus, background kini dikontrol sepenuhnya oleh ambient-bg/JS */
            overflow-x: hidden;
            color: white; /* Warna teks default, akan ditimpa JS untuk tema terang */
            min-height: 100vh;
            transition: color 1.5s ease; /* Transisi warna teks yang halus */
        }

        /* --- DYNAMIC BACKGROUND: Gradient akan diatur oleh JavaScript --- */
        .ambient-bg {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
            overflow: hidden;
            /* Tambahkan transisi untuk efek smooth saat warna ganti */
            transition: background 1.5s ease;
        }

        .orb {
            position: absolute;
            border-radius: 50%;
            filter: blur(80px);
            opacity: 0.6;
            animation: float 10s infinite ease-in-out alternate;
            /* Tambahkan transisi untuk orbs */
            transition: background 1.5s ease;
        }

        /* Definisikan ukuran dan posisi Orb, warnanya diatur JS */
        .orb-1 { width: 400px; height: 400px; top: -50px; left: -50px; animation-duration: 12s; }
        .orb-2 { width: 500px; height: 500px; bottom: -100px; right: -100px; animation-duration: 15s; }
        .orb-3 { width: 300px; height: 300px; top: 40%; left: 40%; animation-duration: 18s; animation-delay: -5s; }

        @keyframes float {
            0% { transform: translate(0, 0) scale(1); }
            100% { transform: translate(30px, 50px) scale(1.1); }
        }

        /* --- LIQUID GLASS iOS 26 CARD STYLE --- */
        .glass-panel {
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(40px) saturate(180%);
            -webkit-backdrop-filter: blur(40px) saturate(180%);
            border: 1px solid rgba(255, 255, 255, 0.15);
            border-radius: 32px;
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        
        /* Tambahan: Penyesuaian Glass Panel untuk tema terang agar teks gelap di dalamnya tetap terbaca */
        body.light-theme .glass-panel {
            background: rgba(0, 0, 0, 0.05); /* Sedikit lebih gelap agar kontras */
            border: 1px solid rgba(0, 0, 0, 0.1);
        }

        .glass-panel:hover {
            border: 1px solid rgba(255, 255, 255, 0.25);
            box-shadow: 0 12px 40px 0 rgba(0, 0, 0, 0.4);
        }

        /* --- LIQUID TOGGLE & BUTTONS --- */
        .liquid-btn {
            background: linear-gradient(135deg, rgba(255,255,255,0.1), rgba(255,255,255,0.05));
            border: 1px solid rgba(255,255,255,0.2);
            backdrop-filter: blur(10px);
            position: relative;
            overflow: hidden;
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        
        body.light-theme .liquid-btn {
            background: linear-gradient(135deg, rgba(0,0,0,0.1), rgba(0,0,0,0.05));
            border: 1px solid rgba(0,0,0,0.2);
        }

        .liquid-btn::before {
            content: '';
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
            transform: translateX(-100%);
            transition: 0.5s;
        }

        .liquid-btn:hover::before {
            transform: translateX(100%);
        }

        .liquid-btn:active {
            transform: scale(0.95);
        }

        .liquid-btn.active {
            background: linear-gradient(135deg, #3b82f6, #06b6d4);
            border-color: rgba(255,255,255,0.4);
            box-shadow: 0 0 20px rgba(59, 130, 246, 0.5);
        }

        /* --- WAVE ANIMATION FOR WATER LEVEL --- */
        .wave-container {
            position: relative;
            overflow: hidden;
            border-radius: 50%;
            /* mask-image for safari border radius fix */
            -webkit-mask-image: -webkit-radial-gradient(white, black);
        }

        .wave {
            position: absolute;
            left: 0;
            width: 200%;
            height: 200%;
            background: linear-gradient(180deg, rgba(59, 130, 246, 0.6), rgba(6, 182, 212, 0.8));
            border-radius: 40%;
            animation: wave-spin 6s infinite linear;
            transform-origin: 50% 48%;
        }

        .wave:nth-child(2) {
            background: linear-gradient(180deg, rgba(59, 130, 246, 0.6), rgba(6, 182, 212, 0.8));
            opacity: 0.5;
            animation: wave-spin 10s infinite linear;
            border-radius: 45%;
        }

        @keyframes wave-spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.2); border-radius: 10px; }
        
        body.light-theme ::-webkit-scrollbar-thumb { background: rgba(0,0,0,0.2); }

        /* Chart Canvas Glow */
        #moistureChart {
            filter: drop-shadow(0 0 10px rgba(59, 130, 246, 0.3));
        }
    </style>
</head>
<body class="p-4 md:p-8 flex justify-center items-center">

    <!-- Ambient Background -->
    <div class="ambient-bg">
        <div class="orb orb-1"></div>
        <div class="orb orb-2"></div>
        <div class="orb orb-3"></div>
    </div>

    <!-- Main App Container -->
    <main class="w-full max-w-[1200px] grid grid-cols-1 lg:grid-cols-12 gap-6 relative z-10">
        
        <!-- Header & Profile (Top Bar) -->
        <header class="col-span-1 lg:col-span-12 glass-panel p-5 flex justify-between items-center animate-fade-in-down">
            <div class="flex items-center gap-4">
                <div class="w-12 h-12 rounded-2xl bg-gradient-to-br from-blue-500 to-cyan-400 flex items-center justify-center shadow-lg shadow-blue-500/20">
                    <i data-lucide="droplets" class="text-white w-6 h-6"></i>
                </div>
                <div>
                    <h1 class="text-2xl font-light tracking-wide text-white">Hydro<span class="font-bold">Smart</span></h1>
                    <p class="text-xs text-white/50 uppercase tracking-widest">Sistem Irigasi Cerdas v2.6</p>
                </div>
            </div>
            
            <div class="flex items-center gap-4">
                <div class="hidden md:flex flex-col items-end mr-2">
                    <span class="text-sm font-medium">Status Koneksi</span>
                    <div class="flex items-center gap-2">
                        <!-- Indikator Status (Akan berubah via JS) -->
                        <span id="connDot" class="w-2 h-2 rounded-full bg-yellow-400 animate-pulse"></span>
                        <span id="connText" class="text-xs text-yellow-400">Menghubungkan...</span>
                    </div>
                </div>
                <button class="w-10 h-10 rounded-full glass-panel flex items-center justify-center border border-white/20 hover:bg-white/10 transition">
                    <i data-lucide="bell" class="w-5 h-5 text-white/80"></i>
                </button>
                <div class="w-10 h-10 rounded-full bg-gradient-to-tr from-purple-500 to-pink-500 p-[2px]">
                    <img src="https://api.dicebear.com/9.x/avataaars/svg?seed=Felix" alt="User" class="rounded-full bg-black/50 w-full h-full">
                </div>
            </div>
        </header>

        <!-- Left Sidebar / Controls -->
        <aside class="col-span-1 lg:col-span-4 flex flex-col gap-6">
            
            <!-- Weather / Time Card -->
            <div class="glass-panel p-6 flex flex-col justify-between h-48 relative overflow-hidden group">
                <div class="absolute top-0 right-0 w-32 h-32 bg-yellow-400/20 rounded-full blur-3xl -mr-10 -mt-10"></div>
                
                <div class="flex justify-between items-start z-10">
                    <div>
                        <h2 class="text-4xl font-thin" id="clock">10:45</h2>
                        <p class="text-white/60 text-sm mt-1" id="date">Senin, 13 Des</p>
                    </div>
                    <!-- Lucide icon for sun/moon will be updated by JS -->
                    <i data-lucide="sun" class="w-10 h-10 text-yellow-300" id="timeIcon"></i>
                </div>
                
                <div class="mt-auto z-10">
                    <div class="flex items-center gap-2 mb-1">
                        <i data-lucide="map-pin" class="w-3 h-3 text-white/50"></i>
                        <span class="text-xs text-white/50">Kebun Utama, Malang</span>
                    </div>
                    <p class="text-lg font-medium">Cerah Berawan</p>
                    <p class="text-sm text-white/60">Estimasi hujan: 10%</p>
                </div>
            </div>

            <!-- Master Control Panel -->
            <div class="glass-panel p-6 flex-1 min-h-[300px]">
                <h3 class="text-lg font-medium mb-6 flex items-center gap-2">
                    <i data-lucide="sliders" class="w-5 h-5 text-blue-400"></i> Kontrol Manual
                </h3>

                <!-- Switch 1: Pump -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <span class="text-white/80 text-sm">Pompa Air Utama</span>
                        <span id="pumpStatusText" class="text-xs text-white/50">MATI</span>
                    </div>
                    <button id="pumpBtn" onclick="togglePump()" class="liquid-btn w-full h-14 rounded-2xl flex items-center justify-between px-4 group">
                        <div class="flex items-center gap-3">
                            <div class="p-2 rounded-xl bg-white/10 group-hover:bg-white/20 transition">
                                <i data-lucide="zap" class="w-5 h-5 text-white"></i>
                            </div>
                            <span class="font-medium tracking-wide">Daya Pompa</span>
                        </div>
                        <div class="w-10 h-5 rounded-full bg-black/40 relative border border-white/10">
                            <div id="pumpToggle" class="absolute left-1 top-1 w-3 h-3 rounded-full bg-white/50 transition-all duration-300"></div>
                        </div>
                    </button>
                </div>

                <!-- Switch 2: Auto Mode -->
                <div class="mb-6">
                    <div class="flex justify-between items-center mb-2">
                        <span class="text-white/80 text-sm">Mode Otomatis (AI)</span>
                        <span id="autoStatusText" class="text-xs text-white/50">AKTIF</span>
                    </div>
                    <button id="autoBtn" onclick="toggleAuto()" class="liquid-btn w-full h-14 rounded-2xl flex items-center justify-between px-4 group active">
                        <div class="flex items-center gap-3">
                            <div class="p-2 rounded-xl bg-white/10 group-hover:bg-white/20 transition">
                                <i data-lucide="cpu" class="w-5 h-5 text-white"></i>
                            </div>
                            <span class="font-medium tracking-wide">Smart Sense</span>
                        </div>
                        <div class="w-10 h-5 rounded-full bg-black/40 relative border border-white/10">
                            <div id="autoToggle" class="absolute left-6 top-1 w-3 h-3 rounded-full bg-white shadow-[0_0_10px_white] transition-all duration-300"></div>
                        </div>
                    </button>
                </div>
                
                <!-- Intensity Slider -->
                <div>
                    <div class="flex justify-between items-center mb-3">
                        <span class="text-white/80 text-sm">Target Kelembaban</span>
                        <span id="targetValue">65%</span>
                    </div>
                    <input type="range" min="0" max="100" value="65" class="w-full h-2 bg-white/10 rounded-lg appearance-none cursor-pointer accent-blue-500 hover:accent-blue-400 transition" oninput="document.getElementById('targetValue').innerText = this.value + '%'">
                </div>
                
                <!-- Input IP ESP (Hidden/Advanced) -->
                <div class="mt-4 pt-4 border-t border-white/10">
                     <p class="text-[10px] text-white/30 mb-1">ALAMAT IP ESP-01</p>
                     <input type="text" id="espIP" value="192.168.4.1" class="w-full bg-black/20 text-xs text-white/70 p-2 rounded border border-white/5 focus:border-blue-500/50 outline-none" placeholder="Masukkan IP ESP">
                </div>

            </div>
        </aside>

        <!-- Main Dashboard Area -->
        <section class="col-span-1 lg:col-span-8 grid grid-cols-1 md:grid-cols-2 gap-6">
            
            <!-- Sensor Data Grid -->
            <div class="glass-panel p-6 col-span-1 md:col-span-2">
                <h3 class="text-lg font-medium mb-4 flex items-center gap-2">
                    <i data-lucide="activity" class="w-5 h-5 text-cyan-400"></i> Monitoring Real-time
                </h3>
                <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                    <!-- Soil Moisture -->
                    <div class="bg-white/5 rounded-2xl p-4 border border-white/10 text-center hover:bg-white/10 transition duration-300">
                        <div class="text-xs text-white/50 mb-1">Tanah</div>
                        <div class="text-2xl font-bold text-green-400 mb-1" id="moistureVal">--%</div>
                        <div class="text-[10px] text-white/40">Kelembaban</div>
                    </div>
                    <!-- Temperature -->
                    <div class="bg-white/5 rounded-2xl p-4 border border-white/10 text-center hover:bg-white/10 transition duration-300">
                        <div class="text-xs text-white/50 mb-1">Udara</div>
                        <div class="text-2xl font-bold text-orange-300 mb-1" id="tempVal">--°C</div>
                        <div class="text-[10px] text-white/40">Suhu Area</div>
                    </div>
                    <!-- Humidity -->
                    <div class="bg-white/5 rounded-2xl p-4 border border-white/10 text-center hover:bg-white/10 transition duration-300">
                        <div class="text-xs text-white/50 mb-1">RH</div>
                        <div class="text-2xl font-bold text-blue-300 mb-1" id="humidVal">--%</div>
                        <div class="text-[10px] text-white/40">Humidity</div>
                    </div>
                    <!-- pH Level -->
                    <div class="bg-white/5 rounded-2xl p-4 border border-white/10 text-center hover:bg-white/10 transition duration-300">
                        <div class="text-xs text-white/50 mb-1">Asiditas</div>
                        <div class="text-2xl font-bold text-purple-300 mb-1" id="phVal">--</div>
                        <div class="text-[10px] text-white/40">pH Tanah</div>
                    </div>
                </div>
            </div>

            <!-- Water Tank Level (Visual) -->
            <div class="glass-panel p-6 flex flex-col items-center justify-center relative">
                <h3 class="absolute top-6 left-6 text-sm font-medium text-white/80 z-10">Tangki Air</h3>
                <div class="w-40 h-40 rounded-full border-4 border-white/10 p-1 relative shadow-2xl shadow-blue-500/10">
                    <div class="w-full h-full rounded-full wave-container bg-gray-900/50 relative">
                        <!-- The Liquid Wave -->
                        <div class="wave" id="waterWave" style="top: 30%"></div>
                        <div class="wave" style="top: 30%; animation-direction: reverse; opacity: 0.3;"></div>
                        
                        <!-- Text Overlay -->
                        <div class="absolute inset-0 flex flex-col items-center justify-center z-20">
                            <span class="text-3xl font-bold text-white drop-shadow-md" id="tankLevel">--%</span>
                            <span class="text-[10px] text-white/70 uppercase tracking-widest mt-1">Cadangan</span>
                        </div>
                    </div>
                </div>
                <div class="mt-6 w-full px-4">
                    <div class="flex justify-between text-xs text-white/40 mb-1">
                        <span>Flow Rate</span>
                        <span>2.4 L/min</span>
                    </div>
                    <div class="w-full bg-white/10 h-1.5 rounded-full overflow-hidden">
                        <div class="bg-blue-500 h-full rounded-full animate-pulse" style="width: 40%"></div>
                    </div>
                </div>
            </div>

            <!-- Chart Area -->
            <div class="glass-panel p-6 flex flex-col justify-between">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-sm font-medium text-white/80">Grafik Kelembaban (24j)</h3>
                    <select class="bg-black/20 border border-white/10 rounded-lg text-xs text-white/70 px-2 py-1 outline-none">
                        <option>Hari Ini</option>
                        <option>Minggu Ini</option>
                    </select>
                </div>
                <div class="relative h-40 w-full flex items-end">
                    <!-- Simple CSS/HTML Chart using flex bars for demo (canvas is better but this keeps it clean without huge libs) -->
                    <canvas id="moistureChart" class="w-full h-full"></canvas>
                </div>
            </div>

            <!-- Upcoming Schedules (List) -->
            <div class="glass-panel col-span-1 md:col-span-2 p-6">
                <h3 class="text-lg font-medium mb-4 flex items-center gap-2">
                    <i data-lucide="calendar-clock" class="w-5 h-5 text-purple-400"></i> Jadwal Penyiraman
                </h3>
                <div class="space-y-3">
                    <div class="flex items-center justify-between p-3 rounded-xl bg-white/5 border border-white/5 hover:bg-white/10 transition cursor-pointer">
                        <div class="flex items-center gap-4">
                            <div class="w-10 h-10 rounded-full bg-blue-500/20 flex items-center justify-center text-blue-400">
                                <i data-lucide="cloud-drizzle" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-medium text-sm">Penyiraman Pagi</h4>
                                <p class="text-xs text-white/40">Zona A & B • Durasi 15 menit</p>
                            </div>
                        </div>
                        <div class="text-right">
                            <div class="text-sm font-bold">06:00</div>
                            <div class="text-xs text-green-400">Selesai</div>
                        </div>
                    </div>

                    <div class="flex items-center justify-between p-3 rounded-xl bg-white/5 border border-white/5 hover:bg-white/10 transition cursor-pointer relative overflow-hidden">
                        <div class="absolute left-0 top-0 bottom-0 w-1 bg-orange-400"></div>
                        <div class="flex items-center gap-4">
                            <div class="w-10 h-10 rounded-full bg-orange-500/20 flex items-center justify-center text-orange-400">
                                <i data-lucide="sun" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="font-medium text-sm">Penyiraman Sore</h4>
                                <p class="text-xs text-white/40">Zona Utama • Durasi 20 menit</p>
                            </div>
                        </div>
                        <div class="text-right">
                            <div class="text-sm font-bold">16:30</div>
                            <div class="text-xs text-orange-300">Menunggu</div>
                        </div>
                    </div>
                </div>
            </div>

        </section>
    </main>

    <!-- Notification Toast (Hidden by default) -->
    <div id="toast" class="fixed bottom-8 right-8 glass-panel px-6 py-4 transform translate-y-32 opacity-0 transition-all duration-500 flex items-center gap-4 z-50">
        <div class="w-8 h-8 rounded-full bg-green-500/20 flex items-center justify-center text-green-400">
            <i data-lucide="check" class="w-4 h-4"></i>
        </div>
        <div>
            <h4 class="text-sm font-bold">Sistem Diperbarui</h4>
            <p class="text-xs text-white/60" id="toastMsg">Status pompa berubah.</p>
        </div>
    </div>

    <script>
        // Initialize Icons
        lucide.createIcons();

        // --- STATE MANAGEMENT ---
        let pumpActive = false;
        let autoMode = true;
        let tankLevel = 70;
        let isConnectedToESP = false;
        
        // --- CLOCK & THEME UPDATE ---
        function updateClock() {
            const now = new Date();
            const hours = now.getHours(); // Ambil jam saat ini
            
            const clockElement = document.getElementById('clock');
            const dateElement = document.getElementById('date');

            const hoursStr = String(now.getHours()).padStart(2, '0');
            const minutes = String(now.getMinutes()).padStart(2, '0');
            clockElement.innerText = `${hoursStr}:${minutes}`;
            
            const options = { weekday: 'long', day: 'numeric', month: 'short' };
            dateElement.innerText = now.toLocaleDateString('id-ID', options);
            
            // Panggil fungsi pembaruan tema
            updateBackgroundTheme(hours);
        }

        function updateBackgroundTheme(hours) {
            const bgElement = document.querySelector('.ambient-bg');
            const orb1 = document.querySelector('.orb-1');
            const orb2 = document.querySelector('.orb-2');
            const orb3 = document.querySelector('.orb-3');
            const body = document.body;
            const timeIconContainer = document.getElementById('timeIcon'); 

            // Cek apakah timeIconContainer sudah diubah menjadi SVG oleh Lucide
            // Kita perlu mengambil elemen SVG anak dari container (atau container itu sendiri jika sudah menjadi SVG)
            const timeIcon = timeIconContainer.querySelector('svg') || timeIconContainer;

            let gradient, orb1Color, orb2Color, orb3Color, iconName, iconColor;
            let isLightTheme = false;

            if (hours >= 5 && hours < 10) {
                // Pagi (Morning): 05:00 - 10:00 -> Biru
                isLightTheme = false;
                gradient = 'linear-gradient(125deg, #1d4ed8, #60a5fa)'; 
                orb1Color = '#3b82f6'; 
                orb2Color = '#0ea5e9'; 
                orb3Color = '#93c5fd'; 
                iconName = 'cloud-sun';
                iconColor = 'text-blue-300';
            } else if (hours >= 10 && hours < 15) {
                // Siang (Day): 10:00 - 15:00 -> Kuning Keputihan (Sangat Cerah)
                isLightTheme = true;
                gradient = 'linear-gradient(125deg, #fef9e6, #fde68a)'; // Kuning Keputihan
                orb1Color = '#fcd34d'; 
                orb2Color = '#f59e0b'; 
                orb3Color = '#fef3c7'; 
                iconName = 'sun';
                iconColor = 'text-yellow-600';
            } else if (hours >= 15 && hours < 19) {
                // Sore (Afternoon): 15:00 - 19:00 -> Orange Kekuningan
                isLightTheme = true;
                gradient = 'linear-gradient(125deg, #f97316, #fb923c)'; // Orange Kekuningan
                orb1Color = '#ef4444'; 
                orb2Color = '#f97316'; 
                orb3Color = '#facc15'; 
                iconName = 'sunset';
                iconColor = 'text-orange-600';
            } else {
                // Malam (Night): 19:00 - 05:00 -> Abu-abu sedikit gelap (SEKARANG)
                isLightTheme = false;
                // Menggunakan warna yang lebih jelas agar perbedaan dari default #000 terlihat
                gradient = 'linear-gradient(125deg, #1f2937, #374151)'; // Abu-abu gelap (Dark Slate/Gray)
                orb1Color = '#4b5563'; // Gray-600
                orb2Color = '#6b7280'; // Gray-500
                orb3Color = '#374151'; // Gray-700
                iconName = 'moon';
                iconColor = 'text-indigo-400';
            }

            // Terapkan tema ke elemen background dan orbs
            bgElement.style.background = gradient;
            orb1.style.background = orb1Color;
            orb2.style.background = orb2Color;
            orb3.style.background = orb3Color;
            
            // --- PERBAIKAN ERROR: Mengatasi masalah class SVG Lucide ---
            timeIconContainer.setAttribute('data-lucide', iconName);
            // Render ulang ikon agar Lucide mengganti SVG sesuai data-lucide baru
            lucide.createIcons(); 
            
            // Ambil lagi SVG yang baru dibuat atau sudah ada (untuk memastikan kita punya elemen SVG yang benar)
            // Karena Lucide menciptakan ulang SVG, kita harus mencari lagi SVG child-nya
            const newTimeIcon = timeIconContainer.querySelector('svg');

            if (newTimeIcon && newTimeIcon.classList) {
                // Hapus semua class warna Tailwind (text-xxx) yang lama
                Array.from(newTimeIcon.classList).forEach(c => {
                    if (c.startsWith('text-')) {
                        newTimeIcon.classList.remove(c);
                    }
                });
                // Tambahkan class warna yang baru
                newTimeIcon.classList.add(iconColor);
            }
            // --- AKHIR PERBAIKAN ERROR ---

            // Atur warna teks global dan kelas body untuk visibilitas
            if (isLightTheme) {
                body.style.color = '#333'; // Teks gelap untuk latar terang
                body.classList.add('light-theme');
                body.classList.remove('dark-theme');
            } else {
                body.style.color = 'white'; // Teks terang untuk latar gelap
                body.classList.add('dark-theme');
                body.classList.remove('light-theme');
            }
        }

        setInterval(updateClock, 1000);
        updateClock(); // Panggil saat inisialisasi

        // --- CHART LOGIC (Visual Only) ---
        const canvas = document.getElementById('moistureChart');
        const ctx = canvas.getContext('2d');
        let chartData = [65, 68, 72, 70, 65, 60, 58, 62, 70, 75, 72, 70];

        function resizeCanvas() {
            const rect = canvas.parentNode.getBoundingClientRect();
            canvas.width = rect.width * 2;
            canvas.height = rect.height * 2;
            ctx.scale(2, 2);
        }
        window.addEventListener('resize', resizeCanvas);
        setTimeout(resizeCanvas, 100);

        function drawChart() {
            if(!canvas.width) return;
            const w = canvas.width / 2;
            const h = canvas.height / 2;
            ctx.clearRect(0, 0, w, h);
            
            const gradient = ctx.createLinearGradient(0, 0, w, 0);
            gradient.addColorStop(0, '#3b82f6');
            gradient.addColorStop(1, '#06b6d4');
            
            ctx.beginPath();
            const step = w / (chartData.length - 1);
            
            chartData.forEach((val, index) => {
                const y = h - (val / 100 * h);
                const x = index * step;
                if(index === 0) ctx.moveTo(x, y);
                else {
                    const prevX = (index - 1) * step;
                    const prevY = h - (chartData[index - 1] / 100 * h);
                    const cx = (prevX + x) / 2;
                    ctx.quadraticCurveTo(cx, prevY, cx, (prevY + y) / 2);
                    ctx.lineTo(x, y);
                }
            });
            
            ctx.strokeStyle = gradient;
            ctx.lineWidth = 4;
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';
            ctx.stroke();

            ctx.lineTo(w, h);
            ctx.lineTo(0, h);
            ctx.closePath();
            const fillGrad = ctx.createLinearGradient(0, 0, 0, h);
            fillGrad.addColorStop(0, 'rgba(59, 130, 246, 0.2)');
            fillGrad.addColorStop(1, 'rgba(59, 130, 246, 0)');
            ctx.fillStyle = fillGrad;
            ctx.fill();

            chartData.forEach((val, index) => {
                if(index % 2 !== 0) return;
                const y = h - (val / 100 * h);
                const x = index * step;
                ctx.beginPath();
                ctx.arc(x, y, 4, 0, Math.PI * 2);
                ctx.fillStyle = '#fff';
                ctx.fill();
            });
        }
        
        // --- BUTTON INTERACTIONS ---
        function togglePump() {
            if(autoMode) {
                showToast("Matikan Mode Otomatis dulu untuk kontrol manual.");
                return;
            }
            // Send command to ESP if connected (Simulated fetch)
            if(isConnectedToESP) {
                const espIP = document.getElementById('espIP').value;
                const cmd = pumpActive ? 'off' : 'on';
                fetch(`http://${espIP}/pump?state=${cmd}`).catch(e => console.log("ESP cmd fail"));
            }

            pumpActive = !pumpActive;
            updatePumpUI();
        }

        function updatePumpUI() {
            const btn = document.getElementById('pumpBtn');
            const toggle = document.getElementById('pumpToggle');
            const status = document.getElementById('pumpStatusText');
            if(pumpActive) {
                btn.classList.add('active');
                toggle.style.left = '26px';
                toggle.style.backgroundColor = 'white';
                toggle.style.boxShadow = '0 0 10px white';
                status.innerText = 'MENYIRAM...';
                status.classList.add('text-blue-300');
                if(!isConnectedToESP) showToast("Pompa Air Diaktifkan (Simulasi)");
            } else {
                btn.classList.remove('active');
                toggle.style.left = '4px';
                toggle.style.backgroundColor = 'rgba(255,255,255,0.5)';
                toggle.style.boxShadow = 'none';
                status.innerText = 'MATI';
                status.classList.remove('text-blue-300');
                if(!isConnectedToESP) showToast("Pompa Air Dimatikan (Simulasi)");
            }
        }

        function toggleAuto() {
            autoMode = !autoMode;
            const btn = document.getElementById('autoBtn');
            const toggle = document.getElementById('autoToggle');
            const status = document.getElementById('autoStatusText');
            const pumpStatus = document.getElementById('pumpStatusText');

            if(autoMode) {
                btn.classList.add('active');
                toggle.style.left = '26px';
                toggle.style.backgroundColor = 'white';
                toggle.style.boxShadow = '0 0 10px white';
                status.innerText = 'AKTIF';
                status.classList.add('text-green-300');
                
                pumpActive = false;
                updatePumpUI();
                pumpStatus.innerText = 'AUTO';
                
                showToast("Mode AI Smart Sense Diaktifkan");
            } else {
                btn.classList.remove('active');
                toggle.style.left = '4px';
                toggle.style.backgroundColor = 'rgba(255,255,255,0.5)';
                toggle.style.boxShadow = 'none';
                status.innerText = 'MANUAL';
                status.classList.remove('text-green-300');
                pumpStatus.innerText = 'MATI';
                showToast("Mode beralih ke Manual");
            }
        }

        function showToast(msg) {
            const toast = document.getElementById('toast');
            document.getElementById('toastMsg').innerText = msg;
            toast.classList.remove('translate-y-32', 'opacity-0');
            setTimeout(() => {
                toast.classList.add('translate-y-32', 'opacity-0');
            }, 3000);
        }

        // --- ESP-01 INTEGRATION LOGIC ---
        
        async function fetchESPData() {
            const espIP = document.getElementById('espIP').value;
            // Endpoint yang diasumsikan: http://192.168.4.1/json
            // Format JSON yang diharapkan: {"moisture": 60, "temp": 28.5, "humidity": 70, "ph": 6.8, "tank": 80}
            
            try {
                // Timeout pendek agar UI tidak hang jika ESP mati
                const controller = new AbortController();
                const timeoutId = setTimeout(() => controller.abort(), 1000);

                const response = await fetch(`http://${espIP}/json`, { 
                    signal: controller.signal,
                    mode: 'cors' // Pastikan ESP handle CORS
                });
                
                const data = await response.json();
                
                // Jika sukses, update dengan data nyata
                updateDashboard(data.moisture, data.temp, data.humidity, data.ph, data.tank);
                setConnectionStatus(true);
                
            } catch (error) {
                // Jika gagal fetch, gunakan simulasi
                // console.log("ESP tidak terhubung, menggunakan simulasi.");
                simulateData();
                setConnectionStatus(false);
            }
        }

        function simulateData() {
            // Random fluctuations untuk demo
            const moisture = Math.floor(68 + Math.random() * 8);
            const temp = (27 + Math.random() * 2).toFixed(1);
            const humid = Math.floor(58 + Math.random() * 5);
            const ph = (6.5 + Math.random() * 0.5).toFixed(1);
            
            // Sim tank depletion
            if(pumpActive) tankLevel -= 0.5;
            if(tankLevel < 0) tankLevel = 100;

            updateDashboard(moisture, temp, humid, ph, tankLevel);
        }

        function updateDashboard(moisture, temp, humid, ph, tank) {
            document.getElementById('moistureVal').innerText = moisture + "%";
            document.getElementById('tempVal').innerText = temp + "°C";
            document.getElementById('humidVal').innerText = humid + "%";
            document.getElementById('phVal').innerText = ph;
            
            document.getElementById('tankLevel').innerText = Math.floor(tank) + "%";
            const waveTop = 100 - tank; 
            document.getElementById('waterWave').style.top = (waveTop - 10) + "%";
            
            // Update chart array
            chartData.shift();
            chartData.push(moisture);
            drawChart();
        }

        function setConnectionStatus(connected) {
            const dot = document.getElementById('connDot');
            const text = document.getElementById('connText');
            
            if(connected) {
                if(!isConnectedToESP) showToast("Terhubung ke ESP-01");
                isConnectedToESP = true;
                dot.classList.remove('bg-yellow-400', 'bg-red-500');
                dot.classList.add('bg-green-400');
                text.innerText = "Terhubung (ESP-01)";
                text.classList.remove('text-yellow-400', 'text-red-500');
                text.classList.add('text-green-400');
            } else {
                isConnectedToESP = false;
                dot.classList.remove('bg-green-400', 'bg-yellow-400');
                dot.classList.add('bg-red-500');
                text.innerText = "Mode Simulasi (Demo)";
                text.classList.remove('text-green-400', 'text-yellow-400');
                text.classList.add('text-red-500');
            }
        }

        // Loop utama setiap 2 detik
        setInterval(fetchESPData, 2000);

    </script>
</body>
</html>
