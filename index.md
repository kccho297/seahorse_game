<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Seahorse Survival: The Great Escape</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="description" content="A high-stakes survival game where players guide a seahorse to escape a trawler net.">

    <script src="https://cdn.tailwindcss.com"></script>

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Bubblegum+Sans&family=Fredoka:wght@300;400;600;700&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        game: ['Bubblegum Sans', 'cursive'],
                        ui: ['Fredoka', 'sans-serif'],
                    },
                    colors: {
                        ocean: { 900: '#0f172a', 800: '#1e293b', 400: '#38bdf8' },
                        seahorse: '#fbbf24',
                        plankton: '#f472b6',
                        net: '#10b981'
                    },
                    animation: {
                        'pulse-fast': 'pulse 1s cubic-bezier(0.4, 0, 0.6, 1) infinite',
                        'float': 'float 3s ease-in-out infinite',
                    },
                    keyframes: {
                        float: {
                            '0%, 100%': { transform: 'translateY(0)' },
                            '50%': { transform: 'translateY(-10px)' },
                        }
                    }
                }
            }
        }
    </script>

    <style>
        body {
            background-color: #020617;
            color: #fff;
            margin: 0;
            overflow: hidden; /* Prevent body scroll */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Fredoka', sans-serif;
        }

        /* The Game Frame */
        #game-container {
            position: relative;
            width: 1000px;
            height: 600px;
            max-width: 95vw;
            max-height: 90vh; /* Keep some margin on mobile */
            background-color: #0f172a;
            border-radius: 24px;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.7);
            border: 4px solid #1e293b;
            overflow: hidden;
            touch-action: none;
            user-select: none;
        }

        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: none;
            z-index: 20;
        }
        .screen.active {
            display: flex;
        }
        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <!-- Game Canvas -->
        <canvas id="gameCanvas"></canvas>

        <!-- HUD: Playing State -->
        <div id="hud-playing" class="absolute top-0 left-0 w-full p-4 flex justify-between items-start z-10 pointer-events-none hidden">
            <div class="flex flex-col gap-4 w-64">
                <!-- Player 1 Status -->
                <div id="p1-hud" class="hidden">
                    <div class="flex justify-between text-white text-sm font-ui font-bold drop-shadow-md mb-1">
                        <span class="text-yellow-400">P1 (WASD + L)</span>
                        <span id="p1-energy-text">100%</span>
                    </div>
                    <div class="h-3 bg-slate-800/50 rounded-full border border-white/20 overflow-hidden backdrop-blur">
                        <div id="p1-energy-bar" class="h-full bg-yellow-400 transition-all duration-300" style="width: 100%"></div>
                    </div>
                </div>
                <!-- Player 2 Status -->
                <div id="p2-hud" class="hidden">
                    <div class="flex justify-between text-white text-sm font-ui font-bold drop-shadow-md mb-1">
                        <span class="text-cyan-400">P2 (ARROWS + ENTER)</span>
                        <span id="p2-energy-text">100%</span>
                    </div>
                    <div class="h-3 bg-slate-800/50 rounded-full border border-white/20 overflow-hidden backdrop-blur">
                        <div id="p2-energy-bar" class="h-full bg-cyan-400 transition-all duration-300" style="width: 100%"></div>
                    </div>
                </div>
            </div>

            <!-- Score Display -->
            <div class="text-center">
                <div id="score-single" class="hidden text-5xl text-white font-bold drop-shadow-[0_4px_4px_rgba(0,0,0,0.8)] font-game">0m</div>
                <div id="score-multi" class="hidden flex gap-8 bg-slate-900/50 p-3 rounded-2xl backdrop-blur-md border border-white/10 font-ui">
                    <div class="flex flex-col items-center">
                        <span class="text-yellow-400 text-xs font-bold uppercase">Player 1</span>
                        <span id="score-p1" class="text-2xl text-white font-bold">0m</span>
                    </div>
                    <div class="w-px bg-white/20"></div>
                    <div class="flex flex-col items-center">
                        <span class="text-cyan-400 text-xs font-bold uppercase">Player 2</span>
                        <span id="score-p2" class="text-2xl text-white font-bold">0m</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- HUD: Warning -->
        <div id="hud-warning" class="absolute top-1/4 left-0 w-full flex-col items-center justify-center animate-pulse-fast z-10 pointer-events-none hidden">
            <div class="bg-red-600 text-white px-8 py-4 rounded-xl border-4 border-red-800 shadow-2xl flex flex-col items-center">
                <div class="flex items-center gap-4 text-4xl font-bold uppercase tracking-widest font-game">
                    ⚠ ️ TRAWLER DETECTED ⚠ ️
                </div>
                <div class="text-center text-xl mt-2 font-ui font-bold">SWIM UP!</div>
            </div>
        </div>

        <!-- Screen: Menu -->
        <div id="screen-menu" class="screen active flex-col items-center justify-center bg-ocean-900/95 backdrop-blur-sm">
            <div class="text-center mb-6 animate-in fade-in zoom-in duration-500">
                <div class="flex justify-center mb-4 animate-float">
                    <div class="bg-white/10 p-6 rounded-full shadow-[0_0_50px_rgba(56,189,248,0.3)]">
                        <svg width="100" height="100" viewBox="-40 -40 80 80" fill="none">
                            <path d="M-10 1 Q -25 5, -10 15" stroke="#fbbf24" stroke-width="1" fill="#fcd34d" />
                            <path d="M0 20 C 1 30, 20 50, 15 30 S 2 30, 5 15" stroke="#fbbf24" stroke-width="8" stroke-linecap="round" fill="none" />
                            <ellipse cx="0" cy="10" rx="12" ry="18" fill="#fbbf24" />
                            <ellipse cx="3" cy="10" rx="8" ry="12" fill="#fcd34d" />
                            <circle cx="0" cy="-15" r="14" fill="#fbbf24" />
                            <rect x="10" y="-19" width="18" height="9" rx="4.5" transform="rotate(12 10 -15)" fill="#fbbf24" />
                            <circle cx="4" cy="-16" r="6.5" fill="white" />
                            <circle cx="7" cy="-16" r="3.5" fill="#0f172a" />
                            <circle cx="0" cy="-8" r="3.5" fill="#f472b6" fill-opacity="0.6" />
                        </svg>
                    </div>
                </div>
                <h1 class="text-6xl text-transparent bg-clip-text bg-gradient-to-b from-yellow-300 to-orange-500 font-bold drop-shadow-lg mb-2 font-game">
                    Seahorse Survival
                </h1>
                <p class="text-lg text-blue-200 font-ui tracking-widest uppercase">The Great Escape</p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 font-ui mb-8">
                <button onclick="startGame(1)" class="group bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-yellow-400 text-white w-56 h-40 rounded-3xl flex flex-col items-center justify-center gap-3 transition-all hover:-translate-y-2 shadow-xl cursor-pointer">
                    <div class="bg-slate-900/50 p-3 rounded-full group-hover:scale-110 transition-transform">
                        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-slate-400 group-hover:text-yellow-400"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                    </div>
                    <div class="text-center">
                        <span class="text-xl font-bold block mb-1">1 Player</span>
                        <span class="text-xs text-slate-400 uppercase tracking-wider">WASD to Swim</span>
                    </div>
                </button>

                <button onclick="startGame(2)" class="group bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-cyan-400 text-white w-56 h-40 rounded-3xl flex flex-col items-center justify-center gap-3 transition-all hover:-translate-y-2 shadow-xl cursor-pointer">
                    <div class="flex gap-2 bg-slate-900/50 p-3 rounded-full group-hover:scale-110 transition-transform">
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-slate-400 group-hover:text-yellow-400"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-slate-400 group-hover:text-cyan-400"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                    </div>
                    <div class="text-center">
                        <span class="text-xl font-bold block mb-1">2 Players</span>
                        <span class="text-xs text-slate-400 uppercase tracking-wider">Arrows for P2</span>
                    </div>
                </button>
            </div>

            <div class="max-w-lg text-center text-slate-400 text-xs font-ui bg-slate-900/50 p-3 rounded-xl border border-slate-700">
                <p>
                    <span class="text-yellow-400 font-bold">TIP:</span> Use your <span class="text-white font-bold px-1 bg-slate-700 rounded">L</span> or <span class="text-white font-bold px-1 bg-slate-700 rounded">ENTER</span> key to camouflage! Hiding in seagrass restores energy, but hiding in open water drains it.
                </p>
            </div>
        </div>

        <!-- Screen: Game Over -->
        <div id="screen-gameover" class="screen flex-col items-center justify-center bg-red-900/90 backdrop-blur-md font-ui">
            <div class="max-w-md w-full bg-slate-900 border border-red-500/30 p-8 rounded-3xl shadow-2xl text-center">
                <div class="mb-4 text-red-500 animate-pulse">
                    <svg width="56" height="56" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mx-auto">
                        <circle cx="9" cy="12" r="1"/><circle cx="15" cy="12" r="1"/>
                        <path d="M8 20v2h8v-2"/><path d="M12.5 17l.5-2"/>
                        <path d="M16 20a2 2 0 0 0 1.56-3.25 8 8 0 1 0-11.12 0A2 2 0 0 0 8 20"/>
                    </svg>
                </div>

                <h2 class="text-3xl text-white font-bold mb-2 font-game tracking-wide">GAME OVER</h2>
                <p id="gameover-reason" class="text-red-200 mb-6 uppercase tracking-widest text-xs font-bold">ELIMINATED</p>

                <!-- Single Player Score -->
                <div id="go-single" class="text-6xl font-bold text-yellow-400 mb-6 drop-shadow-lg font-game hidden">0m</div>

                <!-- Multi Player Score -->
                <div id="go-multi" class="hidden bg-slate-800 p-4 rounded-xl mb-6 border border-white/10">
                    <div id="go-p1-row" class="flex justify-between items-center p-3 rounded-lg mb-2 bg-white/5 border border-transparent">
                        <span class="font-bold text-slate-400" id="go-p1-label">Player 1</span>
                        <span class="text-2xl font-bold text-white" id="go-p1-score">0m</span>
                    </div>
                    <div id="go-p2-row" class="flex justify-between items-center p-3 rounded-lg bg-white/5 border border-transparent">
                        <span class="font-bold text-slate-400" id="go-p2-label">Player 2</span>
                        <span class="text-2xl font-bold text-white" id="go-p2-score">0m</span>
                    </div>
                </div>

                <div class="bg-slate-800 p-4 rounded-xl border-l-4 border-blue-400 mb-6 text-left shadow-lg">
                    <h3 class="text-blue-400 font-bold uppercase text-[10px] tracking-widest mb-1 flex items-center gap-2">
                        Did You Know?
                    </h3>
                    <p id="gameover-fact" class="text-slate-200 leading-snug text-sm">
                        Seahorses are cool.
                    </p>
                </div>

                <button onclick="showMenu()" class="w-full bg-white hover:bg-slate-200 text-slate-900 font-bold text-lg py-3 rounded-xl shadow-lg flex items-center justify-center gap-2 transition-all hover:scale-105 transform font-game cursor-pointer">
                    PLAY AGAIN
                </button>
            </div>
        </div>
    </div>

    <script>
        // --- CONSTANTS & CONFIG ---
        const CONFIG = {
            PLAYER_ACCEL: 0.4,
            PLAYER_MAX_SPEED: 5,
            PLAYER_FRICTION: 0.92,
            PLAYER_RADIUS: 20,
            ENERGY_DECAY_RATE: 0.03,
            ENERGY_MOVE_COST: 0.01,
            ENERGY_CAMO_DRAIN: 0.3,
            BASE_SCROLL_SPEED: 3.0,
            DIFFICULTY_RAMP: 0.0005,
            MAX_SPEED_MULTIPLIER: 4.0,
            SPAWN_RATES: { 
                PLANKTON: 0.08, 
                PLASTIC: 0.02, 
                PREDATOR: 0.015, 
                SEAGRASS: 0.03 
            },
            TRAWL_INTERVAL: 1800,
            TRAWL_WARNING_DURATION: 180,
            TRAWL_SPEED_BASE: 5,
            TRAWL_HEIGHT_RATIO: 0.45,
        };

        const COLORS = {
            waterTop: '#1e40af', waterBottom: '#0f172a',
            seahorseBelly: '#fcd34d', plankton: '#f472b6',
            plastic: '#94a3b8', net: '#065f46', warning: '#ef4444',
            blood: '#ef4444', bubble: '#e0f2fe'
        };

        const FACTS = {
            net: [
                "Bottom Trawling involves dragging heavy weighted nets across the seafloor, clear-cutting the ocean floor.",
                "Trawling destroys ancient coral reefs and sponge gardens that take centuries to grow.",
                "An estimated 37 million seahorses are caught as 'bycatch' in trawling nets every year."
            ],
            plastic: [
                "Seahorses clutch onto drifting debris like cotton swabs, travelling miles into polluted waters.",
                "Microplastics block digestive systems, which is fatal for seahorses within hours.",
                "Ghost nets (abandoned gear) continue to trap and kill marine life for decades."
            ],
            starvation: [
                "Seahorses have no true stomach and teeth. They suck food like a vacuum cleaner.",
                "A seahorse must eat 30-50 times a day just to stay alive.",
                "When seagrass beds disappear due to pollution, plankton populations collapse."
            ],
            predator: [
                "Seahorses are masters of camouflage, changing color to match their surroundings.",
                "Seahorses are poor swimmers and are easy snacks for crabs, tuna, and rays in open water.",
                "Use your Camouflage power to hide from predators!"
            ]
        };

        const PLAYERS_INIT = [
            {
                id: 1, color: '#fbbf24', name: "P1",
                controls: { 
                    up: ['KeyW'], down: ['KeyS'], left: ['KeyA'], right: ['KeyD'], 
                    anchor: ['KeyL']
                },
                yOffset: 0.4
            },
            {
                id: 2, color: '#22d3ee', name: "P2",
                controls: { 
                    up: ['ArrowUp'], down: ['ArrowDown'], left: ['ArrowLeft'], right: ['ArrowRight'], 
                    anchor: ['Enter', 'ControlRight'] 
                },
                yOffset: 0.6
            }
        ];

        // --- GLOBAL STATE ---
        const canvas = document.getElementById('gameCanvas');
        const container = document.getElementById('game-container'); // Ref to container
        const ctx = canvas.getContext('2d');
        let requestID;
        let selectedPlayerCount = 1;
        const keys = new Set();
        
        // Game Objects
        let players = [];
        let entities = [];
        let particles = [];
        let net = { active: false, x: -500, speed: 0, warningPhase: false };
        let frameCount = 0;
        let gameSpeed = 1.0;
        let shake = 0;

        // --- INPUT HANDLING ---
        window.addEventListener('keydown', (e) => keys.add(e.code));
        window.addEventListener('keyup', (e) => keys.delete(e.code));
        window.addEventListener('resize', resizeCanvas);

        function resizeCanvas() {
            // Resize canvas to match the container, not the window
            canvas.width = container.clientWidth;
            canvas.height = container.clientHeight;
        }
        resizeCanvas();

        // --- UI NAVIGATION ---
        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(el => el.classList.remove('active'));
            if(id) document.getElementById(id).classList.add('active');
        }

        function showMenu() {
            showScreen('screen-menu');
            document.getElementById('hud-playing').classList.add('hidden');
            document.getElementById('hud-warning').classList.add('hidden');
        }

        function startGame(count) {
            selectedPlayerCount = count;
            showScreen(null); // Hide menu
            
            // Reset Game State
            players = [];
            entities = [];
            particles = [];
            net = { active: false, x: -500, speed: 0, warningPhase: false };
            frameCount = 0;
            gameSpeed = 1.0;
            shake = 0;
            keys.clear();

            // Setup Players
            for(let i=0; i<count; i++) {
                const conf = PLAYERS_INIT[i];
                players.push({
                    config: conf,
                    x: 100,
                    y: canvas.height * conf.yOffset,
                    vx: 0, vy: 0,
                    radius: CONFIG.PLAYER_RADIUS,
                    energy: 100,
                    maxEnergy: 100,
                    distance: 0,
                    isAnchored: false,
                    isCamouflaged: false,
                    rotation: 0,
                    opacity: 1,
                    isDead: false
                });
            }

            // Setup HUD visibility
            document.getElementById('hud-playing').classList.remove('hidden');
            document.getElementById('p1-hud').classList.remove('hidden');
            document.getElementById('p2-hud').classList.toggle('hidden', count < 2);
            document.getElementById('score-single').classList.toggle('hidden', count > 1);
            document.getElementById('score-multi').classList.toggle('hidden', count < 2);

            // Start Loop
            if (requestID) cancelAnimationFrame(requestID);
            loop();
        }

        function triggerGameOver(reason) {
            cancelAnimationFrame(requestID);
            
            // UI Logic
            const validReason = FACTS[reason] ? reason : 'starvation';
            const factList = FACTS[validReason];
            const fact = factList[Math.floor(Math.random() * factList.length)];

            document.getElementById('gameover-reason').textContent = reason === 'net' ? 'Captured by Trawler' : reason.replace('_', ' ');
            document.getElementById('gameover-fact').textContent = fact;

            const p1Dist = Math.floor(players[0]?.distance || 0);
            const p2Dist = Math.floor(players[1]?.distance || 0);

            if(selectedPlayerCount === 1) {
                document.getElementById('go-single').textContent = p1Dist + "m";
                document.getElementById('go-single').classList.remove('hidden');
                document.getElementById('go-multi').classList.add('hidden');
            } else {
                document.getElementById('go-single').classList.add('hidden');
                document.getElementById('go-multi').classList.remove('hidden');
                document.getElementById('go-p1-score').textContent = p1Dist + "m";
                document.getElementById('go-p2-score').textContent = p2Dist + "m";
                
                const p1Win = p1Dist >= p2Dist;
                const winClass = "bg-yellow-400/20 border-yellow-400/50";
                const loseClass = "bg-white/5 border-transparent";
                
                // Style winner
                const p1Row = document.getElementById('go-p1-row');
                const p2Row = document.getElementById('go-p2-row');
                
                p1Row.className = `flex justify-between items-center p-3 rounded-lg mb-2 border ${p1Win ? winClass : loseClass}`;
                document.getElementById('go-p1-label').className = `font-bold ${p1Win ? 'text-yellow-400' : 'text-slate-400'}`;
                
                p2Row.className = `flex justify-between items-center p-3 rounded-lg border ${!p1Win ? 'bg-cyan-400/20 border-cyan-400/50' : loseClass}`;
                document.getElementById('go-p2-label').className = `font-bold ${!p1Win ? 'text-cyan-400' : 'text-slate-400'}`;
            }

            showScreen('screen-gameover');
        }

        // --- GAME LOGIC ---
        function spawnExplosion(x, y, color, count=20) {
            for(let i=0; i<count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = Math.random() * 5 + 2;
                particles.push({
                    x, y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    life: 40 + Math.random() * 20,
                    maxLife: 60,
                    color,
                    size: Math.random() * 4 + 2
                });
            }
        }

        function killPlayer(p, reason) {
            if(p.isDead) return;
            p.isDead = true;
            p.opacity = 0;
            
            if (reason === 'net') { 
                spawnExplosion(p.x, p.y, p.config.color, 50); 
                spawnExplosion(p.x, p.y, '#fff', 30); 
            }
            else if (reason === 'predator') spawnExplosion(p.x, p.y, COLORS.blood, 60);
            else if (reason === 'plastic') spawnExplosion(p.x, p.y, COLORS.plastic, 30);
            else spawnExplosion(p.x, p.y, COLORS.bubble, 30);

            if(players.every(pl => pl.isDead)) {
                setTimeout(() => triggerGameOver(reason), 1000);
            }
        }

        function spawnEntity(type, x, y) {
            let w=10, h=10, vx=0, vy=0, spawnY=y;
            const speedMult = gameSpeed;

            switch(type) {
                case 'plankton': w=10; h=10; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult; vy=(Math.random()-0.5); break;
                case 'plastic': w=25; h=25; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult*1.2; vy=(Math.random()-0.5)*0.5; break;
                case 'predator_tuna': w=60; h=40; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult*2.8; break;
                case 'predator_crab': w=60; h=40; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult; spawnY=canvas.height-50; break;
                case 'seagrass': w=20; h=80+Math.random()*80; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult; spawnY=canvas.height; break;
            }
            entities.push({ type, x: x, y: spawnY, width: w, height: h, vx, vy, rotation: 0, id: Math.random() });
        }

        function loop() {
            // Update
            frameCount++;
            if(gameSpeed < CONFIG.MAX_SPEED_MULTIPLIER) gameSpeed += CONFIG.DIFFICULTY_RAMP;
            const scrollSpeed = CONFIG.BASE_SCROLL_SPEED * gameSpeed;
            const distStep = scrollSpeed * 0.05;

            // Player Logic
            players.forEach(p => {
                if(p.isDead) return;
                p.distance += distStep;

                let dx=0, dy=0, action=false;
                const ctrl = p.config.controls;
                if(ctrl.up.some(k => keys.has(k))) dy-=1;
                if(ctrl.down.some(k => keys.has(k))) dy+=1;
                if(ctrl.left.some(k => keys.has(k))) dx-=1;
                if(ctrl.right.some(k => keys.has(k))) dx+=1;
                if(ctrl.anchor.some(k => keys.has(k))) action=true;

                if(dx!==0 && dy!==0) { dx*=0.707; dy*=0.707; }

                // Camouflage / Anchor
                const nearSeagrass = entities.some(e => e.type === 'seagrass' && Math.hypot(p.x-e.x, p.y-(e.y-e.height/2)) < 80);
                if(action) {
                    p.isCamouflaged = true;
                    p.opacity = 0.4;
                    if(nearSeagrass) { p.isAnchored = true; }
                    else { p.isAnchored = false; p.energy -= CONFIG.ENERGY_CAMO_DRAIN; }
                } else {
                    p.isCamouflaged = false;
                    p.isAnchored = false;
                    p.opacity = 1.0;
                }

                // Physics
                if(p.isAnchored) {
                    p.x -= scrollSpeed;
                    if(p.x < p.radius) { p.isAnchored=false; p.x=p.radius; }
                    p.energy -= CONFIG.ENERGY_DECAY_RATE * 0.1;
                    p.vx *= 0.8; p.vy *= 0.8;
                } else {
                    p.vx += dx * CONFIG.PLAYER_ACCEL;
                    p.vy += dy * CONFIG.PLAYER_ACCEL;
                    p.vx *= CONFIG.PLAYER_FRICTION;
                    p.vy *= CONFIG.PLAYER_FRICTION;

                    const spd = Math.hypot(p.vx, p.vy);
                    if(spd > CONFIG.PLAYER_MAX_SPEED) {
                        p.vx = (p.vx/spd)*CONFIG.PLAYER_MAX_SPEED;
                        p.vy = (p.vy/spd)*CONFIG.PLAYER_MAX_SPEED;
                    }
                    p.x += p.vx;
                    p.y += p.vy;
                    p.energy -= (CONFIG.ENERGY_DECAY_RATE + spd * CONFIG.ENERGY_MOVE_COST);
                    if(spd > 0.5) p.rotation = p.rotation*0.8 + Math.atan2(p.vy, p.vx)*0.5*0.2;
                }

                // Bounds - ensure we are inside canvas dimensions
                p.x = Math.max(p.radius, Math.min(canvas.width-p.radius, p.x));
                p.y = Math.max(p.radius, Math.min(canvas.height-p.radius, p.y));

                if(p.energy <= 0) killPlayer(p, 'starvation');

                // Net Turbulence
                if(net.warningPhase || net.active) {
                    p.x += (Math.random()-0.5)*2;
                    p.y += (Math.random()-0.5)*2;
                    if(net.active && net.x - p.x > 0 && net.x - p.x < 300) p.vx += 0.15;
                }
            });

            // Net Logic
            const cycle = frameCount % CONFIG.TRAWL_INTERVAL;
            const warningEl = document.getElementById('hud-warning');
            
            if(cycle === CONFIG.TRAWL_INTERVAL - CONFIG.TRAWL_WARNING_DURATION) {
                net.warningPhase = true;
                shake = 5;
                warningEl.classList.remove('hidden');
                warningEl.classList.add('flex');
            }
            if(cycle === 0 && frameCount > 0) {
                net.warningPhase = false;
                net.active = true;
                net.x = canvas.width + 200;
                net.speed = CONFIG.TRAWL_SPEED_BASE * (1 + (gameSpeed-1)*0.5);
                warningEl.classList.add('hidden');
                warningEl.classList.remove('flex');
            }
            if(net.active) {
                net.x -= net.speed;
                const netH = canvas.height * CONFIG.TRAWL_HEIGHT_RATIO;
                const netY = canvas.height - netH;

                players.forEach(p => {
                    if(!p.isDead && p.x > net.x && p.x < net.x+150 && p.y > netY) killPlayer(p, 'net');
                });

                entities.forEach(e => {
                    if(e.x > net.x && e.y > netY) { e.type='bubble'; e.vy=-2; }
                });

                if(frameCount % 4 === 0) {
                    particles.push({
                        x: net.x+40, y: canvas.height,
                        vx: Math.random()*2, vy: -Math.random()*2,
                        life: 60, maxLife: 60, color: 'rgba(101,67,33,0.6)', size: 5+Math.random()*5
                    });
                }
                if(net.x < -600) net.active = false;
            }

            // Spawning
            const rateMult = Math.sqrt(gameSpeed);
            const spawn = CONFIG.SPAWN_RATES;
            
            // REDUCED PLANKTON SPAWN: Divide probability by gameSpeed to reduce clutter at high speeds
            if(Math.random() < (spawn.PLANKTON / gameSpeed) * (players.length>1?1.5:1)) {
                spawnEntity('plankton', canvas.width+20, Math.random()*(canvas.height-50));
            }
            
            if(Math.random() < spawn.PLASTIC * rateMult) spawnEntity('plastic', canvas.width+50, Math.random()*canvas.height);
            if(!net.active) {
                if(Math.random() < spawn.PREDATOR * rateMult) {
                    const type = Math.random()>0.5 ? 'predator_tuna' : 'predator_crab';
                    spawnEntity(type, canvas.width+50, 0);
                }
                if(Math.random() < spawn.SEAGRASS) spawnEntity('seagrass', canvas.width+50, canvas.height);
            }

            // Entity Update
            for(let i=entities.length-1; i>=0; i--) {
                const e = entities[i];
                e.x += e.vx; e.y += e.vy;
                if(e.type === 'plastic') e.rotation = (e.rotation||0) + 0.02;
                if(e.x < -100) { entities.splice(i,1); continue; }

                players.forEach(p => {
                    if(p.isDead) return;
                    if(Math.hypot(p.x-e.x, p.y-e.y) < p.radius + e.width/2) {
                        if(e.type==='plankton') {
                            p.energy = Math.min(p.maxEnergy, p.energy+15);
                            spawnExplosion(e.x, e.y, COLORS.plankton, 3);
                            entities.splice(i,1);
                        } else if(e.type==='plastic') {
                            p.energy -= 15; shake=5; p.vx = -5;
                            spawnExplosion(e.x, e.y, COLORS.plastic, 8);
                            entities.splice(i,1);
                        } else if(e.type.startsWith('predator')) {
                            if(!p.isCamouflaged) killPlayer(p, 'predator');
                        }
                    }
                });
            }

            // Particle Update
            for(let i=particles.length-1; i>=0; i--) {
                const p = particles[i];
                p.x += p.vx; p.y += p.vy;
                p.vx *= 0.95; p.vy *= 0.95;
                p.life--;
                if(p.life <= 0) particles.splice(i,1);
            }

            if(shake > 0) shake *= 0.9;

            // --- DRAW ---
            ctx.clearRect(0,0,canvas.width, canvas.height);
            ctx.save();
            if(shake > 0.5) ctx.translate((Math.random()-0.5)*shake, (Math.random()-0.5)*shake);

            // Water Gradient
            const grad = ctx.createLinearGradient(0, 0, 0, canvas.height);
            grad.addColorStop(0, COLORS.waterTop);
            grad.addColorStop(1, COLORS.waterBottom);
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Warning Flash
            if(net.warningPhase && Math.floor(Date.now()/150)%2===0) {
                ctx.fillStyle = 'rgba(239, 68, 68, 0.15)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }

            // Entities
            entities.forEach(e => {
                ctx.save();
                ctx.translate(e.x, e.y);
                if(e.type==='seagrass') {
                    const sway = Math.sin(Date.now()/200 + e.id*10)*15;
                    ctx.beginPath(); ctx.moveTo(0,0); ctx.quadraticCurveTo(sway, -e.height/2, sway/2, -e.height);
                    ctx.strokeStyle = '#15803d'; ctx.lineWidth = e.width/2; ctx.lineCap = 'round'; ctx.stroke();
                } else if(e.type==='plankton') {
                    const pulse = 1 + Math.sin(Date.now()/100)*0.2;
                    ctx.fillStyle = COLORS.plankton;
                    ctx.beginPath(); ctx.arc(0,0, e.width/2, 0, Math.PI*2); ctx.fill();
                    ctx.globalAlpha=0.5; ctx.beginPath(); ctx.arc(0,0, e.width/2*pulse*1.5, 0, Math.PI*2); ctx.fill();
                } else if(e.type==='plastic') {
                    ctx.rotate(e.rotation);
                    ctx.fillStyle = COLORS.plastic;
                    ctx.beginPath(); ctx.moveTo(-6,-10); ctx.lineTo(6,-10); ctx.lineTo(8,10); ctx.lineTo(-8,10); ctx.fill();
                    ctx.fillStyle = 'rgba(255,255,255,0.5)'; ctx.fillRect(-4,-5,3,12);
                } else if(e.type==='predator_tuna') {
                    ctx.fillStyle = '#3b82f6';
                    ctx.beginPath(); ctx.ellipse(0,0, e.width/2, e.height/1.8, 0,0,Math.PI*2); ctx.fill();
                    ctx.beginPath(); ctx.moveTo(e.width/2,0); ctx.lineTo(e.width/2+12, -8); ctx.lineTo(e.width/2+12, 8); ctx.fill();
                    ctx.fillStyle = '#93c5fd'; ctx.beginPath(); ctx.ellipse(5,5, 8, 4, 0.5, 0, Math.PI*2); ctx.fill();
                    ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(-15, -5, 6, 0, Math.PI*2); ctx.fill();
                    ctx.fillStyle = '#0f172a'; ctx.beginPath(); ctx.arc(-17, -5, 2.5, 0, Math.PI*2); ctx.fill();
                } else if(e.type==='predator_crab') {
                    const tick = Date.now()/150;
                    ctx.fillStyle = '#b91c1c'; ctx.strokeStyle = '#b91c1c'; ctx.lineWidth = 4; ctx.lineCap = 'round'; ctx.lineJoin = 'round';
                    for(let side=-1; side<=1; side+=2) {
                        for(let l=0; l<3; l++) {
                            ctx.beginPath(); ctx.moveTo(side*10, 2); ctx.lineTo(side*(20 + l*5), 8); ctx.lineTo(side*(25 + l*8), 16 + Math.sin(tick+l)*4); ctx.stroke();
                        }
                        ctx.beginPath(); ctx.moveTo(side*10, -5); ctx.lineTo(side*22, -18); ctx.stroke();
                        ctx.save(); ctx.translate(side*25, -22); ctx.rotate(side * (Math.PI/4 + Math.sin(tick)*0.2));
                        ctx.fillStyle = '#b91c1c'; ctx.beginPath(); ctx.ellipse(0,0, 8, 5, 0,0,Math.PI*2); ctx.fill();
                        ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(2,-8); ctx.lineTo(-4,0); ctx.fill();
                        ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(4,-4); ctx.lineTo(-2,2); ctx.fill(); ctx.restore();
                    }
                    ctx.fillStyle = '#b91c1c'; ctx.beginPath(); ctx.ellipse(0,0, 18, 12, 0,0,Math.PI*2); ctx.fill();
                    ctx.strokeStyle = '#b91c1c'; ctx.lineWidth=2; ctx.beginPath(); ctx.moveTo(-6,-10); ctx.lineTo(-6,-16); ctx.stroke(); ctx.beginPath(); ctx.moveTo(6,-10); ctx.lineTo(6,-16); ctx.stroke();
                    ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(-6,-16,3,0,Math.PI*2); ctx.arc(6,-16,3,0,Math.PI*2); ctx.fill();
                    ctx.fillStyle = 'black'; ctx.beginPath(); ctx.arc(-6,-16,1.5,0,Math.PI*2); ctx.arc(6,-16,1.5,0,Math.PI*2); ctx.fill();
                } else if(e.type==='bubble') {
                    ctx.strokeStyle='rgba(255,255,255,0.5)'; ctx.lineWidth=2; ctx.beginPath(); ctx.arc(0,0,e.width/2,0,Math.PI*2); ctx.stroke();
                }
                ctx.restore();
            });

            // Net
            if(net.active) {
                const netH = canvas.height * CONFIG.TRAWL_HEIGHT_RATIO;
                const netY = canvas.height - netH;
                const mouth = net.x;
                const bag = net.x + 400;

                ctx.save();
                ctx.beginPath(); ctx.moveTo(mouth, netY); ctx.lineTo(bag, netY + (canvas.height-netY)*0.4); ctx.lineTo(bag, canvas.height - (canvas.height-netY)*0.4); ctx.lineTo(mouth, canvas.height); ctx.closePath();
                const g = ctx.createLinearGradient(mouth, 0, bag, 0);
                g.addColorStop(0, 'rgba(50,60,70,0.8)'); g.addColorStop(1, 'rgba(20,30,40,0.95)');
                ctx.fillStyle = g; ctx.fill();
                ctx.strokeStyle = 'rgba(200,210,220,0.2)'; ctx.lineWidth=1; ctx.beginPath();
                for(let i=0; i<=5; i++) {
                    const y1 = netY + ((canvas.height-netY)/5)*i;
                    const y2 = (netY+(canvas.height-netY)*0.4) + ((canvas.height-(canvas.height-netY)*0.8)/5)*i;
                    ctx.moveTo(mouth, y1); ctx.lineTo(bag, y2);
                }
                ctx.stroke();
                ctx.fillStyle = '#f97316'; for(let i=0; i<5; i++) { ctx.beginPath(); ctx.arc(mouth, netY+(i*15), 6, 0, Math.PI*2); ctx.fill(); }
                ctx.fillStyle = '#333'; ctx.fillRect(mouth-20, canvas.height-100, 20, 100);
                ctx.restore();
            }

            // Particles
            particles.forEach(p => {
                ctx.globalAlpha = p.life/p.maxLife; ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI*2); ctx.fill();
            });
            ctx.globalAlpha = 1;

            // Players
            players.forEach(p => {
                if(p.opacity <= 0) return;
                ctx.save();
                ctx.translate(p.x, p.y);
                ctx.rotate(p.rotation);
                ctx.globalAlpha = p.opacity;

                if(p.isCamouflaged && !p.isAnchored) { ctx.shadowColor=p.config.color; ctx.shadowBlur=15; }
                
                // Name Tag
                if(!p.isDead) {
                    ctx.save(); ctx.rotate(-p.rotation); ctx.fillStyle='white'; ctx.font='bold 12px Fredoka'; ctx.textAlign='center'; ctx.fillText(p.config.name, 0, -35); ctx.restore();
                }

                // --- SEAHORSE DRAWING ---
                const tick = Date.now() / 150;
                const tailSway = Math.sin(tick) * 3; 
                const finFlutter = Math.sin(tick * 3) * 2;

                ctx.beginPath(); ctx.moveTo(-8, 2); ctx.quadraticCurveTo(-24 + finFlutter, 10, -8, 18);
                ctx.fillStyle = p.config.color; ctx.fill();
                
                ctx.beginPath(); ctx.moveTo(0, 18); ctx.bezierCurveTo(5, 40, 25 + tailSway, 38, 18 + tailSway, 20);
                ctx.lineWidth = 8; ctx.strokeStyle = p.config.color; ctx.lineCap = 'round'; ctx.stroke();
                
                ctx.fillStyle = p.config.color; ctx.beginPath(); ctx.ellipse(0, 10, 12, 18, 0, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = COLORS.seahorseBelly; ctx.beginPath(); ctx.ellipse(4, 10, 7, 12, 0, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = p.config.color; ctx.beginPath(); ctx.arc(0, -15, 14, 0, Math.PI * 2); ctx.fill();
                
                ctx.save(); ctx.translate(12, -13); ctx.rotate(0.2);
                ctx.fillStyle = p.config.color; ctx.beginPath(); ctx.roundRect(0, -5, 16, 10, 5); ctx.fill(); ctx.restore();

                ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(4, -16, 6.5, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = '#0f172a'; ctx.beginPath(); ctx.arc(6, -16, 3.5, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = 'rgba(244, 114, 182, 0.5)'; ctx.beginPath(); ctx.arc(0, -8, 4, 0, Math.PI * 2); ctx.fill();

                ctx.restore();
            });

            ctx.restore();

            // Update DOM HUD (Throttled slightly naturally by loop)
            if(frameCount % 5 === 0 && players.length > 0) {
                if(players[0]) {
                    const el = document.getElementById('p1-energy-bar');
                    const txt = document.getElementById('p1-energy-text');
                    if(el && txt) {
                        el.style.width = Math.max(0, players[0].energy) + "%";
                        el.className = `h-full transition-all duration-300 ${players[0].energy<30?'bg-red-500':'bg-yellow-400'}`;
                        txt.textContent = Math.ceil(Math.max(0, players[0].energy)) + "%";
                        document.getElementById('score-single').textContent = Math.floor(players[0].distance) + "m";
                        document.getElementById('score-p1').textContent = Math.floor(players[0].distance) + "m";
                    }
                }
                if(players[1]) {
                    const el = document.getElementById('p2-energy-bar');
                    const txt = document.getElementById('p2-energy-text');
                    if(el && txt) {
                        el.style.width = Math.max(0, players[1].energy) + "%";
                        el.className = `h-full transition-all duration-300 ${players[1].energy<30?'bg-red-500':'bg-cyan-400'}`;
                        txt.textContent = Math.ceil(Math.max(0, players[1].energy)) + "%";
                        document.getElementById('score-p2').textContent = Math.floor(players[1].distance) + "m";
                    }
                }
            }

            requestID = requestAnimationFrame(loop);
        }
    </script>
</body>
</html>
