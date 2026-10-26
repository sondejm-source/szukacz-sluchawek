<!DOCTYPE html>
<html lang="pl">
<head>
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Szukacz Słuchawek SondejM</title>
    <link rel="manifest" href="manifest.json">
    <script>
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('sw.js');
      }
    </script>
</head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SondejM - Lokalizator Słuchawek</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        /* Animation: Pulse twice per second (0.5s duration) */
        @keyframes pulse2hz {
            0%, 100% {
                opacity: 1;
                transform: scale(1);
                filter: drop-shadow(0 0 14px rgba(239, 68, 68, 0.9));
            }
            50% {
                opacity: 0.35;
                transform: scale(0.95);
                filter: drop-shadow(0 0 2px rgba(239, 68, 68, 0.2));
            }
        }

        .pulsing-logo {
            animation: pulse2hz 0.5s infinite ease-in-out;
        }

        /* Radar sweep animation */
        @keyframes sweep {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .radar-sweep {
            animation: sweep 3s linear infinite;
            transform-origin: bottom right;
        }

        /* Glassmorphism card styles */
        .glass-card {
            background: rgba(17, 24, 39, 0.8);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col items-center justify-between font-sans antialiased selection:bg-red-500 selection:text-white p-4">

    <!-- Container -->
    <div class="w-full max-w-md mx-auto flex flex-col gap-6 my-auto">

        <!-- Header / Logo Section -->
        <header class="text-center py-4 flex flex-col items-center justify-center">
            <h1 class="text-4xl md:text-5xl font-extrabold tracking-wider text-red-500 uppercase select-none pulsing-logo">
                SondejM
            </h1>
            <p class="text-xs text-slate-400 mt-2 font-medium tracking-wide uppercase">
                <i class="fa-solid fa-radar text-red-500 mr-1"></i> Precyzyjny Lokalizator Słuchawek
            </p>
        </header>

        <!-- Main Search Button Card -->
        <div class="glass-card rounded-3xl p-6 shadow-2xl flex flex-col items-center text-center relative overflow-hidden">
            <div class="absolute -right-10 -top-10 w-32 h-32 bg-red-600/10 rounded-full blur-2xl pointer-events-none"></div>

            <!-- Main 'Szukaj' (Search) Trigger -->
            <button id="searchBtn" class="w-full py-5 px-6 bg-gradient-to-r from-red-600 to-rose-700 hover:from-red-500 hover:to-rose-600 active:scale-95 transition-all duration-200 rounded-2xl font-bold text-xl text-white shadow-lg shadow-red-900/40 flex items-center justify-center gap-3 group">
                <i class="fa-solid fa-volume-high text-2xl group-hover:animate-bounce"></i>
                <span>SZUKAJ (5% GŁOŚNOŚCI)</span>
            </button>

            <!-- Sound Type Selector -->
            <div class="w-full mt-4 text-left">
                <label class="text-xs font-medium text-slate-300 flex items-center gap-1.5 mb-2">
                    <i class="fa-solid fa-music text-red-400"></i> Rodzaj Dźwięku Szukania:
                </label>
                <div class="grid grid-cols-3 gap-2">
                    <button id="soundSirenaBtn" data-sound="siren" class="sound-type-btn py-2 px-1 bg-red-950/80 border border-red-500 text-red-300 rounded-xl text-xs font-semibold transition-all text-center">
                        <i class="fa-solid fa-bullhorn block mb-1 text-sm"></i> Głośna Syrena
                    </button>
                    <button id="soundPulseBtn" data-sound="pulse" class="sound-type-btn py-2 px-1 bg-slate-800/80 border border-slate-700/80 text-slate-300 hover:bg-slate-700 rounded-xl text-xs font-semibold transition-all text-center">
                        <i class="fa-solid fa-wave-square block mb-1 text-sm"></i> Pulsacyjny
                    </button>
                    <button id="soundHighPitchBtn" data-sound="highpitch" class="sound-type-btn py-2 px-1 bg-slate-800/80 border border-slate-700/80 text-slate-300 hover:bg-slate-700 rounded-xl text-xs font-semibold transition-all text-center">
                        <i class="fa-solid fa-bolt block mb-1 text-sm"></i> Piszczący (3.8kHz)
                    </button>
                </div>
            </div>

            <!-- Earbud Channel Selectors -->
            <div class="grid grid-cols-2 gap-3 w-full mt-4">
                <button id="leftEarbudBtn" class="py-3 px-4 bg-slate-800/80 hover:bg-slate-700 border border-slate-700/80 rounded-xl flex items-center justify-center gap-2 text-sm font-semibold transition-all">
                    <i class="fa-solid fa-ear-listen text-cyan-400"></i>
                    <span>Lewa Słuchawka</span>
                </button>
                <button id="rightEarbudBtn" class="py-3 px-4 bg-slate-800/80 hover:bg-slate-700 border border-slate-700/80 rounded-xl flex items-center justify-center gap-2 text-sm font-semibold transition-all">
                    <i class="fa-solid fa-ear-listen text-pink-400"></i>
                    <span>Prawa Słuchawka</span>
                </button>
            </div>

            <!-- Sound Status indicator -->
            <div id="soundStatus" class="mt-4 text-xs font-mono text-slate-400 flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-slate-500" id="soundStatusDot"></span>
                <span id="soundStatusText">Gotowy do emitowania sygnału</span>
            </div>

            <!-- Custom Volume Control -->
            <div class="w-full mt-6 pt-5 border-t border-slate-800/80">
                <div class="flex justify-between items-center mb-2">
                    <label for="volumeSlider" class="text-xs font-medium text-slate-300 flex items-center gap-1.5">
                        <i class="fa-solid fa-sliders text-slate-400"></i> Poziom głośności:
                    </label>
                    <span id="volumeValue" class="text-xs font-bold font-mono text-red-400 bg-red-950/60 px-2 py-0.5 rounded border border-red-800/40">5%</span>
                </div>
                <input type="range" id="volumeSlider" min="1" max="100" value="5" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-red-500">
                <p class="text-[10px] text-slate-500 mt-1.5 text-left">
                    <i class="fa-solid fa-circle-info mr-1"></i>Kliknięcie "Szukaj" zawsze wymusza powrót głośności do bezpiecznego progu 5%.
                </p>
            </div>
        </div>

        <!-- Radar / Visual Proximity Section -->
        <div class="glass-card rounded-3xl p-6 shadow-2xl flex flex-col items-center relative overflow-hidden">
            <div class="w-full flex justify-between items-center mb-4">
                <h2 class="text-base font-bold text-slate-200 flex items-center gap-2">
                    <i class="fa-solid fa-location-crosshairs text-red-500"></i> Radar Odległości
                </h2>
                <button id="toggleRadarBtn" class="text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 px-3 py-1.5 rounded-lg font-medium border border-slate-700 transition-colors">
                    <i class="fa-solid fa-power-off text-emerald-400 mr-1"></i> Włącz Radar
                </button>
            </div>

            <!-- Radar Visual Display -->
            <div class="relative w-56 h-56 rounded-full border-2 border-emerald-500/30 bg-slate-900/90 flex items-center justify-center my-2 overflow-hidden shadow-inner">
                <!-- Concentric Rings -->
                <div class="absolute w-44 h-44 rounded-full border border-emerald-500/20"></div>
                <div class="absolute w-32 h-32 rounded-full border border-emerald-500/20"></div>
                <div class="absolute w-20 h-20 rounded-full border border-emerald-500/20"></div>
                <div class="absolute w-8 h-8 rounded-full border border-emerald-500/30 bg-emerald-500/10"></div>

                <!-- Crosshairs -->
                <div class="absolute w-full h-[1px] bg-emerald-500/20"></div>
                <div class="absolute h-full w-[1px] bg-emerald-500/20"></div>

                <!-- Radar Sweep Cone -->
                <div id="radarSweep" class="hidden absolute top-0 left-0 w-28 h-28 bg-gradient-to-br from-emerald-500/40 via-emerald-500/10 to-transparent radar-sweep"></div>

                <!-- Target Blips -->
                <div id="radarBlip" class="hidden absolute w-4 h-4 bg-red-500 rounded-full border-2 border-white shadow-[0_0_12px_#ef4444] animate-ping"></div>
                <div id="radarBlipSolid" class="hidden absolute w-3 h-3 bg-red-500 rounded-full border border-white shadow-[0_0_8px_#ef4444]"></div>

                <!-- User Center Dot -->
                <div class="w-2 h-2 bg-emerald-400 rounded-full shadow-[0_0_6px_#34d399] z-10"></div>
            </div>

            <!-- Distance & Signal Info -->
            <div class="grid grid-cols-2 gap-3 w-full mt-4 text-center">
                <div class="bg-slate-900/60 p-2.5 rounded-xl border border-slate-800">
                    <span class="text-[11px] text-slate-400 block uppercase font-medium">Szacowana Odległość</span>
                    <span id="distanceText" class="text-sm font-bold text-slate-200">Nieaktywny</span>
                </div>
                <div class="bg-slate-900/60 p-2.5 rounded-xl border border-slate-800">
                    <span class="text-[11px] text-slate-400 block uppercase font-medium">Siła Sygnału</span>
                    <span id="rssiText" class="text-sm font-bold text-emerald-400">--- dBm</span>
                </div>
            </div>

            <!-- Bluetooth Scan trigger -->
            <button id="btScanBtn" class="mt-4 w-full py-2.5 px-4 bg-slate-800 hover:bg-slate-700 active:scale-98 border border-slate-700/80 rounded-xl text-xs font-semibold text-slate-300 flex items-center justify-center gap-2 transition-all">
                <i class="fa-brands fa-bluetooth-b text-blue-400"></i>
                <span>Połącz / Skanuj Bluetooth</span>
            </button>
        </div>

        <!-- Footer Info -->
        <footer class="text-center text-xs text-slate-500 py-2">
            Aplikacja wspomagająca lokalizację słuchawek dousznych &copy; SondejM
        </footer>
    </div>

    <!-- JavaScript Logic -->
    <script>
        // State variables
        let audioCtx = null;
        let masterGainNode = null;
        let isPlaying = false;
        let soundInterval = null;
        let activeSoundType = 'siren'; // 'siren', 'pulse', 'highpitch'
        let currentVolume = 0.05; // 5% default
        let activeEarbud = 'both'; // 'left', 'right', 'both'

        let isRadarActive = false;
        let radarSimulationInterval = null;
        let simulatedDistance = 80;

        // UI Element References
        const searchBtn = document.getElementById('searchBtn');
        const soundTypeBtns = document.querySelectorAll('.sound-type-btn');
        const leftEarbudBtn = document.getElementById('leftEarbudBtn');
        const rightEarbudBtn = document.getElementById('rightEarbudBtn');
        const volumeSlider = document.getElementById('volumeSlider');
        const volumeValue = document.getElementById('volumeValue');
        const soundStatusText = document.getElementById('soundStatusText');
        const soundStatusDot = document.getElementById('soundStatusDot');

        const toggleRadarBtn = document.getElementById('toggleRadarBtn');
        const radarSweep = document.getElementById('radarSweep');
        const radarBlip = document.getElementById('radarBlip');
        const radarBlipSolid = document.getElementById('radarBlipSolid');
        const distanceText = document.getElementById('distanceText');
        const rssiText = document.getElementById('rssiText');
        const btScanBtn = document.getElementById('btScanBtn');

        // --- WEB AUDIO API (Sound Generators with Enhanced Gain Output) ---
        function initAudioContext() {
            if (!audioCtx) {
                const AudioContext = window.AudioContext || window.webkitAudioContext;
                audioCtx = new AudioContext();

                // Master Gain Node for boosting maximum achievable loudness
                masterGainNode = audioCtx.createGain();
                masterGainNode.gain.setValueAtTime(1.0, audioCtx.currentTime);
                masterGainNode.connect(audioCtx.destination);
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        }

        // Helper to apply stereo panning
        function applyStereoPanning(panner, now) {
            if (!panner) return;
            if (activeEarbud === 'left') panner.pan.setValueAtTime(-1, now);
            else if (activeEarbud === 'right') panner.pan.setValueAtTime(1, now);
            else panner.pan.setValueAtTime(0, now);
        }

        // Calculate acoustic gain curve: Gives a strong output at 5% and maximum drive up to 100%
        function getEffectiveLoudnessGain() {
            // Amplification scaling factor
            const baseMultiplier = 2.5; 
            return currentVolume * baseMultiplier;
        }

        // Play Sound Tone based on active mode with optimized piercing frequencies
        function playSelectedSound() {
            if (!audioCtx) return;

            try {
                const now = audioCtx.currentTime;
                const osc = audioCtx.createOscillator();
                const gainNode = audioCtx.createGain();
                const panner = audioCtx.createStereoPanner ? audioCtx.createStereoPanner() : null;

                applyStereoPanning(panner, now);

                const effectiveGain = getEffectiveLoudnessGain();

                if (activeSoundType === 'siren') {
                    // 1. Syrena - High piercing sweep (2000Hz -> 4800Hz) optimized for earbud drivers
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(2000, now);
                    osc.frequency.linearRampToValueAtTime(4800, now + 0.22);
                    osc.frequency.linearRampToValueAtTime(2000, now + 0.44);

                    gainNode.gain.setValueAtTime(0.001, now);
                    gainNode.gain.linearRampToValueAtTime(effectiveGain, now + 0.03);
                    gainNode.gain.setValueAtTime(effectiveGain, now + 0.41);
                    gainNode.gain.linearRampToValueAtTime(0.001, now + 0.45);

                    connectNodes(osc, gainNode, panner);
                    osc.start(now);
                    osc.stop(now + 0.46);

                } else if (activeSoundType === 'pulse') {
                    // 2. Dźwięk pulsacyjny - Sharp 3200Hz bursts
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(3200, now);

                    gainNode.gain.setValueAtTime(0.001, now);
                    gainNode.gain.linearRampToValueAtTime(effectiveGain, now + 0.01);
                    gainNode.gain.setValueAtTime(effectiveGain, now + 0.12);
                    gainNode.gain.linearRampToValueAtTime(0.001, now + 0.14);

                    connectNodes(osc, gainNode, panner);
                    osc.start(now);
                    osc.stop(now + 0.15);

                } else if (activeSoundType === 'highpitch') {
                    // 3. Wysokie tony - Piercing 3800Hz to 4500Hz exponential chirp
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(3800, now);
                    osc.frequency.exponentialRampToValueAtTime(4500, now + 0.2);

                    gainNode.gain.setValueAtTime(0.001, now);
                    gainNode.gain.linearRampToValueAtTime(effectiveGain, now + 0.02);
                    gainNode.gain.setValueAtTime(effectiveGain, now + 0.18);
                    gainNode.gain.exponentialRampToValueAtTime(0.001, now + 0.22);

                    connectNodes(osc, gainNode, panner);
                    osc.start(now);
                    osc.stop(now + 0.23);
                }
            } catch (err) {
                console.error('Błąd audio:', err);
            }
        }

        function connectNodes(osc, gainNode, panner) {
            if (panner) {
                osc.connect(gainNode);
                gainNode.connect(panner);
                panner.connect(masterGainNode || audioCtx.destination);
            } else {
                osc.connect(gainNode);
                gainNode.connect(masterGainNode || audioCtx.destination);
            }
        }

        function startEmittingSound() {
            initAudioContext();
            stopEmittingSound(); // Clear any existing loop

            isPlaying = true;
            playSelectedSound();

            // Set pulse timing interval according to sound selection
            const intervalMs = activeSoundType === 'siren' ? 500 : (activeSoundType === 'pulse' ? 220 : 280);
            soundInterval = setInterval(playSelectedSound, intervalMs);

            soundStatusText.innerText = "Emitowanie sygnału lokalizacyjnego...";
            soundStatusText.classList.replace('text-slate-400', 'text-red-400');
            soundStatusDot.className = "w-2 h-2 rounded-full bg-red-500 animate-ping";
            searchBtn.classList.add('ring-4', 'ring-red-500/50');
        }

        function stopEmittingSound() {
            isPlaying = false;
            if (soundInterval) {
                clearInterval(soundInterval);
                soundInterval = null;
            }
            soundStatusText.innerText = "Gotowy do emitowania sygnału";
            soundStatusText.className = "text-slate-400";
            soundStatusDot.className = "w-2 h-2 rounded-full bg-slate-500";
            searchBtn.classList.remove('ring-4', 'ring-red-500/50');
        }

        // --- BUTTON EVENT LISTENERS ---

        // "Szukaj" button action - Resets volume to 5% every time
        searchBtn.addEventListener('click', () => {
            currentVolume = 0.05;
            volumeSlider.value = 5;
            volumeValue.innerText = "5%";

            if (isPlaying) {
                stopEmittingSound();
            } else {
                startEmittingSound();
            }
        });

        // Sound Type Selector Listeners
        soundTypeBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                const targetBtn = e.currentTarget;
                activeSoundType = targetBtn.getAttribute('data-sound');

                // Update UI styles
                soundTypeBtns.forEach(b => {
                    b.className = "sound-type-btn py-2 px-1 bg-slate-800/80 border border-slate-700/80 text-slate-300 hover:bg-slate-700 rounded-xl text-xs font-semibold transition-all text-center";
                });
                targetBtn.className = "sound-type-btn py-2 px-1 bg-red-950/80 border border-red-500 text-red-300 rounded-xl text-xs font-semibold transition-all text-center";

                // Restart loop with new sound parameters if active
                if (isPlaying) {
                    startEmittingSound();
                }
            });
        });

        // Volume Slider listener
        volumeSlider.addEventListener('input', (e) => {
            const val = parseInt(e.target.value, 10);
            currentVolume = val / 100;
            volumeValue.innerText = `${val}%`;
        });

        // Earbud channel toggles
        leftEarbudBtn.addEventListener('click', () => {
            if (activeEarbud === 'left') {
                activeEarbud = 'both';
                leftEarbudBtn.classList.remove('bg-cyan-950', 'border-cyan-500', 'text-cyan-300');
            } else {
                activeEarbud = 'left';
                leftEarbudBtn.classList.add('bg-cyan-950', 'border-cyan-500', 'text-cyan-300');
                rightEarbudBtn.classList.remove('bg-pink-950', 'border-pink-500', 'text-pink-300');
            }
            if (isPlaying) startEmittingSound();
        });

        rightEarbudBtn.addEventListener('click', () => {
            if (activeEarbud === 'right') {
                activeEarbud = 'both';
                rightEarbudBtn.classList.remove('bg-pink-950', 'border-pink-500', 'text-pink-300');
            } else {
                activeEarbud = 'right';
                rightEarbudBtn.classList.add('bg-pink-950', 'border-pink-500', 'text-pink-300');
                leftEarbudBtn.classList.remove('bg-cyan-950', 'border-cyan-500', 'text-cyan-300');
            }
            if (isPlaying) startEmittingSound();
        });

        // --- RADAR & PROXIMITY LOGIC ---

        function toggleRadar(enable) {
            isRadarActive = enable !== undefined ? enable : !isRadarActive;

            if (isRadarActive) {
                toggleRadarBtn.innerHTML = `<i class="fa-solid fa-power-off text-red-400 mr-1"></i> Wyłącz Radar`;
                toggleRadarBtn.classList.replace('bg-slate-800', 'bg-slate-700');
                radarSweep.classList.remove('hidden');
                radarBlip.classList.remove('hidden');
                radarBlipSolid.classList.remove('hidden');

                startRadarSimulation();
            } else {
                toggleRadarBtn.innerHTML = `<i class="fa-solid fa-power-off text-emerald-400 mr-1"></i> Włącz Radar`;
                toggleRadarBtn.classList.replace('bg-slate-700', 'bg-slate-800');
                radarSweep.classList.add('hidden');
                radarBlip.classList.add('hidden');
                radarBlipSolid.classList.add('hidden');

                if (radarSimulationInterval) clearInterval(radarSimulationInterval);
                distanceText.innerText = "Nieaktywny";
                distanceText.className = "text-sm font-bold text-slate-200";
                rssiText.innerText = "--- dBm";
            }
        }

        toggleRadarBtn.addEventListener('click', () => toggleRadar());

        // Radar simulation loop
        function startRadarSimulation() {
            if (radarSimulationInterval) clearInterval(radarSimulationInterval);

            let angle = Math.random() * Math.PI * 2;

            radarSimulationInterval = setInterval(() => {
                simulatedDistance += (Math.random() - 0.52) * 8;
                if (simulatedDistance < 10) simulatedDistance = 10;
                if (simulatedDistance > 90) simulatedDistance = 90;

                angle += 0.05;

                const radiusPx = (simulatedDistance / 100) * 85; 
                const x = Math.cos(angle) * radiusPx;
                const y = Math.sin(angle) * radiusPx;

                radarBlip.style.transform = `translate(${x}px, ${y}px)`;
                radarBlipSolid.style.transform = `translate(${x}px, ${y}px)`;

                const estimatedRssi = Math.round(-40 - (simulatedDistance / 100) * 55);
                rssiText.innerText = `${estimatedRssi} dBm`;

                if (simulatedDistance < 30) {
                    distanceText.innerText = "BLISKO (< 1m)";
                    distanceText.className = "text-sm font-bold text-emerald-400 animate-pulse";
                } else if (simulatedDistance < 65) {
                    distanceText.innerText = "ŚREDNIO (~ 2-4m)";
                    distanceText.className = "text-sm font-bold text-amber-400";
                } else {
                    distanceText.innerText = "DALEKO (> 5m)";
                    distanceText.className = "text-sm font-bold text-rose-400";
                }
            }, 300);
        }

        // Bluetooth Scan
        btScanBtn.addEventListener('click', async () => {
            if (!navigator.bluetooth) {
                alert('Twoja przeglądarka nie wspiera bezpośredniego skanowania Web Bluetooth. Radar przełączy się w tryb symulacji sygnału.');
                toggleRadar(true);
                return;
            }

            try {
                soundStatusText.innerText = "Wyszukiwanie urządzeń Bluetooth...";
                const device = await navigator.bluetooth.requestDevice({
                    acceptAllDevices: true,
                    optionalServices: ['battery_service']
                });

                if (device) {
                    toggleRadar(true);
                    soundStatusText.innerText = `Połączono z: ${device.name || 'Słuchawki'}`;
                }
            } catch (err) {
                console.log('Bluetooth scan canceled or failed:', err);
                toggleRadar(true);
            }
        });
    </script>
</body>
</html>
