<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Doodle Jump</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #87CEEB; font-family: Arial, sans-serif; }
        canvas { display: block; }
        #score { position: absolute; top: 10px; left: 10px; font-size: 20px; color: white; }
        #gameOver { 
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 20px;
            font-size: 24px;
            text-align: center;
        }
        #restartButton {
            margin-top: 10px;
            padding: 10px;
            background: red;
            color: white;
            border: none;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="score">Score: 0</div>
    <div id="gameOver">
        <p>Конец игры</p>
        <p id="finalScore"></p>
        <button id="restartButton">Начать заново</button>
    </div>
    <canvas id="gameCanvas"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
        const gameOverScreen = document.getElementById('gameOver');
        const finalScoreDisplay = document.getElementById('finalScore');
        const restartButton = document.getElementById('restartButton');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let player = { 
            x: canvas.width / 2, 
            y: canvas.height - 50, 
            width: 40, 
            height: 40, 
            dy: 0, 
            dx: 0, 
            score: 0, 
            maxY: canvas.height - 50, 
            spriteFrame: 0 
        };
        const gravity = 0.4;
        const jumpPower = -10;
        const moveSpeed = 5;
        let cameraY = canvas.height / 2;
        const platforms = [];
        const obstacles = [];
        const playerImage = new Image();
        playerImage.src = 'https://i.imgur.com/OZzJ6KA.png';
        let gameOver = false;
        
        function generatePlatform(y) {
            return {
                x: Math.random() * (canvas.width - 70),
                y: y,
                width: 70,
                height: 10
            };
        }
        
        function generateObstacle(y) {
            return {
                x: Math.random() * (canvas.width - 30),
                y: y,
                width: 30,
                height: 30,
                dy: 2 
            };
        }

        for (let i = 0; i < 10; i++) {
            platforms.push(generatePlatform(i * (canvas.height / 10)));
        }

        function gameLoop() {
            if (gameOver) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            player.dy += gravity;
            player.y += player.dy;
            player.x += player.dx;
            
            if (player.y > canvas.height - player.height) {
                endGame();
            }
            
            if (player.y < player.maxY) {
                player.maxY = player.y;
                player.score = Math.abs(Math.floor((canvas.height - player.maxY) / 10));
                scoreDisplay.textContent = 'Score: ' + player.score;
                cameraY = player.maxY - canvas.height / 2;
            }
            
            ctx.save();
            ctx.translate(0, -cameraY);
            
            player.spriteFrame = player.dy < 0 ? 1 : 0;
            ctx.drawImage(playerImage, player.spriteFrame * 40, 0, 40, 40, player.x, player.y, player.width, player.height);
            
            ctx.fillStyle = 'brown';
            platforms.forEach((platform, index) => {
                ctx.fillRect(platform.x, platform.y, platform.width, platform.height);
                if (
                    player.dy > 0 &&
                    player.y + player.height > platform.y &&
                    player.y + player.height < platform.y + platform.height &&
                    player.x < platform.x + platform.width &&
                    player.x + player.width > platform.x
                ) {
                    player.dy = jumpPower;
                }
            });
            
            ctx.fillStyle = 'red';
            obstacles.forEach((obstacle, index) => {
                obstacle.y += obstacle.dy;
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
                if (
                    player.x < obstacle.x + obstacle.width &&
                    player.x + player.width > obstacle.x &&
                    player.y < obstacle.y + obstacle.height &&
                    player.y + player.height > obstacle.y
                ) {
                    endGame();
                }
            });
            
            while (platforms.length > 0 && platforms[0].y > cameraY + canvas.height) {
                platforms.shift();
                obstacles.push(generateObstacle(cameraY - canvas.height / 10));
            }
            
            let highestY = platforms[platforms.length - 1].y;
            while (platforms.length < 10 || highestY > cameraY - canvas.height / 10) {
                highestY -= canvas.height / 10;
                platforms.push(generatePlatform(highestY));
            }
            
            ctx.restore();
            requestAnimationFrame(gameLoop);
        }

        function endGame() {
            gameOver = true;
            finalScoreDisplay.textContent = 'Итоговый счет: ' + player.score;
            gameOverScreen.style.display = 'block';
        }

        function restartGame() {
            location.reload();
        }

        restartButton.addEventListener('click', restartGame);
        
        gameLoop();
    </script>
</body>
</html>
