# Dashhhh-2.0
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Seattle Dash - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #192a56; font-family: -apple-system, sans-serif; position: fixed; }

        #game-stage {
            position: relative; width: 100vw; height: 100vh;
            background: #23272a; overflow: hidden;
        }

        /* Road Lines */
        .lane-marker {
            position: absolute; top: 0; bottom: 0; width: 4px;
            background: rgba(255,255,255,0.15); z-index: 1;
        }

        /* HIGH-VISIBILITY CHARACTER POSITION */
        #hero {
            position: absolute; 
            bottom: 35%; /* Pushed high up to clear the Safari bottom bar */
            width: 75px; height: 75px; font-size: 60px;
            display: flex; justify-content: center; align-items: center;
            transition: left 0.1s ease-out; z-index: 100; transform: translateX(-50%);
        }
        
        /* Hearts attached to player's feet */
        .heart-sparkle {
            position: absolute; font-size: 25px; z-index: 90;
            pointer-events: none; transform: translateX(-50%);
            animation: heartRise 0.8s forwards;
        }
        @keyframes heartRise {
            0% { opacity: 1; bottom: 35%; }
            100% { opacity: 0; bottom: 55%; transform: translate(-50%, -50px) scale(1.5); }
        }

        /* HUD - Top Safety */
        #hud {
            position: absolute; top: 10%; width: 100%;
            display: flex; flex-direction: column; align-items: center;
            z-index: 200; color: white; font-weight: bold; text-shadow: 2px 2px 5px #000;
        }
        .bar-outer { width: 70%; height: 14px; background: rgba(0,0,0,0.6); border-radius: 10px; margin-top: 10px; overflow: hidden; border: 2px solid white; }
        #bar-inner { width: 0%; height: 100%; background: #00d2d3; transition: width 0.3s; }

        .obstacle { position: absolute; font-size: 45px; transform: translateX(-50%); z-index: 50; }
        .overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.96); display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; z-index: 1000; padding: 40px; color: white; }
        .btn-go { padding: 20px 60px; border-radius: 50px; background: #00d2d3; color: white; border: none; font-size: 1.4rem; font-weight: bold; margin-top: 25px; }
    </style>
</head>
<body>

<div id="game-stage">
    <div class="lane-marker" style="left: 33.3%;"></div>
    <div class="lane-marker" style="left: 66.6%;"></div>
    
    <div id="hero">🏃‍♀️</div>

    <div id="hud">
        <div>Progress: <span id="score-val">0</span>/15</div>
        <div class="bar-outer"><div id="bar-inner"></div></div>
        <div style="font-size: 10px; margin-top: 5px;">DESTINATION: SEATTLE</div>
    </div>
</div>

<div id="start-screen" class="overlay">
    <h1 style="color: #00d2d3;">Road to Seattle! 🏔️</h1>
    <p style="margin-top: 15px;">Dodge the books and get to Seattle!</p>
    <button class="btn-go" onclick="beginAdventure()">START RUNNING 🎶</button>
</div>

<div id="win-screen" class="overlay" style="display:none; background: #fff; color: #333;">
    <h1 style="color: #00d2d3;">We Reached Seattle! ✈️</h1>
    <p style="margin: 20px 0;">Time for citizenM!</p>
    <p style="font-size: 1.6rem; font-weight: bold;">I love you, Giselle! ❤️</p>
</div>

<audio id="bgm" loop><source src="https://www.bensound.com/bensound-music/bensound-sunny.mp3" type="audio/mpeg"></audio>

<script>
    let active = false, tickets = 0, lane = 1, invincible = false, currentSpeed = 7;
    const laneX = [16.6, 50, 83.3];
    const hero = document.getElementById('hero');
    const hudScore = document.getElementById('score-val');
    const progressBar = document.getElementById('bar-inner');
    const music = document.getElementById('bgm');

    function beginAdventure() {
        document.getElementById('start-screen').style.display = 'none';
        active = true;
        music.play().catch(() => {});
        spawnItems();
    }

    let touchX;
    document.addEventListener('touchstart', (e) => touchX = e.touches[0].clientX);
    document.addEventListener('touchend', (e) => {
        if (!active) return;
        let diff = e.changedTouches[0].clientX - touchX;
        if (Math.abs(diff) > 25) {
            if (diff > 0 && lane < 2) lane++;
            else if (diff < 0 && lane > 0) lane--;
            hero.style.left = laneX[lane] + '%';
        }
    });

    function spawnItems() {
        if (!active) return;
        const item = document.createElement('div');
        item.className = 'obstacle';
        let r = Math.random();
        item.dataset.type = (r > 0.95) ? 'k' : (r > 0.65 ? 't' : 'b');
        item.innerHTML = (item.dataset.type === 'k') ? '🔑' : (item.dataset.type === 't' ? '✈️' : '📚');
        item.style.left = laneX[Math.floor(Math.random()*3)] + '%';
        item.style.top = '-50px';
        document.getElementById('game-stage').appendChild(item);

        let y = -50;
        const fall = setInterval(() => {
            if (!active) { clearInterval(fall); item.remove(); return; }
            y += (invincible ? 15 : currentSpeed);
            item.style.top = y + 'px';

            let pRect = hero.getBoundingClientRect();
            let iRect = item.getBoundingClientRect();

            if (pRect.left < iRect.right - 10 && pRect.right > iRect.left + 10 && pRect.top < iRect.bottom - 10 && pRect.bottom > iRect.top + 10) {
                if (item.dataset.type === 'b' && !invincible) {
                    active = false;
                    alert("Study stress! Try again, Giselle!");
                    location.reload();
                } else if (item.dataset.type === 't') {
                    tickets++; 
                    currentSpeed += 0.3; 
                    updateHUD(); 
                    item.remove();
                    clearInterval(fall);
                } else if (item.dataset.type === 'k') {
                    triggerPower(); item.remove(); clearInterval(fall);
                }
            }
            if (y > window.innerHeight) { clearInterval(fall); item.remove(); }
        }, 16);
        setTimeout(spawnItems, Math.max(500, 1100 - (tickets * 40)));
    }

    function updateHUD() {
        hudScore.innerText = tickets;
        progressBar.style.width = (tickets / 15 * 100) + '%';
        if (tickets >= 15) { 
            active = false; 
            document.getElementById('win-screen').style.display = 'flex'; 
        }
    }

    function triggerPower() {
        invincible = true;
        hero.innerHTML = '✈️';
        const trail = setInterval(() => {
            if (!invincible) { clearInterval(trail); return; }
            const h = document.createElement('div');
            h.className = 'heart-sparkle';
            h.innerHTML = '❤️';
            h.style.left = hero.style.left;
            document.getElementById('game-stage').appendChild(h);
            setTimeout(() => h.remove(), 700);
        }, 120);

        setTimeout(() => {
            invincible = false;
            hero.innerHTML = '🏃‍♀️';
        }, 4000);
    }

    hero.style.left = '50%';
</script>
</body>
</html>
