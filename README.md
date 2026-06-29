<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Retro 3D Neo Racer</title>
    <!-- Load p5.js and p5.sound library for audio generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/addons/p5.sound.min.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #05050a;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            user-select: none;
        }
        #canvas-container {
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        /* Custom UI Overlays */
        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(5, 5, 15, 0.85);
            color: #fff;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.4s ease;
            z-index: 10;
        }
        .overlay.active {
            opacity: 1;
            pointer-events: auto;
        }
        .menu-card {
            background: linear-gradient(135deg, #151525 0%, #0a0a15 100%);
            border: 2px solid #3944bc;
            border-radius: 12px;
            padding: 30px 40px;
            width: 450px;
            max-width: 90%;
            text-align: center;
            box-shadow: 0 0 30px rgba(57, 68, 188, 0.4);
            transform: scale(0.9);
            transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        .overlay.active .menu-card {
            transform: scale(1);
        }
        h1 {
            font-size: 2.5rem;
            margin-top: 0;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 2px;
            color: #00ffcc;
            text-shadow: 0 0 10px rgba(0, 255, 204, 0.5);
        }
        h2 {
            font-size: 1.8rem;
            color: #ff007f;
            margin-bottom: 20px;
        }
        p {
            color: #ccc;
            line-height: 1.5;
            margin-bottom: 25px;
        }
        .btn {
            background: linear-gradient(90deg, #ff007f, #7928ca);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 1.1rem;
            font-weight: bold;
            border-radius: 25px;
            cursor: pointer;
            text-transform: uppercase;
            letter-spacing: 1px;
            transition: all 0.2s ease;
            box-shadow: 0 4px 15px rgba(255, 0, 127, 0.4);
            margin: 5px;
        }
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(255, 0, 127, 0.6);
            filter: brightness(1.1);
        }
        .btn:active {
            transform: translateY(1px);
        }
        /* Upgrade Grid */
        .upgrade-grid {
            display: flex;
            flex-direction: column;
            gap: 12px;
            margin-bottom: 25px;
            text-align: left;
        }
        .upgrade-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(255, 255, 255, 0.05);
            padding: 10px 15px;
            border-radius: 8px;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        .upgrade-info {
            flex-grow: 1;
        }
        .upgrade-name {
            font-weight: bold;
            color: #fff;
        }
        .upgrade-level {
            font-size: 0.85rem;
            color: #00ffcc;
        }
        .btn-upgrade {
            background: #22c55e;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            font-size: 0.9rem;
            transition: background 0.2s;
        }
        .btn-upgrade:hover:not(:disabled) {
            background: #16a34a;
        }
        .btn-upgrade:disabled {
            background: #4b5563;
            cursor: not-allowed;
            opacity: 0.5;
        }
        .currency-display {
            font-size: 1.2rem;
            color: #eab308;
            font-weight: bold;
            margin-bottom: 15px;
        }
        /* Instruction display */
        .controls-hint {
            font-size: 0.85rem;
            color: #64748b;
            margin-top: 15px;
        }
    </style>
</head>
<body>

    <div id="canvas-container"></div>

    <!-- Start Menu Overlay -->
    <div id="start-overlay" class="overlay active">
        <div class="menu-card">
            <h1>NEO RACER 3D</h1>
            <p>Race through 15 high-speed neon-lit obstacle courses. Upgrade your ride, drift around tight bends, hit boost ramps, and unlock absolute speed velocity.</p>
            <div style="background: rgba(0,0,0,0.3); padding: 10px; border-radius: 6px; margin-bottom: 20px; font-size: 0.9rem; text-align: left;">
                <strong>CONTROLS:</strong><br>
                • W / S : Accelerate / Brake / Reverse<br>
                • A / D : Steering Control<br>
                • SPACE : Handbrake Drift / Slide<br>
                • SHIFT : Nitro Boost (When available)<br>
                • ESC : Pause Game
            </div>
            <button class="btn" onclick="game.startFirstLevel()">Start Race</button>
        </div>
    </div>

    <!-- Upgrade / Level Complete Overlay -->
    <div id="upgrade-overlay" class="overlay">
        <div class="menu-card">
            <h1 id="complete-title">LEVEL COMPLETE!</h1>
            <div class="currency-display">Cash Earned: $<span id="cash-earned">0</span> | Total Balance: $<span id="total-cash">0</span></div>
            <p>Select a performance upgrade to bolster your hyper-vehicle stats:</p>
            
            <div class="upgrade-grid">
                <div class="upgrade-row">
                    <div class="upgrade-info">
                        <div class="upgrade-name">Top Speed</div>
                        <div class="upgrade-level" id="lvl-speed">Level: 1/5</div>
                    </div>
                    <button class="btn-upgrade" id="btn-up-speed" onclick="game.purchaseUpgrade('topSpeed')">Upgrade ($100)</button>
                </div>
                <div class="upgrade-row">
                    <div class="upgrade-info">
                        <div class="upgrade-name">Acceleration</div>
                        <div class="upgrade-level" id="lvl-accel">Level: 1/5</div>
                    </div>
                    <button class="btn-upgrade" id="btn-up-accel" onclick="game.purchaseUpgrade('acceleration')">Upgrade ($100)</button>
                </div>
                <div class="upgrade-row">
                    <div class="upgrade-info">
                        <div class="upgrade-name">Handling</div>
                        <div class="upgrade-level" id="lvl-handle">Level: 1/5</div>
                    </div>
                    <button class="btn-upgrade" id="btn-up-handle" onclick="game.purchaseUpgrade('handling')">Upgrade ($100)</button>
                </div>
                <div class="upgrade-row">
                    <div class="upgrade-info">
                        <div class="upgrade-name">Brakes</div>
                        <div class="upgrade-level" id="lvl-brakes">Level: 1/5</div>
                    </div>
                    <button class="btn-upgrade" id="btn-up-brakes" onclick="game.purchaseUpgrade('brakes')">Upgrade ($100)</button>
                </div>
                <div class="upgrade-row">
                    <div class="upgrade-info">
                        <div class="upgrade-name">Armor Structural Integrity</div>
                        <div class="upgrade-level" id="lvl-armor">Level: 1/5</div>
                    </div>
                    <button class="btn-upgrade" id="btn-up-armor" onclick="game.purchaseUpgrade('armor')">Upgrade ($100)</button>
                </div>
            </div>

            <button class="btn" id="btn-next-level" onclick="game.advanceLevel()">Next Track</button>
        </div>
    </div>

    <!-- Pause Overlay -->
    <div id="pause-overlay" class="overlay">
        <div class="menu-card">
            <h1>GAME PAUSED</h1>
            <p>Race execution safely suspended.</p>
            <button class="btn" onclick="game.togglePause()">Resume</button>
            <button class="btn" style="background: #374151;" onclick="game.restartCurrentLevel()">Restart Track</button>
        </div>
    </div>

    <!-- Game Over Overlay -->
    <div id="gameover-overlay" class="overlay">
        <div class="menu-card">
            <h1 style="color: #ef4444; text-shadow: 0 0 10px rgba(239, 68, 68, 0.5);">VEHICLE DESTROYED</h1>
            <p id="gameover-reason">Your structural hull integrity hit critical zero limit values.</p>
            <button class="btn" onclick="game.restartCurrentLevel()">Respawn Car</button>
        </div>
    </div>

    <!-- Victory Overlay -->
    <div id="victory-overlay" class="overlay">
        <div class="menu-card">
            <h1 style="color: #eab308; text-shadow: 0 0 10px rgba(234, 179, 8, 0.5);">GRAND CHAMPION!</h1>
            <p>Incredible! You have conquered all 15 tracks, navigated through every lethal hazard grid, and established ultimate road dominance.</p>
            <button class="btn" onclick="game.resetEntireGame()">Play Again</button>
        </div>
    </div>

    <script>
        // Global system bindings for architecture compliance
        let game;
        let polySynth; 

        function setup() {
            const canvas = createCanvas(windowWidth, windowHeight, WEBGL);
            canvas.parent('canvas-container');
            frameRate(60);
            
            // Audio initialization safely scoped
            polySynth = new p5.PolySynth();
            
            // Initialize Master Game instance
            game = new Game();
        }

        function draw() {
            game.update();
            game.render();
        }

        function windowResized() {
            resizeCanvas(windowWidth, windowHeight);
        }

        function keyPressed() {
            if (keyCode === ESCAPE) {
                game.togglePause();
            }
        }

        // ==========================================
        // AUDIO SYNTH ENGINE (No external file deps)
        // ==========================================
        class SoundEngine {
            // throttle state
            static lastEngineFreq = 0;
            static lastEnginePlayTime = 0;

            static playCrash() {
                try {
                    userStartAudio();
                    polySynth.play('G2', 0.6, 0, 0.1);
                    polySynth.play('C2', 0.5, 0.05, 0.2);
                } catch(e){}
            }
            static playCoin() {
                try {
                    userStartAudio();
                    polySynth.play('E5', 0.3, 0, 0.08);
                    polySynth.play('G6', 0.3, 0.06, 0.15);
                } catch(e){}
            }
            static playLevelUp() {
                try {
                    userStartAudio();
                    polySynth.play('C4', 0.4, 0, 0.1);
                    polySynth.play('E4', 0.4, 0.1, 0.1);
                    polySynth.play('G4', 0.4, 0.2, 0.1);
                    polySynth.play('C5', 0.5, 0.3, 0.3);
                } catch(e){}
            }
            static playEngineSound(speedRatio, isNitro) {
                // Throttled engine rev emulation to avoid many overlapping notes
                if (frameCount % 6 !== 0) return;
                try {
                    let baseFreq = map(speedRatio, 0, 1, 55, 160);
                    if (isNitro) baseFreq *= 1.4;

                    // Only play if above tiny throttle or a significant frequency change
                    if (speedRatio > 0.05) {
                        let timeNow = millis();
                        if ((abs(baseFreq - SoundEngine.lastEngineFreq) > 10) || (timeNow - SoundEngine.lastEnginePlayTime > 180)) {
                            polySynth.play(baseFreq, 0.08, 0, 0.05);
                            SoundEngine.lastEngineFreq = baseFreq;
                            SoundEngine.lastEnginePlayTime = timeNow;
                        }
                    }
                } catch(e){}
            }
        }

        // ==========================================
        // GAME CORE CLASS
        // ==========================================
        class Game {
            constructor() {
                this.state = 'START'; // START, PLAYING, PAUSED, UPGRADE, GAMEOVER, VICTORY
                this.currentLevelIndex = 1;
                this.money = 0;
                
                // Fetch saved game progress safely
                this.loadProgress();

                // Ensure player exists before applying upgrades so saved upgrades apply correctly
                this.player = new Player(this);
                this.upgradeSystem = new UpgradeSystem(this);
                // Apply upgrades again to be safe now that player exists
                this.upgradeSystem.applyUpgradesToPlayer();

                this.cameraController = new CameraController(this);
                this.level = new Level(this.currentLevelIndex);
                
                this.levelTimer = 0;
            }

            startFirstLevel() {
                // Unlock audio context on explicit user gesture (Start Race button)
                try { userStartAudio(); } catch(e) {}
                document.getElementById('start-overlay').classList.remove('active');
                this.state = 'PLAYING';
                this.loadLevel(this.currentLevelIndex);
            }

            loadLevel(index) {
                this.currentLevelIndex = index;
                this.level = new Level(index);
                this.player.resetPosition(this.level.getStartPoint());
                this.levelTimer = 0;
                this.state = 'PLAYING';
                this.saveProgress();
                
                // Clear UI active classes safely
                document.querySelectorAll('.overlay').forEach(el => el.classList.remove('active'));
            }

            update() {
                if (this.state !== 'PLAYING') return;

                this.levelTimer += deltaTime / 1000;
                this.player.update(this.level);
                this.level.update(this.player);
                this.cameraController.update(this.player);

                // Audio tick simulation
                let speedRatio = abs(this.player.car.speed) / this.player.car.stats.topSpeed;
                SoundEngine.playEngineSound(speedRatio, this.player.car.isBoosting);

                // Check critical track progression metrics
                if (this.player.car.health <= 0) {
                    this.triggerGameOver("Your vehicle frame suffered critical structural explosion.");
                }

                // Finish line execution boundaries
                if (this.level.hasPassedFinishLine(this.player.car.pos)) {
                    this.triggerLevelComplete();
                }
            }

            render() {
                // Setup atmospheric space depth
                background(10, 10, 22);
                
                // WebGL lighting maps
                ambientLight(80, 80, 100);
                directionalLight(255, 255, 255, 0.5, 1, -0.3);
                pointLight(0, 255, 204, this.player.car.pos.x, this.player.car.pos.y - 100, this.player.car.pos.z);

                // Establish custom primary viewing matrix context
                this.cameraController.apply();

                // Draw Track Geometry Environments
                this.level.render();

                // Draw Actor Agents
                this.player.render();

                // Draw Flat Screen Space Interface via alternate matrix operations
                this.renderHUD();
            }

            renderHUD() {
                // Projecting standard flat typography over 3D context manually safely inside WebGL limits
                push();
                // Push orthographic projection reset safely by avoiding native camera wipe constraints
                resetMatrix();
                translate(-width/2, -height/2, 0);
                
                // Top-left dashboard metrics
                fill(255);
                noStroke();
                textSize(18);
                textAlign(LEFT, TOP);
                text(`TRACK GRID: ${this.currentLevelIndex} / 15`, 20, 20);
                
                let speedKmh = Math.floor(abs(this.player.car.speed) * 12);
                text(`VELOCITY: ${speedKmh} KM/H`, 20, 45);
                text(`TIME SEC: ${this.levelTimer.toFixed(1)}`, 20, 70);
                text(`CREDITS: $${this.money}`, 20, 95);

                // Dynamic Health Bar Overlay UI matrix
                fill(40);
                rect(20, 125, 150, 12, 4);
                let hpPct = max(0, this.player.car.health / 100);
                fill(hpPct > 0.4 ? '#00ffcc' : '#ef4444');
                rect(20, 125, 150 * hpPct, 12, 4);

                // Nitro Tank gauge
                fill(40);
                rect(20, 145, 150, 6, 2);
                fill('#7928ca');
                rect(20, 145, 150 * (this.player.car.nitroEnergy / 100), 6, 2);

                // Top right Mini-map pipeline transformation
                this.drawMiniMap();

                pop();
            }

            drawMiniMap() {
                push();
                translate(width - 140, 20);
                fill(15, 15, 30, 180);
                stroke(57, 68, 188);
                strokeWeight(2);
                rect(0, 0, 120, 120, 8);

                // Map track coordinates scalar conversion
                noStroke();
                fill(255, 0, 127);
                
                // Plot checkpoints inside minimap
                for (let cp of this.level.checkpoints) {
                    let mx = map(cp.x, -2000, 2000, 10, 110);
                    let mz = map(cp.z, -15000, 2000, 110, 10);
                    ellipse(mx, mz, 4, 4);
                }

                // Plot Player Marker
                let px = map(this.player.car.pos.x, -2000, 2000, 10, 110);
                let pz = map(this.player.car.pos.z, -15000, 2000, 110, 10);
                fill(0, 255, 204);
                ellipse(px, pz, 7, 7);
                pop();
            }

            togglePause() {
                if (this.state === 'PLAYING') {
                    this.state = 'PAUSED';
                    document.getElementById('pause-overlay').classList.add('active');
                } else if (this.state === 'PAUSED') {
                    this.state = 'PLAYING';
                    document.getElementById('pause-overlay').classList.remove('active');
                }
            }

            triggerGameOver(reason) {
                this.state = 'GAMEOVER';
                SoundEngine.playCrash();
                document.getElementById('gameover-reason').innerText = reason;
                document.getElementById('gameover-overlay').classList.add('active');
            }

            triggerLevelComplete() {
                this.state = 'UPGRADE';
                SoundEngine.playLevelUp();
                
                let cashReward = 100 + (this.currentLevelIndex * 50) + max(0, Math.floor(300 - this.levelTimer));
                this.money += cashReward;

                document.getElementById('cash-earned').innerText = cashReward;
                document.getElementById('total-cash').innerText = this.money;
                
                this.upgradeSystem.updateUIElements();
                
                if (this.currentLevelIndex >= 15) {
                    document.getElementById('complete-title').innerText = "ALL RACES COMPLETE!";
                    document.getElementById('btn-next-level').innerText = "Claim Victory";
                } else {
                    document.getElementById('complete-title').innerText = `TRACK ${this.currentLevelIndex} CLEARED!`;
                    document.getElementById('btn-next-level').innerText = "Next Track";
                }

                document.getElementById('upgrade-overlay').classList.add('active');
            }

            purchaseUpgrade(statKey) {
                this.upgradeSystem.buyUpgrade(statKey);
            }

            advanceLevel() {
                if (this.currentLevelIndex >= 15) {
                    this.state = 'VICTORY';
                    document.getElementById('upgrade-overlay').classList.remove('active');
                    document.getElementById('victory-overlay').classList.add('active');
                } else {
                    this.loadLevel(this.currentLevelIndex + 1);
                }
            }

            restartCurrentLevel() {
                this.player.car.health = 100;
                this.loadLevel(this.currentLevelIndex);
            }

            resetEntireGame() {
                localStorage.clear();
                this.currentLevelIndex = 1;
                this.money = 0;
                // Recreate player before upgrades so upgrade application works
                this.player = new Player(this);
                this.upgradeSystem = new UpgradeSystem(this);
                this.upgradeSystem.applyUpgradesToPlayer();
                document.getElementById('victory-overlay').classList.remove('active');
                this.loadLevel(1);
            }

            saveProgress() {
                let data = {
                    levelIndex: this.currentLevelIndex,
                    money: this.money,
                    upgrades: this.upgradeSystem.levels
                };
                localStorage.setItem('neo_racer_save', JSON.stringify(data));
            }

            loadProgress() {
                let saved = localStorage.getItem('neo_racer_save');
                if (saved) {
                    try {
                        let parsed = JSON.parse(saved);
                        this.currentLevelIndex = parsed.levelIndex || 1;
                        this.money = parsed.money || 0;
                    } catch(e) {}
                }
            }
        }

        // ==========================================
        // CAR PHYSICAL AGENT LAYER
        // ==========================================
        class Car {
            constructor(upgradeStats) {
                this.pos = createVector(0, 0, 0);
                this.vel = createVector(0, 0, 0);
                this.angle = 0;
                this.speed = 0;
                this.health = 100;
                this.nitroEnergy = 100;
                this.isBoosting = false;

                this.skidMarks = [];
                this.particles = [];

                // Map stats dynamically based on upgrade vectors
                this.stats = upgradeStats;
            }

            update(controls, level) {
                // Performance modifiers
                let currentMaxSpeed = this.isBoosting ? this.stats.topSpeed * 1.5 : this.stats.topSpeed;
                let currentAccel = this.isBoosting ? this.stats.acceleration * 1.8 : this.stats.acceleration;

                // Handle Input Physics Matrix Mapping
                if (controls.forward) {
                    this.speed += currentAccel;
                    if (this.speed > currentMaxSpeed) this.speed = currentMaxSpeed;
                } else if (controls.backward) {
                    this.speed -= this.stats.brakes;
                    if (this.speed < -this.stats.topSpeed * 0.4) this.speed = -this.stats.topSpeed * 0.4;
                } else {
                    // Standard environmental roll friction loss
                    this.speed *= 0.96;
                }

                // Turning circle formulas dynamically linked with forward vector velocity
                if (abs(this.speed) > 0.1) {
                    let turningFactor = map(abs(this.speed), 0, this.stats.topSpeed, 0.01, this.stats.handling);
                    if (this.speed < 0) turningFactor *= -1; // Inverse reversing directional turn logic

                    if (controls.left) this.angle += turningFactor;
                    if (controls.right) this.angle -= turningFactor;

                    // Execute drift mechanics injection
                    if (controls.drift) {
                        this.angle += (controls.left ? 0.015 : 0) - (controls.right ? 0.015 : 0);
                        this.speed *= 0.98; // Additional kinetic velocity rub friction loss
                        this.spawnSkidMark();
                    }
                }

                // Nitro injection operational tracking bounds
                if (controls.nitro && this.nitroEnergy > 0 && controls.forward) {
                    this.isBoosting = true;
                    this.nitroEnergy -= 1.5;
                    this.spawnBoostParticle();
                } else {
                    this.isBoosting = false;
                    if (this.nitroEnergy < 100) this.nitroEnergy += 0.1;
                }

                // Structural velocity conversion matrix
                this.vel.x = sin(this.angle) * this.speed;
                this.vel.z = cos(this.angle) * this.speed;

                this.pos.add(this.vel);

                // Keep car grounded firmly above variable terrain matrix (Y positions check)
                let groundY = level.getGroundHeightAt(this.pos.x, this.pos.z);
                this.pos.y = groundY;

                // Update sub-particle buffer structures
                for (let i = this.particles.length - 1; i >= 0; i--) {
                    this.particles[i].update();
                    if (this.particles[i].alpha <= 0) this.particles.splice(i, 1);
                }
                if (this.skidMarks.length > 60) this.skidMarks.shift();
            }

            inflictDamage(amount) {
                // Apply armor safety calculations
                let damageTaken = amount * (1 - this.stats.armorModifier);
                this.health -= damageTaken;
                SoundEngine.playCrash();
                
                // Rebound impact recoil matrix offset
                this.speed = -this.speed * 0.4;
            }

            spawnSkidMark() {
                this.skidMarks.push({
                    x: this.pos.x - sin(this.angle)*5,
                    y: this.pos.y + 0.5,
                    z: this.pos.z - cos(this.angle)*5
                });
            }

            spawnBoostParticle() {
                this.particles.push(new Particle(this.pos.x, this.pos.y + 3, this.pos.z, '#7928ca'));
            }

            render() {
                // Draw trailing tracking skid paths
                push();
                fill(30, 30, 40);
                noStroke();
                for (let sm of this.skidMarks) {
                    push();
                    translate(sm.x, sm.y, sm.z);
                    box(8, 0.2, 4);
                    pop();
                }
                pop();

                // Draw linked active drive-exhaust simulation particles
                for (let p of this.particles) {
                    p.render();
                }

                // Primary Mesh Group Transformation Matrix Execution
                push();
                translate(this.pos.x, this.pos.y + 4, this.pos.z);
                rotateY(this.angle);

                // Main chassis fuselage structure block assembly
                stroke(0);
                strokeWeight(1);
                fill(0, 255, 204); // Cyberpunk bright core color
                if (this.isBoosting) fill(255, 0, 127);
                box(14, 6, 26);

                // Driver cabin glass bubble assembly
                push();
                translate(0, 4, -2);
                fill(15, 15, 40, 200);
                box(10, 4, 12);
                pop();

                // Wheel hub assemblies cylinders
                fill(20);
                // Front axle layout offset points
                push(); translate(7, -2, 8); rotateZ(HALF_PI); cylinder(3, 2); pop();
                push(); translate(-7, -2, 8); rotateZ(HALF_PI); cylinder(3, 2); pop();
                // Rear heavy power axle setup
                push(); translate(7, -2, -8); rotateZ(HALF_PI); cylinder(3.5, 2.5); pop();
                push(); translate(-7, -2, -8); rotateZ(HALF_PI); cylinder(3.5, 2.5); pop();

                pop();
            }
        }

        // ==========================================
        // PLAYER INPUT / MANIFEST ARCHITECTURE
        // ==========================================
        class Player {
            constructor(gameInstance) {
                this.game = gameInstance;
                this.carStats = { topSpeed: 18, acceleration: 0.35, handling: 0.04, brakes: 0.6, armorModifier: 0.0 };
                this.car = new Car(this.carStats);
            }

            resetPosition(startVector) {
                this.car.pos = startVector.copy();
                this.car.vel.set(0,0,0);
                this.car.speed = 0;
                this.car.angle = 0;
                this.car.health = 100;
                this.car.skidMarks = [];
            }

            update(level) {
                let controls = {
                    forward: keyIsDown(87) || keyIsDown(UP_ARROW),    // W
                    backward: keyIsDown(83) || keyIsDown(DOWN_ARROW), // S
                    left: keyIsDown(65) || keyIsDown(LEFT_ARROW),     // A
                    right: keyIsDown(68) || keyIsDown(RIGHT_ARROW),   // D
                    drift: keyIsDown(32),                             // Space
                    nitro: keyIsDown(16)                              // Shift
                };

                this.car.update(controls, level);
            }

            render() {
                this.car.render();
            }
        }

        // ==========================================
        // PROCEDURAL TRACK ENGINE / GRAPHICS SCENE
        // ==========================================
        class Level {
            constructor(levelIndex) {
                this.index = levelIndex;
                this.roadSegments = [];
                this.obstacles = [];
                this.checkpoints = [];
                this.decorations = [];

                this.generateTrackStructure();
            }

            generateTrackStructure() {
                // Linear layout metrics scaling strictly with difficulty index parameters
                let segmentCount = 15 + this.index * 6;
                let roadWidth = max(70, 180 - this.index * 8);
                let currentZ = 0;
                let currentX = 0;
                let currentAngle = 0;

                for (let i = 0; i < segmentCount; i++) {
                    // Procedural turn calculations introducing narrow high level zigzags
                    if (i > 3 && i < segmentCount - 2) {
                        if (random() < 0.4 + (this.index * 0.03)) {
                            currentAngle += random([-0.25, 0.25, -0.4, 0.4]) * (this.index * 0.15);
                        }
                    }

                    // Special Feature Generation: Jump Ramps setup mapping coordinates
                    let hasRamp = (i > 4 && i % 6 === 0 && random() > 0.3);
                    let rampHeight = hasRamp ? 25 : 0;

                    let segment = {
                        x: currentX,
                        y: hasRamp ? -10 : 0, 
                        z: currentZ,
                        width: roadWidth,
                        length: 300,
                        angle: currentAngle,
                        isRamp: hasRamp,
                        rampHeight: rampHeight
                    };

                    this.roadSegments.push(segment);

                    // Scatter structural asset components securely into decoration zones
                    if (i < segmentCount - 1) {
                        // Left-Right scenery items distribution buffers outside road width bounds
                        this.decorations.push({ x: currentX - roadWidth - random(40, 150), z: currentZ + random(-100, 100), type: random(['TREE', 'BUILDING']) });
                        this.decorations.push({ x: currentX + roadWidth + random(40, 150), z: currentZ + random(-100, 100), type: random(['TREE', 'BUILDING']) });
                    }

                    // Inject interactive hazard obstacle maps depending on current level scale multiplier metrics
                    if (i > 2 && i < segmentCount - 1) {
                        let obstacleCount = Math.floor(random(1, 2 + (this.index / 4)));
                        for (let o = 0; o < obstacleCount; o++) {
                            let ox = currentX + random(-roadWidth * 0.4, roadWidth * 0.4);
                            let oz = currentZ + random(-120, 120);
                            let type = random(['BARRIER', 'SPIKE_WALL', 'COIN', 'MOVING_BLOCK']);
                            this.obstacles.push(new Obstacle(ox, segment.y, oz, type, this.index));
                        }
                    }

                    // Cache structural coordinates checkpoint boundaries track maps data arrays
                    this.checkpoints.push(createVector(currentX, segment.y, currentZ));

                    // Step vectors calculation markers forward
                    currentZ -= cos(currentAngle) * 300;
                    currentX += sin(currentAngle) * 300;
                }
            }

            getStartPoint() {
                return createVector(0, 0, 400);
            }

            getGroundHeightAt(x, z) {
                // Match closest physical lane structure node configuration map calculations
                let closestSeg = null;
                let minDist = 999999;

                for (let seg of this.roadSegments) {
                    let d = dist(x, z, seg.x, seg.z);
                    if (d < minDist) {
                        minDist = d;
                        closestSeg = seg;
                    }
                }

                if (closestSeg && closestSeg.isRamp && minDist < 150) {
                    // Compute interpolation altitude slope along length parameters matrix bounds
                    return closestSeg.y + Math.max(0, (1 - (minDist / 150)) * closestSeg.rampHeight);
                }
                return 0;
            }

            update(player) {
                // Update active obstacle behaviors (Moving blocks calculations)
                for (let obs of this.obstacles) {
                    obs.update();
                    
                    // Box-sphere proximity checking collision boundary
                    if (obs.active && dist(player.car.pos.x, player.car.pos.y, player.car.pos.z, obs.pos.x, obs.pos.y, obs.pos.z) < obs.radius + 6) {
                        if (obs.type === 'COIN') {
                            obs.active = false;
                            player.game.money += 25;
                            SoundEngine.playCoin();
                        } else {
                            // Damage obstruction tracking node
                            player.car.inflictDamage(obs.damageValue);
                            obs.active = false; // Disable collision repeatedly
                        }
                    }
                }
            }

            hasPassedFinishLine(playerPos) {
                let lastSeg = this.roadSegments[this.roadSegments.length - 1];
                return playerPos.z <= lastSeg.z;
            }

            render() {
                // Infinite horizon flat grid terrain projection
                push();
                translate(0, 0, this.checkpoints[Math.floor(this.checkpoints.length/2)].z);
                fill(12, 45, 25);
                noStroke();
                // Ground structural context floor plane mapping
                rotateX(HALF_PI);
                rect(-10000, -10000, 20000, 20000);
                pop();

                // Render compiled road tracks segments
                for (let i = 0; i < this.roadSegments.length; i++) {
                    let seg = this.roadSegments[i];
                    push();
                    translate(seg.x, seg.y, seg.z);
                    rotateY(seg.angle);

                    // Alternating checker aesthetic lane paint markers layout blocks
                    fill(i % 2 === 0 ? 30 : 45);
                    stroke(57, 68, 188); // Neon glowing track outline walls
                    strokeWeight(1);
                    
                    if (seg.isRamp) {
                        // Specialized geometry rendering node transformations for jumps
                        fill(75, 30, 90);
                        rotateX(-0.12);
                        box(seg.width, 4, seg.length);
                    } else {
                        box(seg.width, 2, seg.length);
                    }
                    pop();
                }

                // Render dynamic object array lists context
                for (let obs of this.obstacles) {
                    obs.render();
                }

                // Render background structures assets matrix environment lines
                for (let dec of this.decorations) {
                    push();
                    translate(dec.x, 0, dec.z);
                    if (dec.type === 'TREE') {
                        fill(15, 80, 45); stroke(0);
                        translate(0, 15, 0);
                        cone(12, 30);
                    } else {
                        fill(25, 25, 45); stroke(50, 100, 255);
                        translate(0, 40, 0);
                        box(35, 80, 35);
                    }
                    pop();
                }

                // Draw finish line arch structure decoration over end sector array node points
                let finalSeg = this.roadSegments[this.roadSegments.length - 1];
                push();
                translate(finalSeg.x, finalSeg.y, finalSeg.z);
                rotateY(finalSeg.angle);
                
                // Left-Right architectural framework beam columns setup blocks
                fill(255, 0, 127); noStroke();
                push(); translate(-finalSeg.width/2, 25, 0); box(6, 50, 6); pop();
                push(); translate(finalSeg.width/2, 25, 0); box(6, 50, 6); pop();
                // Overhanging sign banner component display block structure
                translate(0, 50, 0);
                fill(15, 15, 30);
                stroke(255, 0, 127);
                strokeWeight(2);
                box(finalSeg.width, 14, 8);
                pop();
            }
        }

        // ==========================================
        // INTERACTIVE OBSTACLE ENTITY
        // ==========================================
        class Obstacle {
            constructor(x, y, z, type, levelIndex) {
                this.pos = createVector(x, y, z);
                this.type = type;
                this.active = true;
                
                this.baseX = x;
                this.moveRange = random(40, 120);
                this.moveSpeed = random(0.02, 0.05) + (levelIndex * 0.003);

                // Property map assignments bounding circles safely
                switch (type) {
                    case 'BARRIER':
                        this.radius = 12; this.damageValue = 20; colorMode(); this.color = '#ef4444'; break;
                    case 'SPIKE_WALL':
                        this.radius = 16; this.damageValue = 35; this.color = '#7f1d1d'; break;
                    case 'MOVING_BLOCK':
                        this.radius = 15; this.damageValue = 25; this.color = '#ea580c'; break;
                    case 'COIN':
                        this.radius = 8; this.damageValue = 0; this.color = '#eab308'; break;
                }
            }

            update() {
                if (this.type === 'MOVING_BLOCK') {
                    // Left right sweep simulation parameter calculations
                    this.pos.x = this.baseX + sin(frameCount * this.moveSpeed) * this.moveRange;
                }
            }

            render() {
                if (!this.active) return;

                push();
                translate(this.pos.x, this.pos.y + 6, this.pos.z);
                fill(this.color);
                stroke(0);
                strokeWeight(1);

                if (this.type === 'COIN') {
                    rotateY(frameCount * 0.05);
                    cylinder(this.radius, 2);
                } else if (this.type === 'SPIKE_WALL') {
                    box(this.radius * 2, 14, 6);
                } else if (this.type === 'MOVING_BLOCK') {
                    box(this.radius * 1.8, 12, this.radius * 1.8);
                } else {
                    // Standard structural street barrier barrel setup visual representation node
                    cylinder(this.radius * 0.6, 12);
                }
                pop();
            }
        }

        // ==========================================
        // THIRD PERSON CAMERA CONTROLLER SYSTEM
        // ==========================================
        class CameraController {
            constructor() {
                this.pos = createVector(0, 120, 300);
                this.lookAtTarget = createVector(0, 0, 0);
            }

            update(player) {
                // Compute look ahead trailing positions mathematically vectors mappings formulas
                let targetCamX = player.car.pos.x - sin(player.car.angle) * 85;
                let targetCamZ = player.car.pos.z - cos(player.car.angle) * 85;
                let targetCamY = player.car.pos.y + 32;

                // Smooth linear interpolation damping routine transitions framework
                this.pos.x = lerp(this.pos.x, targetCamX, 0.08);
                this.pos.y = lerp(this.pos.y, targetCamY, 0.08);
                this.pos.z = lerp(this.pos.z, targetCamZ, 0.08);

                // Focus focal point structural target mapping slightly in front of car bumper edge
                this.lookAtTarget.x = player.car.pos.x + sin(player.car.angle) * 30;
                this.lookAtTarget.y = player.car.pos.y + 10;
                this.lookAtTarget.z = player.car.pos.z + cos(player.car.angle) * 30;
            }

            apply() {
                // Apply p5 WebGL matrix stack transformation natively via parameters tracking arrays
                camera(this.pos.x, this.pos.y, this.pos.z, 
                       this.lookAtTarget.x, this.lookAtTarget.y, this.lookAtTarget.z, 
                       0, -1, 0); // Reverse Y matrix sign parameter matching WEBGL coordinate directions
            }
        }

        // ==========================================
        // PERFORMANCE STATE METRIC UPGRADE SYSTEM
        // ==========================================
        class UpgradeSystem {
            constructor(gameInstance) {
                this.game = gameInstance;
                
                // Track current system purchase metrics safely
                this.levels = { topSpeed: 1, acceleration: 1, handling: 1, brakes: 1, armor: 1 };
                this.cost = 150;

                this.loadUpgradeLevels();
                this.applyUpgradesToPlayer();
            }

            applyUpgradesToPlayer() {
                if (!this.game || !this.game.player) return;
                
                // Mapping discrete configuration matrices directly modifying vehicle physics
                this.game.player.carStats.topSpeed = 16 + (this.levels.topSpeed * 2.5);
                this.game.player.carStats.acceleration = 0.3 + (this.levels.acceleration * 0.08);
                this.game.player.carStats.handling = 0.035 + (this.levels.handling * 0.007);
                this.game.player.carStats.brakes = 0.5 + (this.levels.brakes * 0.15);
                this.game.player.carStats.armorModifier = (this.levels.armor - 1) * 0.18; // Max lvl 5 gives roughly ~72% damage block protection safety shielding
            }

            buyUpgrade(statKey) {
                if (this.levels[statKey] >= 5) return; // System hard ceiling limits index tracking bounds Max level 5
                
                if (this.game.money >= this.cost) {
                    this.game.money -= this.cost;
                    this.levels[statKey]++;
                    this.applyUpgradesToPlayer();
                    this.updateUIElements();
                    this.game.saveProgress();
                    
                    // Live interface cash balance indicator update injection handling
                    document.getElementById('total-cash').innerText = this.game.money;
                }
            }

            updateUIElements() {
                // Synchronize HTML element displays data loops explicitly mapping variables
                document.getElementById('lvl-speed').innerText = `Level: ${this.levels.topSpeed}/5`;
                document.getElementById('lvl-accel').innerText = `Level: ${this.levels.acceleration}/5`;
                document.getElementById('lvl-handle').innerText = `Level: ${this.levels.handling}/5`;
                document.getElementById('lvl-brakes').innerText = `Level: ${this.levels.brakes}/5`;
                document.getElementById('lvl-armor').innerText = `Level: ${this.levels.armor}/5`;

                // Handle button lock out evaluation logic when maxed or lacking credits
                this.evalBtnState('topSpeed', 'btn-up-speed');
                this.evalBtnState('acceleration', 'btn-up-accel');
                this.evalBtnState('handling', 'btn-up-handle');
                this.evalBtnState('brakes', 'btn-up-brakes');
                this.evalBtnState('armor', 'btn-up-armor');
            }

            evalBtnState(key, domId) {
                let btn = document.getElementById(domId);
                if (this.levels[key] >= 5) {
                    btn.innerText = "MAX LEVEL";
                    btn.disabled = true;
                } else {
                    btn.innerText = `Upgrade ($${this.cost})`;
                    btn.disabled = (this.game.money < this.cost);
                }
            }

            loadUpgradeLevels() {
                let saved = localStorage.getItem('neo_racer_save');
                if (saved) {
                    try {
                        let parsed = JSON.parse(saved);
                        if (parsed.upgrades) this.levels = parsed.upgrades;
                    } catch(e) {}
                }
            }
        }

        // ==========================================
        // AMBIENT ENGINE GAS EXHAUST DUST PARTICLE
        // ==========================================
        class Particle {
            constructor(x, y, z, colorStr) {
                this.pos = createVector(x, y, z);
                this.vel = createVector(random(-1.5, 1.5), random(1, 3), random(-1.5, 1.5));
                this.alpha = 255;
                this.color = colorStr;
                this.size = random(2, 5);
            }

            update() {
                this.pos.add(this.vel);
                this.alpha -= 8; // Fade routine sequence over time cycle loops
            }

            render() {
                push();
                translate(this.pos.x, this.pos.y, this.pos.z);
                noStroke();
                fill(121, 40, 202, this.alpha);
                box(this.size);
                pop();
            }
        }
    </script>
</body>
</html>
