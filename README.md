<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>CyberStrike - Ultra Mod</title>
    <style>
        :root { --neon: #00f2ff; --danger: #ff0055; --gold: #ffcc00; }
        body { margin: 0; overflow: hidden; background: #050505; font-family: 'Segoe UI', sans-serif; touch-action: none; }
        canvas { display: block; filter: blur(0.5px) contrast(1.2); }
        
        /* Modern Mod Menü */
        #modMenu {
            position: absolute; top: 70px; right: 15px;
            background: rgba(0, 0, 0, 0.85); color: var(--neon);
            padding: 15px; border: 2px solid var(--neon); border-radius: 12px;
            box-shadow: 0 0 20px var(--neon); z-index: 100; width: 180px;
            backdrop-filter: blur(10px); display: none; transition: 0.3s;
        }

        #toggleMenuBtn {
            position: absolute; top: 15px; right: 15px; z-index: 101;
            background: var(--neon); color: #000; border: none;
            padding: 10px 15px; border-radius: 8px; font-weight: bold;
            box-shadow: 0 0 10px var(--neon); cursor: pointer;
        }

        /* Ateş Butonu - Modern Görünüm */
        #fireBtn {
            position: absolute; bottom: 40px; right: 40px;
            width: 90px; height: 90px; background: rgba(0, 242, 255, 0.2);
            border: 4px solid var(--neon); border-radius: 50%;
            color: var(--neon); font-weight: bold; z-index: 100;
            box-shadow: 0 0 15px var(--neon); outline: none;
        }

        #ui { position: absolute; top: 20px; left: 20px; color: white; text-shadow: 0 0 10px var(--neon); font-size: 28px; pointer-events: none; }
        .mod-item { margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; font-size: 14px; }
        .mod-item button { background: #222; color: #fff; border: 1px solid #444; border-radius: 4px; padding: 4px 8px; transition: 0.2s; }
        .active { background: var(--neon) !important; color: #000 !important; font-weight: bold; }
    </style>
</head>
<body>

    <div id="ui">SCORE: <span id="score">0</span></div>
    <button id="toggleMenuBtn" onclick="toggleMenu()">MOD MENU</button>

    <div id="modMenu">
        <div style="text-align:center; font-weight:bold; margin-bottom:15px; border-bottom:1px solid #333;">CYBER CHEATS</div>
        <div class="mod-item">AIMBOT <button id="aimbotBtn" onclick="toggleMod('aimbot')">OFF</button></div>
        <div class="mod-item">ESP LINES <button id="espBtn" onclick="toggleMod('esp')">OFF</button></div>
        <div class="mod-item">GOD MODE <button id="godBtn" onclick="toggleMod('god')">OFF</button></div>
        <div class="mod-item">INSTA-FIRE <button id="rapidBtn" onclick="toggleMod('rapid')">OFF</button></div>
    </div>

    <button id="fireBtn">FIRE</button>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
        
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let score = 0;
        let gameActive = true;
        let shake = 0;
        let mods = { aimbot: false, esp: false, god: false, rapid: false };
        
        const player = { x: canvas.width / 2, y: canvas.height - 150, radius: 22 };
        const enemies = [], bullets = [], particles = [];

        function toggleMenu() {
            const m = document.getElementById('modMenu');
            m.style.display = m.style.display === "block" ? "none" : "block";
        }

        function toggleMod(m) {
            mods[m] = !mods[m];
            document.getElementById(m+'Btn').innerText = mods[m] ? 'ON' : 'OFF';
            document.getElementById(m+'Btn').classList.toggle('active');
        }

        function createParticles(x, y, color) {
            for(let i=0; i<10; i++) {
                particles.push({
                    x, y, 
                    vx: (Math.random()-0.5)*10, 
                    vy: (Math.random()-0.5)*10, 
                    life: 1, 
                    color
                });
            }
        }

        function shoot() {
            bullets.push({ x: player.x, y: player.y - 20, speed: 15 });
            createParticles(player.x, player.y - 20, '#00f2ff');
        }

        document.getElementById('fireBtn').addEventListener('touchstart', (e) => { e.preventDefault(); shoot(); });

        window.addEventListener('touchmove', (e) => {
            if(!mods.aimbot) player.x = e.touches[0].clientX;
        });

        function update() {
            if (!gameActive) return;
            
            // Screen Shake Efekti
            let sx = (Math.random()-0.5)*shake;
            let sy = (Math.random()-0.5)*shake;
            ctx.setTransform(1,0,0,1,sx,sy);
            if(shake > 0) shake *= 0.9;

            ctx.fillStyle = "rgba(5, 5, 5, 0.3)"; // Motion Blur efekti
            ctx.fillRect(-10, -10, canvas.width+20, canvas.height+20);

            if(mods.rapid && Math.random() < 0.2) shoot();
            if(mods.aimbot && enemies.length > 0) player.x += (enemies[0].x - player.x) * 0.2;

            // Mermiler ve Partiküller
            particles.forEach((p, i) => {
                p.x += p.vx; p.y += p.vy; p.life -= 0.02;
                if(p.life <= 0) particles.splice(i, 1);
                ctx.fillStyle = p.color;
                ctx.globalAlpha = p.life;
                ctx.fillRect(p.x, p.y, 3, 3);
            });
            ctx.globalAlpha = 1;

            bullets.forEach((b, i) => {
                b.y -= b.speed;
                ctx.fillStyle = "#fff";
                ctx.shadowBlur = 15; ctx.shadowColor = "#00f2ff";
                ctx.fillRect(b.x-2, b.y, 4, 15);
                if(b.y < 0) bullets.splice(i,1);
            });

            // Oyuncu Çizimi (Neon Gemi)
            ctx.shadowBlur = 20; ctx.shadowColor = "#00f2ff";
            ctx.strokeStyle = "#00f2ff"; ctx.lineWidth = 4;
            ctx.beginPath();
            ctx.moveTo(player.x, player.y - 25);
            ctx.lineTo(player.x - 20, player.y + 15);
            ctx.lineTo(player.x + 20, player.y + 15);
            ctx.closePath(); ctx.stroke();

            if (Math.random() < 0.04) {
                enemies.push({ x: Math.random()*canvas.width, y: -50, speed: 3+Math.random()*4, r: 20 });
            }

            enemies.forEach((en, i) => {
                en.y += en.speed;
                if(mods.esp) {
                    ctx.beginPath(); ctx.moveTo(player.x, player.y); ctx.lineTo(en.x, en.y);
                    ctx.strokeStyle = "rgba(255, 0, 85, 0.3)"; ctx.lineWidth = 1; ctx.stroke();
                }

                ctx.shadowColor = "#ff0055"; ctx.strokeStyle = "#ff0055";
                ctx.strokeRect(en.x-15, en.y-15, 30, 30); // Düşmanlar kare neon

                // Mermi Çarpışma
                bullets.forEach((b, bi) => {
                    if(Math.hypot(b.x - en.x, b.y - en.y) < 25) {
                        createParticles(en.x, en.y, "#ff0055");
                        enemies.splice(i, 1); bullets.splice(bi, 1);
                        score++; scoreDisplay.innerText = score;
                        shake = 10;
                    }
                });

                if(Math.hypot(player.x - en.x, player.y - en.y) < 40 && !mods.god) {
                    gameActive = false; alert("SİSTEM ÇÖKTÜ! Skor: " + score); location.reload();
                }
                if(en.y > canvas.height) enemies.splice(i, 1);
            });

            requestAnimationFrame(update);
        }
        update();
    </script>
</body>
</html>
