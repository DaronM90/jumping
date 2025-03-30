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
    </style>
</head>
<body>
    <div id="score">Score: 0</div>
    <canvas id="gameCanvas"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
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
        const playerImage = new Image();
        playerImage.src = 'https://i.imgur.com/OZzJ6KA.png';
        
        function generatePlatform(y) {
            return {
                x: Math.random() * (canvas.width - 70),
                y: y,
                width: 70,
                height: 10
            };
        }
        
        for (let i = 0; i < 10; i++) {
            platforms.push(generatePlatform(i * (canvas.height / 10)));
        }

        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            player.dy += gravity;
            player.y += player.dy;
            player.x += player.dx;
            
            if (player.y > canvas.height - player.height) {
                player.dy = jumpPower;
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
            
            // Удаление платформ, которые вышли за нижний край экрана
            while (platforms.length > 0 && platforms[0].y > cameraY + canvas.height) {
                platforms.shift();
            }
            
            // Генерация новых платформ выше текущего верхнего уровня
            let highestY = platforms[platforms.length - 1].y;
            while (platforms.length < 10 || highestY > cameraY - canvas.height / 10) {
                highestY -= canvas.height / 10;
                platforms.push(generatePlatform(highestY));
            }
            
            ctx.restore();
            requestAnimationFrame(gameLoop);
        }

        function handleMove(event) {
            const touchX = event.touches ? event.touches[0].clientX : event.clientX;
            player.dx = (touchX < canvas.width / 2) ? -moveSpeed : moveSpeed;
        }

        function stopMove() {
            player.dx = 0;
        }

        canvas.addEventListener('mousedown', handleMove);
        canvas.addEventListener('mouseup', stopMove);
        canvas.addEventListener('touchstart', handleMove);
        canvas.addEventListener('touchend', stopMove);
        
        gameLoop();
    </script>
</body>
</html>
