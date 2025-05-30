<!DOCTYPE html>
<html>
<head>
    <title>Flappy Bird</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }
        
        #game-container {
            position: relative;
        }
        
        #game-canvas {
            border: 2px solid #333;
            background-color: skyblue;
        }
        
        #start-screen, #game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
        }
        
        #game-over {
            display: none;
        }
        
        button {
            padding: 10px 20px;
            font-size: 18px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 20px;
        }
        
        button:hover {
            background-color: #45a049;
        }
        
        h1 {
            color: #ffeb3b;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="game-canvas" width="400" height="600"></canvas>
        
        <div id="start-screen">
            <h1>Flappy Bird</h1>
            <p>Press SPACE or click to jump</p>
            <button id="start-btn">Start Game</button>
        </div>
        
        <div id="game-over">
            <h1>Game Over</h1>
            <p>Score: <span id="score">0</span></p>
            <button id="restart-btn">Play Again</button>
        </div>
    </div>

    <script>
        // Game variables
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('start-screen');
        const gameOverScreen = document.getElementById('game-over');
        const scoreElement = document.getElementById('score');
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');

        // Bird properties
        const bird = {
            x: 50,
            y: canvas.height / 2,
            width: 40,
            height: 30,
            velocity: 0,
            gravity: 0.5,
            jump: -8
        };

        // Game state
        let pipes = [];
        let score = 0;
        let gameRunning = false;
        let animationId;
        const pipeGap = 150;
        const pipeWidth = 60;
        const pipeSpeed = 2;

        // Event listeners
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') jump();
        });
        canvas.addEventListener('click', jump);

        // Start game function
        function startGame() {
            // Reset game state
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            score = 0;
            gameRunning = true;
            
            // Hide screens
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            
            // Start game loop
            gameLoop();
        }

        // End game function
        function endGame() {
            gameRunning = false;
            scoreElement.textContent = score;
            gameOverScreen.style.display = 'flex';
            cancelAnimationFrame(animationId);
        }

        // Make bird jump
        function jump() {
            if (gameRunning) {
                bird.velocity = bird.jump;
            }
        }

        // Create new pipes
        function createPipe() {
            const minHeight = 50;
            const maxHeight = canvas.height - pipeGap - minHeight;
            const height = Math.floor(Math.random() * (maxHeight - minHeight + 1)) + minHeight;

            pipes.push({
                x: canvas.width,
                height: height,
                passed: false
            });
        }

        // Update game state
        function update() {
            // Update bird position
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            // Check for collisions with ground or ceiling
            if (bird.y + bird.height > canvas.height || bird.y < 0) {
                endGame();
            }

            // Create new pipes periodically
            if (gameRunning && frames % 100 === 0) {
                createPipe();
            }

            // Update pipes
            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= pipeSpeed;

                // Check for collisions with pipes
                if (
                    bird.x + bird.width > pipes[i].x &&
                    bird.x < pipes[i].x + pipeWidth &&
                    (bird.y < pipes[i].height || bird.y + bird.height > pipes[i].height + pipeGap)
                ) {
                    endGame();
                }

                // Increase score when passing a pipe
                if (!pipes[i].passed && bird.x > pipes[i].x + pipeWidth) {
                    pipes[i].passed = true;
                    score++;
                }

                // Remove pipes that are off screen
                if (pipes[i].x + pipeWidth < 0) {
                    pipes.splice(i, 1);
                }
            }
        }

        // Draw everything
        function draw() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw sky
            ctx.fillStyle = 'skyblue';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw bird
            ctx.fillStyle = 'yellow';
            ctx.fillRect(bird.x, bird.y, bird.width, bird.height);

            // Draw pipes
            ctx.fillStyle = 'green';
            pipes.forEach(pipe => {
                // Top pipe
                ctx.fillRect(pipe.x, 0, pipeWidth, pipe.height);
                // Bottom pipe
                ctx.fillRect(pipe.x, pipe.height + pipeGap, pipeWidth, canvas.height - pipe.height - pipeGap);
            });

            // Draw score
            ctx.fillStyle = 'black';
            ctx.font = '30px Arial';
            ctx.fillText(score, canvas.width / 2, 50);
        }

        // Frame counter
        let frames = 0;

        // Main game loop
        function gameLoop() {
            frames++;
            update();
            draw();
            
            if (gameRunning) {
                animationId = requestAnimationFrame(gameLoop);
            }
        }
    </script>
</body>
</html>
