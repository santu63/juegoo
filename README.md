<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pong Retro</title>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(45deg, #1a0033, #330066);
            overflow: hidden;
            flex-direction: column;
            touch-action: none;
            font-family: 'Press Start 2P', cursive;
        }
        #gameCanvas {
            border: 4px solid #ffcc00;
            background: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAQAAAAECAYAAACp8Z5+AAAACXBIWXMAAAsTAAALEwEAmpwYAAAAJ0lEQVQImWNgYGBg+H///w8DAwMDAwPD/////z8DAwPD/////z8DAwMAhFoCrL6eH1AAAAAASUVORK5CYII=');
            box-shadow: 0 0 20px rgba(255, 204, 0, 0.8);
            display: none; /* Oculto al inicio */
        }
        #score {
            position: absolute;
            top: 20px;
            left: 20px;
            color: #00ffcc;
            font-size: 24px;
            text-shadow: 2px 2px #ff00ff, -2px -2px #000;
        }
        #highScore {
            position: absolute;
            top: 50px;
            left: 20px;
            color: #ff00ff;
            font-size: 24px;
            text-shadow: 2px 2px #00ffcc, -2px -2px #000;
        }
        #gameOver {
            position: absolute;
            color: #ff4444;
            font-size: 50px;
            text-shadow: 3px 3px #000, -3px -3px #fff;
            display: none;
            animation: blink 0.5s infinite;
        }
        .controls {
            margin-top: 20px;
            display: flex;
            gap: 30px;
        }
        .control-btn {
            width: 70px;
            height: 70px;
            background: radial-gradient(circle, #ff4444, #cc0000);
            border: 4px solid #fff;
            border-radius: 50%;
            color: #fff;
            font-size: 30px;
            text-align: center;
            line-height: 70px;
            user-select: none;
            cursor: pointer;
            box-shadow: 0 0 10px #ff4444;
            transition: transform 0.1s;
        }
        .control-btn:active {
            transform: scale(0.9);
        }
        #restartBtn {
            width: 120px;
            height: 50px;
            background: linear-gradient(#00ff00, #00cc00);
            border: 4px solid #fff;
            border-radius: 10px;
            color: #fff;
            font-size: 20px;
            cursor: pointer;
            box-shadow: 0 0 15px #00ff00;
            transition: transform 0.1s;
            display: none; /* Oculto hasta Game Over */
        }
        #restartBtn:hover {
            transform: scale(1.05);
        }
        #loadingScreen, #startScreen {
            position: absolute;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            background: linear-gradient(45deg, #1a0033, #330066);
            color: #ffcc00;
            font-size: 40px;
            text-shadow: 3px 3px #ff00ff, -3px -3px #00ffcc;
        }
        #startBtn {
            width: 150px;
            height: 60px;
            background: linear-gradient(#ffcc00, #ff9900);
            border: 4px solid #fff;
            border-radius: 10px;
            color: #fff;
            font-size: 24px;
            cursor: pointer;
            box-shadow: 0 0 15px #ffcc00;
            transition: transform 0.1s;
            margin-top: 20px;
        }
        #startBtn:hover {
            transform: scale(1.05);
        }
        @keyframes blink {
            50% { opacity: 0; }
        }
        @media (max-width: 600px) {
            #gameCanvas { width: 100%; height: auto; }
            #score, #highScore { font-size: 18px; }
            #gameOver { font-size: 40px; }
        }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
