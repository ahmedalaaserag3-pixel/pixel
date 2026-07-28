<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pixel World - عالم الألعاب الجماعي</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #1a1a1a;
            color: white;
            font-family: 'Tahoma', sans-serif;
            overflow: hidden;
            text-align: center;
            touch-action: none;
        }
        #login-screen {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(135deg, #6e8efb, #a777e3);
        }
        input, select, button {
            padding: 14px 20px;
            margin: 10px;
            border-radius: 10px;
            border: none;
            font-size: 16px;
            outline: none;
            width: 80%;
            max-width: 300px;
        }
        button {
            background-color: #ffcc00;
            color: #333;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
        }
        button:active {
            transform: scale(0.95);
        }
        #game-screen {
            display: none;
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        canvas {
            background: #222;
            display: block;
            width: 100%;
            height: 70vh;
        }
        #ui-bar {
            position: absolute;
            top: 10px;
            right: 15px;
            background: rgba(0,0,0,0.6);
            padding: 8px 15px;
            border-radius: 8px;
            font-size: 14px;
        }
        /* أزرار التحكم باللمس للموبايل */
        #touch-controls {
            position: absolute;
            bottom: 20px;
            width: 100%;
            height: 25vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .dpad {
            position: relative;
            width: 160px;
            height: 160px;
        }
        .t-btn {
            position: absolute;
            background: rgba(255, 255, 255, 0.2);
            border: 2px solid rgba(255, 255, 255, 0.4);
            color: white;
            font-size: 24px;
            font-weight: bold;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none;
            -webkit-user-select: none;
        }
        .t-btn:active {
            background: rgba(255, 204, 0, 0.5);
        }
        #btn-up { top: 0; left: 55px; }
        #btn-down { bottom: 0; left: 55px; }
        #btn-left { top: 55px; left: 0; }
        #btn-right { top: 55px; right: 0; }
    </style>
</head>
<body>

    <!-- شاشة تسجيل الدخول -->
    <div id="login-screen">
        <h1>🌟 أهلاً بك في Pixel World 🌟</h1>
        <p>اكتب اسمك ودخل الماب عشان تلعب!</p>
        <input type="text" id="username" placeholder="اسم اللاعب..." maxlength="15">
        <select id="map-select">
            <option value="map1">ماب المدينة الكبيرة 🏙️</option>
            <option value="map2">ماب جزيرة المغامرات 🌴</option>
            <option value="map3">ماب الملاهي السريعة 🎢</option>
        </select>
        <button onclick="startGame()">دخول اللعبة 🚀</button>
    </div>

    <!-- شاشة اللعبة -->
    <div id="game-screen">
        <div id="ui-bar">
            <span id="player-display-name"></span> | 
            <span id="current-map-name">المدينة</span>
        </div>
        <canvas id="gameCanvas" width="800" height="450"></canvas>

        <!-- أزرار التحكم باللمس -->
        <div id="touch-controls">
            <div class="dpad">
                <div class="t-btn" id="btn-up">⬆️</div>
                <div class="t-btn" id="btn-down">⬇️</div>
                <div class="t-btn" id="btn-left">⬅️</div>
                <div class="t-btn" id="btn-right">➡️</div>
            </div>
        </div>
    </div>

    <script>
        const loginScreen = document.getElementById('login-screen');
        const gameScreen = document.getElementById('game-screen');
        const usernameInput = document.getElementById('username');
        const mapSelect = document.getElementById('map-select');
        const playerDisplayName = document.getElementById('player-display-name');
        const currentMapName = document.getElementById('current-map-name');
        
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let myPlayer = {
            x: 400,
            y: 225,
            size: 30,
            speed: 5,
            color: '#' + Math.floor(Math.random()*16777215).toString(16),
            name: ''
        };

        let keys = {};

        function startGame() {
            const name = usernameInput.value.trim();
            if (!name) {
                alert('يا غالي لازم تكتب اسم الأول!');
                return;
            }
            myPlayer.name = name;
            playerDisplayName.innerText = name;
            currentMapName.innerText = mapSelect.options[mapSelect.selectedIndex].text;

            loginScreen.style.display = 'none';
            gameScreen.style.display = 'block';

            gameLoop();
        }

        // دعم الكيبورد (للكمبيوتر)
        window.addEventListener('keydown', (e) => keys[e.key] = true);
        window.addEventListener('keyup', (e) => keys[e.key] = false);

        // دعم اللمس (للموبايل)
        function bindTouchButton(id, keyName) {
            const btn = document.getElementById(id);
            btn.addEventListener('touchstart', (e) => { e.preventDefault(); keys[keyName] = true; });
            btn.addEventListener('touchend', (e) => { e.preventDefault(); keys[keyName] = false; });
            btn.addEventListener('mousedown', () => keys[keyName] = true);
            btn.addEventListener('mouseup', () => keys[keyName] = false);
        }

        bindTouchButton('btn-up', 'ArrowUp');
        bindTouchButton('btn-down', 'ArrowDown');
        bindTouchButton('btn-left', 'ArrowLeft');
        bindTouchButton('btn-right', 'ArrowRight');

        function update() {
            if (keys['ArrowUp'] || keys['w'] || keys['W']) myPlayer.y -= myPlayer.speed;
            if (keys['ArrowDown'] || keys['s'] || keys['S']) myPlayer.y += myPlayer.speed;
            if (keys['ArrowLeft'] || keys['a'] || keys['A']) myPlayer.x -= myPlayer.speed;
            if (keys['ArrowRight'] || keys['d'] || keys['D']) myPlayer.x += myPlayer.speed;

            // حدود الشاشة
            if (myPlayer.x < 0) myPlayer.x = 0;
            if (myPlayer.x > canvas.width - myPlayer.size) myPlayer.x = canvas.width - myPlayer.size;
            if (myPlayer.y < 0) myPlayer.y = 0;
            if (myPlayer.y > canvas.height - myPlayer.size) myPlayer.y = canvas.height - myPlayer.size;
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // رسم الشبكة في الخلفية
            ctx.strokeStyle = '#333';
            for (let i = 0; i < canvas.width; i += 40) {
                ctx.beginPath();
                ctx.moveTo(i, 0);
                ctx.lineTo(i, canvas.height);
                ctx.stroke();
            }

            // رسم اللاعب
            ctx.fillStyle = myPlayer.color;
            ctx.fillRect(myPlayer.x, myPlayer.y, myPlayer.size, myPlayer.size);

            // رسم اسم اللاعب
            ctx.fillStyle = 'white';
            ctx.font = '14px Tahoma';
            ctx.textAlign = 'center';
            ctx.fillText(myPlayer.name, myPlayer.x + (myPlayer.size / 2), myPlayer.y - 10);
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }
    </script>
</body>
</html>
