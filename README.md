<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة الطائر المرفرف</title>
    <meta name="google-site-verification" content="l6FSehriSN6V1o0fUyEB7SLy7Ym0pepQBHWJM0tbyV0" />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Luckiest+Guy&family=Press+Start+2P&family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');
        @import url('https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap');

        :root {
            --bg-color-start: #3498db;
            --bg-color-end: #8e44ad;
            --bird-color: #f1c40f;
            --pipe-color: #2d3436;
            --message-box-bg: rgba(44, 62, 80, 0.9);
            --message-box-border: #ecf0f1;
            --text-color: #ecf0f1;
            --button-color: #27ae60;
            --button-hover-color: #2ecc71;
            --font-family: 'Tajawal', sans-serif;
        }

        body {
            font-family: var(--font-family);
            margin: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(135deg, var(--bg-color-start), var(--bg-color-end));
            color: var(--text-color);
            overflow: hidden;
            text-align: center;
            transition: background 0.5s ease-in-out;
        }

        .container {
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background: rgba(52, 73, 94, 0.7);
            padding: 2rem;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            transition: transform 0.3s ease-in-out;
            border: 2px solid #5d6d7e;
        }

        #gameCanvas {
            background-color: transparent;
            border-radius: 10px;
            box-shadow: inset 0 0 15px rgba(0, 0, 0, 0.3);
            border: 2px solid #34495e;
        }

        .game-info {
            margin-top: 1.5rem;
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 400px;
            padding: 0 1rem;
        }

        .score, .high-score {
            font-size: 1.8rem;
            font-weight: bold;
            color: var(--text-color);
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        .message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: var(--message-box-bg);
            border-radius: 20px;
            padding: 3rem;
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.6);
            text-align: center;
            z-index: 1000;
            transition: all 0.4s ease-in-out;
            border: 3px solid var(--message-box-border);
            animation: fadeIn 0.5s ease-out;
        }

        .message-box h1 {
            font-size: 2.5rem;
            margin-top: 0;
            color: #e74c3c;
            text-shadow: 2px 2px 3px rgba(0, 0, 0, 0.3);
        }

        .message-box p {
            font-size: 1.5rem;
            margin: 1.5rem 0;
            color: #bdc3c7;
        }

        .message-box button {
            background-color: var(--button-color);
            color: white;
            border: none;
            padding: 1rem 2rem;
            font-size: 1.2rem;
            border-radius: 10px;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.2s, box-shadow 0.2s;
            box-shadow: 0 6px 10px rgba(0, 0, 0, 0.3);
            font-family: var(--font-family);
        }

        .message-box button:hover {
            background-color: var(--button-hover-color);
            transform: translateY(-4px);
            box-shadow: 0 8px 15px rgba(0, 0, 0, 0.4);
        }
        
        .message-box button:active {
            transform: translateY(0);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
        }

        .hidden {
            display: none;
            opacity: 0;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translate(-50%, -50%) scale(0.8); }
            to { opacity: 1; transform: translate(-50%, -50%) scale(1); }
        }
    </style>
