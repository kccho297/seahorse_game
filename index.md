<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Seahorse Survival: The Great Escape</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

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
                    }
                }
            }
        }
    </script>

    <style>
        body {
            background-color: #0f172a;
            color: #fff;
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
            display: none; /* Hidden by default */
            z-index: 10;
        }
        .screen.active {
            display: flex;
        }
        canvas {
            display: block;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>

    <div id="hud-playing" class="absolute top-0 left-0 w-full p-4 flex justify-between items-start z-10 pointer-events-none hidden">
        <div class="flex flex-col gap-4 w-64">
            <div id="p1-hud" class="hidden">
                <div class="flex justify-between text-white text-sm font-ui font-bold drop-shadow-md">
                    <span class="text-yellow-400">P1 (WASD)</span>
                    <span id="p1-energy-text">100%</span>
                </div>
                <div class="h-3 bg-slate-800/50 rounded-full border border-white/20 overflow-hidden backdrop-blur">
                    <div id="p1-energy-bar" class="h-full bg-yellow-400 transition-all duration-300" style="width: 100%"></div>
                </div>
            </div>
            <div id="p2-hud" class="hidden">
                <div class="flex justify-between text-white text-sm font-ui font-bold drop-shadow-md">
                    <span class="text-cyan-400">P2 (ARROWS)</span>
                    <span id="p2-energy-text">100%</span>
                </div>
                <div class="h-3 bg-slate-800/50 rounded-full border border-white/20 overflow-hidden backdrop-blur">
                    <div id="p2-energy-bar" class="h-full bg-cyan-400 transition-all duration-300" style="width: 100%"></div>
                </div>
            </div>
        </div>
        <div class="text-center">
            <div id="score-single" class="hidden text-5xl text-white font-bold drop-shadow-[0_4px_4px_rgba(0,0,0,0.8)] font-game">0m</div>
            <div id="score-multi" class="hidden flex gap-12 bg-slate-900/50 p-4 rounded-2xl backdrop-blur-md border border-white/10 font-ui">
                <div class="flex flex-col items-center">
                    <span class="text-yellow-400 text-sm font-bold uppercase">Player 1</span>
                    <span id="score-p1" class="text-4xl text-white font-bold">0m</span>
                </div>
                <div class="w-px bg-white/20"></div>
                <div class="flex flex-col items-center">
                    <span class="text-cyan-400 text-sm font-bold uppercase">Player 2</span>
                    <span id="score-p2" class="text-4xl text-white font-bold">0m</span>
                </div>
            </div>
        </div>
    </div>

    <div id="hud-warning" class="absolute top-1/4 left-0 w-full flex-col items-center justify-center animate-pulse-fast z-20 pointer-events-none hidden">
        <div class="bg-red-600 text-white px-8 py-4 rounded-xl border-4 border-red-800 shadow-2xl flex flex-col items-center">
            <div class="flex items-center gap-4 text-4xl font-bold uppercase tracking-widest font-game">
                ⚠ ️ TRAWLER DETECTED ⚠ ️
            </div>
            <div class="text-center text-xl mt-2 font-ui font-bold">SWIM UP!</div>
        </div>
    </div>

    <div id="screen-landing" class="screen active flex-col items-center justify-center bg-slate-900 z-50">
        <div class="text-center animate-in fade-in zoom-in duration-500">
            <div class="flex justify-center mb-6">
                <div class="bg-white/10 p-6 rounded-full shadow-[0_0_50px_rgba(56,189,248,0.3)]">
                    <svg width="140" height="140" viewBox="-40 -40 80 80" fill="none" class="animate-bounce">
                        <path d="M-10 1 Q -25 5, -10 15" stroke="#fbbf24" stroke-width="1" fill="#fcd34d" />
      
                       <path d="M0 20 C 1 30, 20 50, 15 30 S 2 30, 5 15"stroke="#fbbf24" stroke-width="8" stroke-linecap="round" fill="none" />
<ellipse cx="0" cy="10" rx="12" ry="18" fill="#fbbf24" />
<ellipse cx="3" cy="10" rx="8"ry="12" fill="#fcd34d" />
                        <circle cx="0" cy="-15" r="14" fill="#fbbf24" />
                        
                        <rect x="10" y="-19" width="18" height="9" rx="4.5" transform="rotate(12 10 -15)" fill="#fbbf24" />
                        
                        <circle cx="4" cy="-16" r="6.5" fill="white" />
                        <circle cx="7" cy="-16" r="3.5" fill="#0f172a" />
                        <circle cx="0" cy="-8" r="3.5" fill="#f472b6" fill-opacity="0.6" />
                    </svg>
                </div>
            </div>
            <h1 class="text-7xl text-transparent bg-clip-text bg-gradient-to-b from-yellow-300 to-orange-500 font-bold drop-shadow-lg mb-4 font-game">Seahorse Survival</h1>
            <p class="text-xl text-blue-200 font-ui mb-12 tracking-widest uppercase">The Great Escape</p>

            <div class="flex gap-8 justify-center font-ui">
                <button onclick="goToMenu(1)" class="group bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-yellow-400 text-white w-64 h-40 rounded-3xl flex flex-col items-center justify-center gap-4 transition-all hover:-translate-y-2 shadow-xl cursor-pointer">
                    <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-slate-400 group-hover:text-yellow-400"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                    <span class="text-2xl font-bold">1 Player</span>
                </button>
                <button onclick="goToMenu(2)" class="group bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-cyan-400 text-white w-64 h-40 rounded-3xl flex flex-col items-center justify-center gap-4 transition-all hover:-translate-y-2 shadow-xl cursor-pointer">
                    <div class="flex gap-2 text-slate-400 group-hover:text-cyan-400">
                        <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                        <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
                    </div>
                    <span class="text-2xl font-bold">2 Players</span>
                </button>
            </div>
        </div>
    </div>

    <div id="screen-menu" class="screen items-center justify-center bg-ocean-900/90 backdrop-blur-sm z-50">
        <div class="max-w-md w-full bg-white/10 border border-white/20 p-8 rounded-3xl shadow-2xl text-center backdrop-blur-xl font-ui">
            <div class="flex justify-center mb-4 relative">
                <button onclick="goToLanding()" class="absolute top-2 left-0 text-white/50 hover:text-white flex items-center gap-2 text-sm uppercase font-bold tracking-widest">
                    &larr; Back
                </button>
                <div class="bg-yellow-400 p-4 rounded-full shadow-lg mt-8">
                    <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="#0f172a" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L15 8L21 9L17 14L18 20L12 17L6 20L7 14L3 9L9 8L12 2Z" stroke-linejoin="round"/></svg>
                </div>
            </div>
            <h2 id="menu-title" class="text-4xl text-white mb-2 font-bold font-game">Solo Adventure</h2>
            <p class="text-blue-200 mb-8 text-sm">Escape the net. Avoid plastic. Survive.</p>

            <div id="controls-grid" class="grid grid-cols-1 gap-4 mb-8">
                <div class="bg-black/30 p-4 rounded-xl text-left">
                    <h3 class="text-yellow-400 font-bold mb-2 uppercase text-xs tracking-wider">Player 1</h3>
                    <div class="text-xs text-slate-300 space-y-2">
                        <p><strong class="text-white bg-slate-700 px-1 rounded">WASD</strong> to Swim</p>
                        <p><strong class="text-white bg-slate-700 px-1 rounded">F / Shift</strong> to Camouflage</p>
                    </div>
                </div>
                <div id="p2-controls" class="bg-black/30 p-4 rounded-xl text-left hidden">
                    <h3 class="text-cyan-400 font-bold mb-2 uppercase text-xs tracking-wider">Player 2</h3>
                    <div class="text-xs text-slate-300 space-y-2">
                        <p><strong class="text-white bg-slate-700 px-1 rounded">Arrows</strong> to Swim</p>
                        <p><strong class="text-white bg-slate-700 px-1 rounded">Enter</strong> to Camouflage</p>
                    </div>
                </div>
            </div>
            <div class="bg-blue-500/20 p-4 rounded-xl mb-8 text-left border border-blue-500/30">
                <div class="text-xs font-bold text-blue-300 uppercase mb-1 flex items-center gap-2"> ⚡ Power: Camouflage</div>
                <p class="text-xs text-slate-200">Hold your Action Key to hide from predators! <strong>Warning:</strong> Using it in open water drains energy.
                    Hide in seagrass to conserve strength.</p>
            </div>
            <button onclick="initGame()" class="w-full bg-gradient-to-r from-yellow-400 to-orange-500 hover:from-yellow-300 hover:to-orange-400 text-ocean-900 font-bold text-2xl py-4 rounded-xl shadow-lg hover:scale-105 transition-transform flex items-center justify-center gap-3 font-game cursor-pointer">
                START GAME
            </button>
        </div>
    </div>

    <div id="screen-gameover" class="screen items-center justify-center bg-red-900/90 backdrop-blur-md z-50">
        <div class="max-w-lg w-full bg-slate-900 border border-red-500/30 p-8 rounded-3xl shadow-2xl text-center">
            <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="#ef4444" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mx-auto mb-4 animate-pulse"><circle cx="9" cy="12" r="1"/><circle cx="15" cy="12" r="1"/><path d="M8 20v2h8v-2"/><path d="M12.5 17l.5-2"/><path d="M16 20a2 2 0 0 0 1.56-3.25 8 8 0 1 0-11.12 0A2 2 0 0 0 8 20"/></svg>
            <h2 class="text-4xl text-white font-bold mb-2 font-game">GAME OVER</h2>

            <div id="go-single" class="text-6xl font-bold text-yellow-400 mb-6 font-ui drop-shadow-lg hidden">0m</div>
            <div id="go-multi" class="bg-slate-800 p-4 rounded-xl mb-6 border border-white/10 hidden">
                <div class="flex items-center justify-center gap-2 mb-4 text-xl font-bold uppercase tracking-widest text-white">
                    Leaderboard
                </div>
                <div class="space-y-2 font-ui" id="leaderboard-content">
                    </div>
            </div>
            <div class="bg-slate-800 p-6 rounded-xl border-l-4 border-blue-400 mb-8 text-left shadow-lg">
                <h3 class="text-blue-400 font-bold uppercase text-xs tracking-widest mb-2 flex items-center gap-2">
                    Did You Know?
                </h3>
                <p id="death-fact" class="text-slate-200 font-ui leading-relaxed text-lg">
                    Seahorses are cool.
                </p>
            </div>
            <button onclick="goToLanding()" class="w-full bg-white hover:bg-slate-200 text-slate-900 font-bold text-xl py-4 rounded-xl shadow-lg flex items-center justify-center gap-3 transition-colors hover:scale-105 transform font-game cursor-pointer">
                PLAY AGAIN
            </button>
        </div>
    </div>

    <script>
        // --- DATA & CONFIG ---
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
            SPAWN_RATES: { PLANKTON: 0.1, PLASTIC: 0.02, PREDATOR: 0.015, SEAGRASS: 0.03 },
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
                "An estimated 37 million seahorses are caught as 'bycatch' in trawling nets every year.",
                "Trawling creates sediment plumes visible from space, choking marine life."
            ],
            plastic: [
                "Seahorses clutch onto drifting debris like cotton swabs, travelling miles into polluted waters.",
                "Microplastics block digestive systems, which is fatal for seahorses within hours.",
                "Ghost nets (abandoned gear) continue to trap and kill marine life for decades.",
                "80% of ocean plastic comes from land-based sources."
            ],
            starvation: [
                "Seahorses have no true stomach and teeth. They suck food like a vacuum cleaner.",
                "A seahorse must eat 30-50 times a day just to stay alive.",
                "When seagrass beds disappear due to pollution, plankton populations collapse.",
                "Using Camouflage in open water burns energy! Hide in seagrass to conserve strength."
            ],
            predator: [
                "Seahorses are masters of camouflage, changing color to match their surroundings.",
                "Seahorses are poor swimmers and are easy snacks for crabs, tuna, and rays in open water.",
                "Use your Camouflage power (Shift/Enter) to hide from predators!",
                "Despite bony armor, seahorses are preyed upon heavily."
            ]
        };

        const PLAYERS_INIT = [
            {
                id: 1, color: '#fbbf24', name: "P1",
                controls: { up: ['KeyW'], down: ['KeyS'], left: ['KeyA'], right: ['KeyD'], anchor: ['KeyF', 'ShiftLeft'] },
                yOffset: 0.4
            },
            {
                id: 2, color: '#22d3ee', name: "P2",
                controls: { up: ['ArrowUp'], down: ['ArrowDown'], left: ['ArrowLeft'], right: ['ArrowRight'], anchor: ['Enter', 'ControlRight'] },
                yOffset: 0.6
            }
        ];

        // --- STATE ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let width, height;
        let requestID;
        let frameCount = 0;
        let gameSpeed = 1.0;
        let shake = 0;
        let selectedPlayerCount = 1;

        const keys = new Set();
        const players = [];
        const entities = [];
        const particles = [];
        const net = { active: false, x: -500, speed: 0, warningPhase: false };

        // --- INITIALIZATION ---
        function resize() {
            width = window.innerWidth;
            height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;
        }
        window.addEventListener('resize', resize);
        resize();
        window.addEventListener('keydown', e => keys.add(e.code));
        window.addEventListener('keyup', e => keys.delete(e.code));

        // --- NAVIGATION ---
        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(el => el.classList.remove('active'));
            if(id) document.getElementById(id).classList.add('active');
            
            // Toggle HUDs
            document.getElementById('hud-playing').classList.toggle('hidden', id !== null);
            document.getElementById('hud-warning').classList.add('hidden'); // reset warning
        }

        function goToLanding() {
            showScreen('screen-landing');
        }

        function goToMenu(count) {
            selectedPlayerCount = count;
            document.getElementById('menu-title').textContent = count === 2 ? "Co-op Mode" : "Solo Adventure";
            document.getElementById('controls-grid').classList.toggle('grid-cols-2', count === 2);
            document.getElementById('p2-controls').classList.toggle('hidden', count !== 2);
            showScreen('screen-menu');
        }

        function initGame() {
            // Reset State
            showScreen(null); // Hide all overlay screens, show canvas HUD
            document.getElementById('hud-playing').classList.remove('hidden');
            
            // Setup UI for players
            document.getElementById('p1-hud').classList.remove('hidden');
            document.getElementById('p2-hud').classList.toggle('hidden', selectedPlayerCount < 2);
            document.getElementById('score-single').classList.toggle('hidden', selectedPlayerCount === 2);
            document.getElementById('score-multi').classList.toggle('hidden', selectedPlayerCount === 1);

            // Initialize objects
            players.length = 0;
            entities.length = 0;
            particles.length = 0;
            keys.clear();
            frameCount = 0;
            gameSpeed = 1.0;
            shake = 0;
            net.active = false;
            net.warningPhase = false;
            net.x = -500;

            for(let i=0; i<selectedPlayerCount; i++) {
                const conf = PLAYERS_INIT[i];
                players.push({
                    id: conf.id,
                    color: conf.color,
                    name: conf.name,
                    x: 100,
                    y: height * conf.yOffset,
                    vx: 0, vy: 0,
                    radius: CONFIG.PLAYER_RADIUS,
                    energy: 100,
                    maxEnergy: 100,
                    distance: 0,
                    isAnchored: false,
                    isCamouflaged: false,
                    rotation: 0,
                    opacity: 1,
                    isDead: false,
                    controls: conf.controls
                });
            }
            if(requestID) cancelAnimationFrame(requestID);
            loop();
        }

        function gameOver(reason) {
            cancelAnimationFrame(requestID);
            
            // Set Fact
            const list = FACTS[reason] || FACTS.starvation;
            const fact = list[Math.floor(Math.random() * list.length)];
            document.getElementById('death-fact').textContent = fact;

            // Set Scores
            if(selectedPlayerCount === 1) {
                document.getElementById('go-single').textContent = Math.floor(players[0].distance) + "m";
                document.getElementById('go-single').classList.remove('hidden');
                document.getElementById('go-multi').classList.add('hidden');
            } else {
                document.getElementById('go-single').classList.add('hidden');
                document.getElementById('go-multi').classList.remove('hidden');
                
                const p1Dist = Math.floor(players[0].distance);
                const p2Dist = Math.floor(players[1].distance);
                const p1Win = p1Dist >= p2Dist;

                const html = `
                    <div class="flex justify-between items-center p-3 rounded-lg ${p1Win ? 'bg-yellow-400/20 border border-yellow-400/50' : 'bg-white/5'}">
                        <span class="font-bold ${p1Win ? 'text-yellow-400' : 'text-slate-400'}">Player 1</span>
                        <span class="text-2xl font-bold text-white">${p1Dist}m</span>
                    </div>
                    <div class="flex justify-between items-center p-3 rounded-lg ${!p1Win ? 'bg-cyan-400/20 border border-cyan-400/50' : 'bg-white/5'}">
                        <span class="font-bold ${!p1Win ? 'text-cyan-400' : 'text-slate-400'}">Player 2</span>
                        <span class="text-2xl font-bold text-white">${p2Dist}m</span>
                    </div>
                `;
                document.getElementById('leaderboard-content').innerHTML = html;
            }
            
            showScreen('screen-gameover');
        }

        // --- GAME HELPERS ---
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
            
            if (reason === 'net') { spawnExplosion(p.x, p.y, p.color, 50); spawnExplosion(p.x, p.y, '#fff', 30); }
            else if (reason === 'predator') spawnExplosion(p.x, p.y, COLORS.blood, 60);
            else if (reason === 'plastic') spawnExplosion(p.x, p.y, COLORS.plastic, 30);
            else spawnExplosion(p.x, p.y, COLORS.bubble, 30);

            // Check All Dead
            if(players.every(pl => pl.isDead)) {
                setTimeout(() => gameOver(reason), 1000);
            }
        }

        function spawnEntity(type, x, y) {
            let w=10, h=10, vx=0, vy=0;
            const speedMult = gameSpeed;

            switch(type) {
                case 'plankton': w=10; h=10; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult; vy=(Math.random()-0.5); break;
                case 'plastic': w=25; h=25; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult*1.2; vy=(Math.random()-0.5)*0.5; break;
                case 'predator_tuna': w=60; h=40; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult*2.8; break;
                case 'predator_crab': w=60; h=40; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult; y=height-50; break;
                case 'seagrass': w=20; h=80+Math.random()*80; vx = -CONFIG.BASE_SCROLL_SPEED*speedMult; y=height; break;
            }
            entities.push({ id: Math.random(), type, x, y, width: w, height: h, vx, vy, rotation: 0 });
        }

        // --- GAME LOOP ---
        function loop() {
            frameCount++;

            // 1. UPDATE LOGIC
            if(gameSpeed < CONFIG.MAX_SPEED_MULTIPLIER) gameSpeed += CONFIG.DIFFICULTY_RAMP;
            const scrollSpeed = CONFIG.BASE_SCROLL_SPEED * gameSpeed;
            const distStep = scrollSpeed * 0.05;

            // Players
            players.forEach(p => {
                if(p.isDead) return;
                p.distance += distStep;

                // Input
                let dx=0, dy=0, action=false;
                const ctrl = p.controls;
                if(ctrl.up.some(k => keys.has(k))) dy-=1;
                if(ctrl.down.some(k => keys.has(k))) dy+=1;
                if(ctrl.left.some(k => keys.has(k))) dx-=1;
                if(ctrl.right.some(k => keys.has(k))) dx+=1;
                if(ctrl.anchor.some(k => keys.has(k))) action=true;

                if(dx!==0 && dy!==0) { dx*=0.707; dy*=0.707; }

                // Camouflage Logic
                let nearSeagrass = entities.some(e => e.type === 'seagrass' && Math.hypot(p.x-e.x, p.y-(e.y-e.height/2)) < 80);
                
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

                // Bounds
                p.x = Math.max(p.radius, Math.min(width-p.radius, p.x));
                p.y = Math.max(p.radius, Math.min(height-p.radius, p.y));

                // Starvation
                if(p.energy <= 0) killPlayer(p, 'starvation');

                // Turbulence
                if(net.warningPhase || net.active) {
                    p.x += (Math.random()-0.5)*2;
                    p.y += (Math.random()-0.5)*2;
                    if(net.active && net.x - p.x > 0 && net.x - p.x < 300) p.vx += 0.1;
                }
            });

            // Net Logic
            const cycle = frameCount % CONFIG.TRAWL_INTERVAL;
            const warningEl = document.getElementById('hud-warning');

            if(cycle === CONFIG.TRAWL_INTERVAL - CONFIG.TRAWL_WARNING_DURATION) {
                net.warningPhase = true;
                shake = 5;
                warningEl.classList.remove('hidden');
            }
            if(cycle === 0 && frameCount > 0) {
                net.warningPhase = false;
                net.active = true;
                net.x = width + 200;
                net.speed = CONFIG.TRAWL_SPEED_BASE * (1 + (gameSpeed-1)*0.5);
                warningEl.classList.add('hidden');
            }
            if(net.active) {
                net.x -= net.speed;
                const netH = height * CONFIG.TRAWL_HEIGHT_RATIO;
                const netY = height - netH;

                // Collisions
                players.forEach(p => {
                    if(!p.isDead && p.x > net.x && p.x < net.x+150 && p.y > netY) killPlayer(p, 'net');
                });

                // Clear entities
                entities.forEach(e => {
                    if(e.x > net.x && e.y > netY) { e.type='bubble'; e.vy=-2; }
                });
                // Mud
                if(frameCount % 4 === 0) {
                    particles.push({
                        id: Math.random(), x: net.x+40, y: height,
                        vx: Math.random()*2, vy: -Math.random()*2,
                        life: 60, maxLife: 60, color: 'rgba(101,67,33,0.6)', size: 5+Math.random()*5
                    });
                }
                if(net.x < -600) net.active = false;
            }

            // Spawning
            const rateMult = Math.sqrt(gameSpeed);
            const spawn = CONFIG.SPAWN_RATES;
            if(Math.random() < spawn.PLANKTON * (players.length>1?1.5:1)) spawnEntity('plankton', width+20, Math.random()*(height-50));
            if(Math.random() < spawn.PLASTIC * rateMult) spawnEntity('plastic', width+50, Math.random()*height);
            if(!net.active) {
                if(Math.random() < spawn.PREDATOR * rateMult) {
                    const type = Math.random()>0.5 ? 'predator_tuna' : 'predator_crab';
                    const y = type==='predator_tuna' ? Math.random()*(height-150) : height-30;
                    spawnEntity(type, width+50, y);
                }
                if(Math.random() < spawn.SEAGRASS) spawnEntity('seagrass', width+50, height);
            }

            // Entities Update
            for(let i=entities.length-1; i>=0; i--) {
                const e = entities[i];
                e.x += e.vx; e.y += e.vy;
                if(e.type === 'plastic') e.rotation = (e.rotation||0) + 0.02;
                if(e.x < -100) { entities.splice(i,1); continue; }

                // Player Collision
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

            // Particles Update
            for(let i=particles.length-1; i>=0; i--) {
                const p = particles[i];
                p.x += p.vx; p.y += p.vy;
                p.vx *= 0.95; p.vy *= 0.95;
                p.life--;
                if(p.life <= 0) particles.splice(i,1);
            }

            if(shake > 0) shake *= 0.9;

            // UI Update
            if(frameCount % 5 === 0) {
                if(players[0]) {
                    document.getElementById('p1-energy-bar').style.width = Math.max(0, players[0].energy) + "%";
                    document.getElementById('p1-energy-bar').className = `h-full transition-all duration-300 ${players[0].energy<30?'bg-red-500':'bg-yellow-400'}`;
                    document.getElementById('p1-energy-text').textContent = Math.ceil(Math.max(0, players[0].energy)) + "%";
                    const d = Math.floor(players[0].distance);
                    document.getElementById('score-single').textContent = d + "m";
                    document.getElementById('score-p1').textContent = d + "m";
                }
                if(players[1]) {
                    document.getElementById('p2-energy-bar').style.width = Math.max(0, players[1].energy) + "%";
                    document.getElementById('p2-energy-bar').className = `h-full transition-all duration-300 ${players[1].energy<30?'bg-red-500':'bg-cyan-400'}`;
                    document.getElementById('p2-energy-text').textContent = Math.ceil(Math.max(0, players[1].energy)) + "%";
                    document.getElementById('score-p2').textContent = Math.floor(players[1].distance) + "m";
                }
            }

            // 2. DRAW
            ctx.save();
            if(shake > 0.5) ctx.translate((Math.random()-0.5)*shake, (Math.random()-0.5)*shake);

            // Water
            const grad = ctx.createLinearGradient(0, 0, 0, height);
            grad.addColorStop(0, COLORS.waterTop);
            grad.addColorStop(1, COLORS.waterBottom);
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, width, height);

            if(net.warningPhase && Math.floor(Date.now()/150)%2===0) {
                ctx.fillStyle = 'rgba(239, 68, 68, 0.15)';
                ctx.fillRect(0, 0, width, height);
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
                    ctx.fillStyle = '#b91c1c';
                    ctx.strokeStyle = '#b91c1c';
                    ctx.lineWidth = 4;
                    ctx.lineCap = 'round';
                    ctx.lineJoin = 'round';
                    
                    for(let side=-1; side<=1; side+=2) {
                        for(let l=0; l<3; l++) {
                            ctx.beginPath();
                            ctx.moveTo(side*10, 2);
                            ctx.lineTo(side*(20 + l*5), 8);
                            ctx.lineTo(side*(25 + l*8), 16 + Math.sin(tick+l)*4);
                            ctx.stroke();
                        }
                        ctx.beginPath();
                        ctx.moveTo(side*10, -5);
                        ctx.lineTo(side*22, -18);
                        ctx.stroke();

                        ctx.save();
                        ctx.translate(side*25, -22);
                        ctx.rotate(side * (Math.PI/4 + Math.sin(tick)*0.2));
                        ctx.fillStyle = '#b91c1c';
                        ctx.beginPath(); ctx.ellipse(0,0, 8, 5, 0,0,Math.PI*2); ctx.fill();
                        ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(2,-8); ctx.lineTo(-4,0); ctx.fill();
                        ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(4,-4); ctx.lineTo(-2,2); ctx.fill();
                        ctx.restore();
                    }
                    ctx.fillStyle = '#b91c1c';
                    ctx.beginPath(); ctx.ellipse(0,0, 18, 12, 0,0,Math.PI*2); ctx.fill();

                    ctx.strokeStyle = '#b91c1c'; ctx.lineWidth=2;
                    ctx.beginPath(); ctx.moveTo(-6,-10); ctx.lineTo(-6,-16); ctx.stroke();
                    ctx.beginPath(); ctx.moveTo(6,-10); ctx.lineTo(6,-16); ctx.stroke();
                    
                    ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(-6,-16,3,0,Math.PI*2); ctx.arc(6,-16,3,0,Math.PI*2); ctx.fill();
                    ctx.fillStyle = 'black'; ctx.beginPath(); ctx.arc(-6,-16,1.5,0,Math.PI*2); ctx.arc(6,-16,1.5,0,Math.PI*2); ctx.fill();
                } else if(e.type==='bubble') {
                    ctx.strokeStyle='rgba(255,255,255,0.5)'; ctx.lineWidth=2; ctx.beginPath(); ctx.arc(0,0,e.width/2,0,Math.PI*2); ctx.stroke();
                }
                ctx.restore();
            });

            // Net
            if(net.active) {
                const netH = height * CONFIG.TRAWL_HEIGHT_RATIO;
                const netY = height - netH;
                const mouth = net.x;
                const bag = net.x + 400;

                ctx.save();
                ctx.beginPath(); ctx.moveTo(mouth, netY); ctx.lineTo(bag, netY + (height-netY)*0.4); ctx.lineTo(bag, height - (height-netY)*0.4); ctx.lineTo(mouth, height); ctx.closePath();
                const g = ctx.createLinearGradient(mouth, 0, bag, 0);
                g.addColorStop(0, 'rgba(50,60,70,0.8)'); g.addColorStop(1, 'rgba(20,30,40,0.95)');
                ctx.fillStyle = g; ctx.fill();

                // Ropes (simplified)
                ctx.strokeStyle = 'rgba(200,210,220,0.2)'; ctx.lineWidth=1; ctx.beginPath();
                for(let i=0; i<=5; i++) {
                    const y1 = netY + ((height-netY)/5)*i;
                    const y2 = (netY+(height-netY)*0.4) + ((height-(height-netY)*0.8)/5)*i;
                    ctx.moveTo(mouth, y1); ctx.lineTo(bag, y2);
                }
                ctx.stroke();

                // Doors & Floats
                ctx.fillStyle = '#f97316'; for(let i=0; i<5; i++) { ctx.beginPath(); ctx.arc(mouth, netY+(i*15), 6, 0, Math.PI*2); ctx.fill(); }
                ctx.fillStyle = '#333'; ctx.fillRect(mouth-20, height-100, 20, 100);
                ctx.restore();
            }

            // Particles
            particles.forEach(p => {
                ctx.globalAlpha = p.life/p.maxLife; ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI*2); ctx.fill();
            });
            ctx.globalAlpha = 1;

            // Players (UPDATED SEAHORSE DRAWING)
            players.forEach(p => {
                if(p.opacity <= 0) return;
                ctx.save();
                ctx.translate(p.x, p.y);
                ctx.rotate(p.rotation);
                ctx.globalAlpha = p.opacity;

                if(p.isCamouflaged && !p.isAnchored) { ctx.shadowColor=p.color; ctx.shadowBlur=15; }

                // Labels
                if(!p.isDead && p.name) {
                    ctx.save(); ctx.rotate(-p.rotation); ctx.fillStyle='white'; ctx.font='10px Fredoka'; ctx.textAlign='center'; ctx.fillText(p.name, 0, -35); ctx.restore();
                }

                // --- DRAW SEAHORSE (Long Snout, Big Fin, Long Tail) ---
                const tick = Date.now() / 150;
                const tailSway = Math.sin(tick) * 3; 
                const finFlutter = Math.sin(tick * 3) * 2;

                // 1. Big Dorsal Fin (Back/Left)
                ctx.beginPath();
                ctx.moveTo(-8, 2);
                // Control point -24 makes it stick out much further (Bigger)
                ctx.quadraticCurveTo(-24 + finFlutter, 10, -8, 18);
                ctx.fillStyle = p.color;
                ctx.fill();
                ctx.strokeStyle = 'rgba(0,0,0,0.1)';
                ctx.lineWidth = 1;
                ctx.stroke();

                // 2. Long Tail (Curled Forward/Right & Animated)
                ctx.beginPath();
                ctx.moveTo(0, 18);
                // Deep Y (40) and wide X (25) makes it longer
                ctx.bezierCurveTo(5, 40, 25 + tailSway, 38, 18 + tailSway, 20);
                ctx.lineWidth = 8;
                ctx.strokeStyle = p.color;
                ctx.lineCap = 'round';
                ctx.stroke();

                // 3. Body
                ctx.fillStyle = p.color;
                ctx.beginPath(); ctx.ellipse(0, 10, 12, 18, 0, 0, Math.PI * 2); ctx.fill();

                // 4. Belly
                ctx.fillStyle = COLORS.seahorseBelly;
                ctx.beginPath(); ctx.ellipse(4, 10, 7, 12, 0, 0, Math.PI * 2); ctx.fill();

                // 5. Head
                ctx.fillStyle = p.color;
                ctx.beginPath(); ctx.arc(0, -15, 14, 0, Math.PI * 2); ctx.fill();

                // 6. Long Snout & Eye
                ctx.save(); ctx.translate(12, -13); ctx.rotate(0.2);
                ctx.fillStyle = p.color; 
                ctx.beginPath(); 
                // Increased width to 16 (Longer)
                ctx.roundRect(0, -5, 16, 10, 5); 
                ctx.fill();
                ctx.restore();

                ctx.fillStyle = 'white'; ctx.beginPath(); ctx.arc(4, -16, 6.5, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = '#0f172a'; ctx.beginPath(); ctx.arc(6, -16, 3.5, 0, Math.PI * 2); ctx.fill();
                
                // Blush
                ctx.fillStyle = 'rgba(244, 114, 182, 0.5)';
                ctx.beginPath(); ctx.arc(0, -8, 4, 0, Math.PI * 2); ctx.fill();
                // --- END DRAW SEAHORSE ---

                ctx.restore();
            });

            ctx.restore();
            requestID = requestAnimationFrame(loop);
        }

        // Start on landing
        goToLanding();
    </script>
</body>
</html>