</head>
<body>
    <div id="loadingScreen">Cargando...</div>
    <div id="startScreen" style="display: none;">
        <div>Pong Retro</div>
        <button id="startBtn">Iniciar</button>
    </div>
    <div id="score" style="display: none;">Puntuación: 0</div>
    <div id="highScore" style="display: none;">Récord: 0</div>
    <canvas id="gameCanvas" width="600" height="400"></canvas>
    <div id="gameOver">Game Over!</div>
    <div class="controls" style="display: none;">
        <div class="control-btn" id="leftBtn">←</div>
        <div class="control-btn" id="rightBtn">→</div>
    </div>
    <button id="restartBtn">Reiniciar</button>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
        const highScoreDisplay = document.getElementById('highScore');
        const gameOverDisplay = document.getElementById('gameOver');
        const leftBtn = document.getElementById('leftBtn');
        const rightBtn = document.getElementById('rightBtn');
        const restartBtn = document.getElementById('restartBtn');
        const loadingScreen = document.getElementById('loadingScreen');
        const startScreen = document.getElementById('startScreen');
        const startBtn = document.getElementById('startBtn');
        const controls = document.querySelector('.controls');

        const paddle = {
            width: 100,
            height: 10,
            x: canvas.width / 2 - 50,
            y: canvas.height - 20,
            speed: 5
        };

        const ball = {
            radius: 10,
            x: canvas.width / 2,
            y: canvas.height / 2,
            speedX: 2.5,
            speedY: -2.5,
            baseSpeed: 2.5
        };

        let score = 0;
        let highScore = localStorage.getItem('highScore') ? parseInt(localStorage.getItem('highScore')) : 0;
        let gameOver = false;
        let movingLeft = false;
        let movingRight = false;
        let lastUpdate = 0;
        const moveDelay = 16;

        highScoreDisplay.textContent = `Récord: ${highScore}`;

        // Simular pantalla de carga
        setTimeout(() => {
            loadingScreen.style.display = 'none';
            startScreen.style.display = 'flex';
        }, 1500);

        // Iniciar juego
        startBtn.addEventListener('click', () => {
            startScreen.style.display = 'none';
            canvas.style.display = 'block';
            scoreDisplay.style.display = 'block';
            highScoreDisplay.style.display = 'block';
            controls.style.display = 'flex';
            requestAnimationFrame(gameLoop);
        });

        // Controles para PC (ratón)
        canvas.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            paddle.x = e.clientX - rect.left - paddle.width / 2;
            if (paddle.x < 0) paddle.x = 0;
            if (paddle.x + paddle.width > canvas.width) paddle.x = canvas.width - paddle.width;
        });

        // Controles táctiles
        canvas.addEventListener('touchmove', (e) => {
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const touch = e.touches[0];
            paddle.x = touch.clientX - rect.left - paddle.width / 2;
            if (paddle.x < 0) paddle.x = 0;
            if (paddle.x + paddle.width > canvas.width) paddle.x = canvas.width - paddle.width;
        });

        leftBtn.addEventListener('touchstart', () => movingLeft = true);
        leftBtn.addEventListener('touchend', () => movingLeft = false);
        rightBtn.addEventListener('touchstart', () => movingRight = true);
        rightBtn.addEventListener('touchend', () => movingRight = false);

        leftBtn.addEventListener('mousedown', () => movingLeft = true);
        leftBtn.addEventListener('mouseup', () => movingLeft = false);
        rightBtn.addEventListener('mousedown', () => movingRight = true);
        rightBtn.addEventListener('mouseup', () => movingRight = false);

        restartBtn.addEventListener('click', restartGame);

        function restartGame() {
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('highScore', highScore);
                highScoreDisplay.textContent = `Récord: ${highScore}`;
            }
            score = 0;
            scoreDisplay.textContent = `Puntuación: ${score}`;
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.speedX = ball.baseSpeed;
            ball.speedY = -ball.baseSpeed;
            paddle.x = canvas.width / 2 - 50;
            gameOver = false;
            gameOverDisplay.style.display = 'none';
            restartBtn.style.display = 'none';
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#ff00ff';
            ctx.fillRect(paddle.x, paddle.y, paddle.width, paddle.height);
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fillStyle = '#00ffcc';
            ctx.fill();
            ctx.closePath();
        }

        function update(timestamp) {
            if (gameOver) return;

            if (timestamp - lastUpdate < moveDelay) return;
            lastUpdate = timestamp;

            if (movingLeft && paddle.x > 0) paddle.x -= paddle.speed;
            if (movingRight && paddle.x + paddle.width < canvas.width) paddle.x += paddle.speed;

            ball.x += ball.speedX;
            ball.y += ball.speedY;

            if (ball.x - ball.radius < 0 || ball.x + ball.radius > canvas.width) {
                ball.speedX = -ball.speedX;
                score++;
                updateBallSpeed();
                scoreDisplay.textContent = `Puntuación: ${score}`;
            }
            if (ball.y - ball.radius < 0) {
                ball.speedY = -ball.speedY;
                score++;
                updateBallSpeed();
                scoreDisplay.textContent = `Puntuación: ${score}`;
            }

            if (
                ball.y + ball.radius > paddle.y &&
                ball.y - ball.radius < paddle.y + paddle.height &&
                ball.x > paddle.x &&
                ball.x < paddle.x + paddle.width
            ) {
                ball.speedY = -ball.speedY;
            }

            if (ball.y + ball.radius > canvas.height) {
                gameOver = true;
                gameOverDisplay.style.display = 'block';
                restartBtn.style.display = 'block';
            }
        }

        function updateBallSpeed() {
            const speedIncrease = Math.min(ball.baseSpeed + (score * 0.05), 8);
            ball.speedX = Math.sign(ball.speedX) * speedIncrease;
            ball.speedY = Math.sign(ball.speedY) * speedIncrease;
        }

        function gameLoop(timestamp) {
            draw();
            update(timestamp);
            requestAnimationFrame(gameLoop);
        }
    </script>
</body>
</html>