</head>
<body>
    <div class="container">
        <canvas id="gameCanvas"></canvas>
    </div>
    <div class="game-info">
        <div class="score">النتيجة: <span id="currentScore">0</span></div>
        <div class="high-score">أعلى نتيجة: <span id="highScore">0</span></div>
    </div>
    
    <div id="startScreen" class="message-box">
        <h1>مرحبا بك في لعبة الطائر المرفرف!</h1>
        <p>انقر أو اضغط على الشاشة للعب</p>
        <button id="startButton">بدء اللعبة</button>
    </div>

    <div id="gameOverScreen" class="message-box hidden">
        <h1>انتهت اللعبة!</h1>
        <p>النتيجة النهائية: <span id="finalScore">0</span></p>
        <button id="restartButton">إعادة اللعب</button>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const body = document.body;

        let isGameRunning = false;
        let score = 0;
        let highScore = localStorage.getItem('flappyHighScore') || 0;
        let currentPipeSpeed = 2;

        const bird = {
            x: 50,
            y: 0, 
            radius: 12,
            gravity: 0.2,
            velocity: 0,
            lift: -4.5,
            color: '#f1c40f',
            strokeColor: '#e67e22',
            strokeWidth: 2,
            shape: 'circle'
        };

        let pipes = [];
        const pipeWidth = 45;
        const pipeGap = 130; 
        let frameCount = 0;
        const pipeInterval = 90; 
        const pipeColor = '#2d3436';

        const currentScoreElement = document.getElementById('currentScore');
        const highScoreElement = document.getElementById('highScore');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const finalScoreElement = document.getElementById('finalScore');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');

        const colorPalettes = {
            'default': ['#3498db', '#8e44ad'],
            '5': ['#f1c40f', '#27ae60'], // Star theme
            '10': ['#3498db', '#2ecc71'], // Speed up 1
            '15': ['#f39c12', '#c0392b'], // Diamond theme
            '20': ['#e74c3c', '#9b59b6'], // Stage 2
            '30': ['#9b59b6', '#3498db'], // Stage 3
            '50': ['#2c3e50', '#7f8c8d'], // Final stage
            '200': ['#FFD700', '#DAA520'] // Legendary theme
        };

        let particles = [];
        let continuousParticles = [];

        function createParticles(num, color) {
            particles = [];
            for (let i = 0; i < num; i++) {
                particles.push({
                    x: bird.x,
                    y: bird.y,
                    size: Math.random() * 3 + 1,
                    speedX: Math.random() * 4 - 2,
                    speedY: Math.random() * 4 - 2,
                    color: color
                });
            }
        }
        
        function updateParticles() {
            for (let i = particles.length - 1; i >= 0; i--) {
                particles[i].x += particles[i].speedX;
                particles[i].y += particles[i].speedY;
                particles[i].size -= 0.05;
                if (particles[i].size <= 0.2) {
                    particles.splice(i, 1);
                }
            }
        }
        
        function drawParticles() {
            for (const particle of particles) {
                ctx.fillStyle = particle.color;
                ctx.beginPath();
                ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2);
                ctx.fill();
            }
        }
        
        function createContinuousParticles() {
            if (isGameRunning) {
                const particle = {
                    x: bird.x,
                    y: bird.y,
                    size: Math.random() * 2 + 1,
                    speedX: Math.random() * 0.5 - 0.25,
                    speedY: Math.random() * 0.5 - 0.25,
                    color: 'rgba(255, 255, 255, ' + (Math.random() * 0.5 + 0.5) + ')'
                };
                continuousParticles.push(particle);
            }
        }

        function updateContinuousParticles() {
            for (let i = continuousParticles.length - 1; i >= 0; i--) {
                continuousParticles[i].x -= currentPipeSpeed + continuousParticles[i].speedX;
                continuousParticles[i].y += continuousParticles[i].speedY;
                continuousParticles[i].size -= 0.02;
                if (continuousParticles[i].size <= 0.1 || continuousParticles[i].x < 0) {
                    continuousParticles.splice(i, 1);
                }
            }
        }

        function drawContinuousParticles() {
            for (const particle of continuousParticles) {
                ctx.fillStyle = particle.color;
                ctx.beginPath();
                ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        function drawBird() {
            if (bird.shape === 'circle') {
                ctx.beginPath();
                ctx.arc(bird.x, bird.y, bird.radius, 0, Math.PI * 2);
                ctx.fillStyle = bird.color;
                ctx.strokeStyle = bird.strokeColor;
                ctx.lineWidth = bird.strokeWidth;
                ctx.fill();
                ctx.stroke();
                ctx.closePath();
            } else if (bird.shape.startsWith('star')) {
                const outerRadius = bird.radius * (bird.shape === 'star-large' ? 2 : 1.5);
                const innerRadius = bird.radius * 0.7;
                const points = 5;
                let rotation = Math.PI / 2 * 3;
                let x = bird.x;
                let y = bird.y;
                let step = Math.PI / points;
                ctx.beginPath();
                ctx.moveTo(x, y - outerRadius);
                for (let i = 0; i < points; i++) {
                    ctx.lineTo(x + Math.cos(rotation) * outerRadius, y + Math.sin(rotation) * outerRadius);
                    rotation += step;
                    ctx.lineTo(x + Math.cos(rotation) * innerRadius, y + Math.sin(rotation) * innerRadius);
                    rotation += step;
                }
                ctx.lineTo(x, y - outerRadius);
                ctx.closePath();
                ctx.fillStyle = bird.color;
                ctx.strokeStyle = bird.strokeColor;
                ctx.lineWidth = bird.strokeWidth;
                ctx.fill();
                ctx.stroke();
            } else if (bird.shape === 'diamond') {
                const size = bird.radius * 2;
                ctx.beginPath();
                ctx.moveTo(bird.x, bird.y - size / 1.5);
                ctx.lineTo(bird.x + size, bird.y);
                ctx.lineTo(bird.x, bird.y + size / 1.5);
                ctx.lineTo(bird.x - size, bird.y);
                ctx.closePath();
                ctx.fillStyle = bird.color;
                ctx.strokeStyle = bird.strokeColor;
                ctx.lineWidth = bird.strokeWidth;
                ctx.fill();
                ctx.stroke();
            } else if (bird.shape === 'super') {
                 ctx.beginPath();
                 ctx.arc(bird.x, bird.y, bird.radius * 1.5, 0, Math.PI * 2);
                 ctx.fillStyle = '#663399'; // Super purple
                 ctx.strokeStyle = '#DAA520';
                 ctx.lineWidth = bird.strokeWidth + 2;
                 ctx.fill();
                 ctx.stroke();
                 ctx.closePath();
            } else if (bird.shape === 'super-super') {
                 ctx.beginPath();
                 ctx.arc(bird.x, bird.y, bird.radius * 1.8, 0, Math.PI * 2);
                 ctx.fillStyle = '#FF4500'; // Super-super orange-red
                 ctx.strokeStyle = '#DAA520';
                 ctx.lineWidth = bird.strokeWidth + 2;
                 ctx.fill();
                 ctx.stroke();
                 ctx.closePath();
            } else if (bird.shape === 'legendary') {
                 ctx.beginPath();
                 ctx.arc(bird.x, bird.y, bird.radius * 2, 0, Math.PI * 2);
                 ctx.fillStyle = '#FFD700'; // Gold
                 ctx.strokeStyle = '#DAA520';
                 ctx.lineWidth = bird.strokeWidth + 2;
                 ctx.fill();
                 ctx.stroke();
                 ctx.closePath();
            }
        }

        function updateBird() {
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            if (bird.y + bird.radius > canvas.height || bird.y - bird.radius < 0) {
                gameOver();
            }
        }

        function drawPipe(x, height) {
            ctx.fillStyle = pipeColor;
            ctx.fillRect(x, 0, pipeWidth, height);
            ctx.fillRect(x, height + pipeGap, pipeWidth, canvas.height - (height + pipeGap));
        }

        function updatePipes() {
            frameCount++;

            if (frameCount % pipeInterval === 0) {
                const pipeHeight = Math.random() * (canvas.height - pipeGap - 80) + 40;
                pipes.push({ x: canvas.width, height: pipeHeight });
            }

            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= currentPipeSpeed;

                if (
                    bird.x + bird.radius > pipes[i].x &&
                    bird.x - bird.radius < pipes[i].x + pipeWidth
                ) {
                    if (
                        bird.y - bird.radius < pipes[i].height ||
                        bird.y + bird.radius > pipes[i].height + pipeGap
                    ) {
                        gameOver();
                    }
                }

                if (pipes[i].x + pipeWidth < bird.x && !pipes[i].passed) {
                    score++;
                    pipes[i].passed = true;
                    currentScoreElement.textContent = score;
                    
                    if (score === 5) {
                        createParticles(50, '#FFD700');
                        bird.shape = 'star';
                        bird.color = '#FFD700';
                        bird.strokeColor = '#DAA520';
                        updateBackground();
                    } else if (score === 10) {
                        createParticles(75, '#FFFFFF');
                        bird.shape = 'star-large';
                        bird.color = '#FFFFFF';
                        bird.strokeColor = '#f1c40f';
                        updateBackground();
                    } else if (score === 15) {
                        createParticles(100, '#00FFFF');
                        bird.shape = 'diamond';
                        bird.color = '#00FFFF';
                        bird.strokeColor = '#008B8B';
                        updateBackground();
                    } else if (score === 20) {
                        createParticles(125, '#FF1493');
                        bird.shape = 'super';
                        bird.color = '#FF1493';
                        bird.strokeColor = '#8A2BE2';
                        updateBackground();
                    } else if (score === 30) {
                        createParticles(150, '#FF0000');
                        bird.shape = 'super-super';
                        bird.color = '#FF0000';
                        bird.strokeColor = '#B22222';
                        updateBackground();
                    } else if (score === 50) {
                        createParticles(200, '#32CD32');
                        bird.shape = 'legendary';
                        bird.color = '#32CD32';
                        bird.strokeColor = '#008000';
                        updateBackground();
                    }
                    
                    if (score >= 10 && score < 20) {
                        currentPipeSpeed = 2.5;
                    } else if (score >= 20 && score < 30) {
                        currentPipeSpeed = 3.0;
                    } else if (score >= 30 && score < 50) {
                        currentPipeSpeed = 3.5;
                    } else if (score >= 50 && score < 200) {
                        currentPipeSpeed = 4.0;
                    } else if (score >= 200) {
                        currentPipeSpeed = 5.0;
                    }
                }

                if (pipes[i].x + pipeWidth < 0) {
                    pipes.splice(i, 1);
                }
            }
        }

        function gameLoop() {
            if (!isGameRunning) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            updateBird();
            updatePipes();
            drawBird();
            for (const pipe of pipes) {
                drawPipe(pipe.x, pipe.height);
            }
            createContinuousParticles();
            updateParticles();
            drawParticles();
            updateContinuousParticles();
            drawContinuousParticles();

            requestAnimationFrame(gameLoop);
        }

        function birdJump() {
            if (isGameRunning) {
                bird.velocity = bird.lift;
            } else if (gameOverScreen.classList.contains('hidden')) {
                startGame();
            }
        }

        function showMessageBox(element) {
            element.classList.remove('hidden');
        }

        function hideMessageBox(element) {
            element.classList.add('hidden');
        }

        function startGame() {
            hideMessageBox(startScreen);
            hideMessageBox(gameOverScreen);
            isGameRunning = true;
            score = 0;
            currentPipeSpeed = 2;
            currentScoreElement.textContent = score;
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            frameCount = 0;
            bird.shape = 'circle';
            bird.color = '#f1c40f';
            bird.strokeColor = '#e67e22';
            updateBackground();
            gameLoop();
        }

        function gameOver() {
            isGameRunning = false;
            finalScoreElement.textContent = score;
            showMessageBox(gameOverScreen);
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('flappyHighScore', highScore);
                highScoreElement.textContent = highScore;
            }
        }

        function updateBackground() {
            let palette;
            if (score >= 50) {
                palette = colorPalettes['50'];
            } else if (score >= 30) {
                palette = colorPalettes['30'];
            } else if (score >= 20) {
                palette = colorPalettes['20'];
            } else if (score >= 15) {
                palette = colorPalettes['15'];
            } else if (score >= 10) {
                palette = colorPalettes['10'];
            } else if (score >= 5) {
                palette = colorPalettes['5'];
            } else {
                palette = colorPalettes['default'];
            }
            body.style.background = `linear-gradient(135deg, ${palette[0]}, ${palette[1]})`;
        }

        window.onload = function() {
            window.addEventListener('resize', resizeCanvas);
            window.addEventListener('mousedown', birdJump);
            window.addEventListener('touchstart', birdJump);
            window.addEventListener('keydown', (e) => {
                if (e.code === 'Space' || e.code === 'ArrowUp') {
                    birdJump();
                }
            });

            startButton.addEventListener('click', startGame);
            restartButton.addEventListener('click', startGame);

            resizeCanvas();
            highScoreElement.textContent = highScore;
            updateBackground();
        };

        function resizeCanvas() {
            canvas.width = Math.min(window.innerWidth * 0.8, 400);
            canvas.height = Math.min(window.innerHeight * 0.7, 500);
            bird.y = canvas.height / 2;
        }

    </script>
</body>
</html>
