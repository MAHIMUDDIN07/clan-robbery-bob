[Uploading index (3).htm
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clan Robbery Bob</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #222;
            font-family: Arial, sans-serif;
        }
        #gameCanvas {
            display: block;
            background-color: #333;
            margin: 20px auto;
            border: 2px solid #555;
        }
        #gameUI {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-size: 18px;
        }
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
        }
        button {
            padding: 10px 20px;
            font-size: 18px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 10px;
        }
        button:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>
    <div id="gameUI">
        <div>Gold: <span id="gold">0</span></div>
        <div>Lives: <span id="lives">3</span></div>
        <div>Level: <span id="level">1</span></div>
    </div>
    
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    
    <div id="startScreen">
        <h1>Clan Robbery Bob</h1>
        <p>Steal gold from enemy clans without getting caught!</p>
        <button id="startButton">Start Game</button>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const goldDisplay = document.getElementById('gold');
        const livesDisplay = document.getElementById('lives');
        const levelDisplay = document.getElementById('level');
        const startScreen = document.getElementById('startScreen');
        const startButton = document.getElementById('startButton');
        
        let gold = 0;
        let lives = 3;
        let level = 1;
        let gameRunning = false;
        
        const player = {
            x: 50,
            y: 300,
            width: 30,
            height: 50,
            speed: 5,
            color: '#3498db',
            direction: 1
        };
        
        let enemies = [];
        let coins = [];
        let walls = [];
        
        function initLevel() {
            enemies = [];
            coins = [];
            walls = [];
            for (let i = 0; i < 5; i++) {
                walls.push({
                    x: Math.random() * (canvas.width - 100) + 50,
                    y: Math.random() * (canvas.height - 100) + 50,
                    width: 80,
                    height: 20,
                    color: '#7f8c8d'
                });
            }
            for (let i = 0; i < 5 + level * 2; i++) {
                coins.push({
                    x: Math.random() * (canvas.width - 30) + 15,
                    y: Math.random() * (canvas.height - 30) + 15,
                    radius: 15,
                    color: '#f1c40f',
                    value: 10 * level
                });
            }
            for (let i = 0; i < level; i++) {
                enemies.push({
                    x: Math.random() * (canvas.width - 40) + 20,
                    y: Math.random() * (canvas.height - 40) + 20,
                    width: 40,
                    height: 40,
                    speed: 2 + level * 0.5,
                    color: '#e74c3c',
                    directionX: Math.random() > 0.5 ? 1 : -1,
                    directionY: Math.random() > 0.5 ? 1 : -1
                });
            }
            player.x = 50;
            player.y = 300;
        }

        function startGame() {
            gold = 0;
            lives = 3;
            level = 1;
            gameRunning = true;
            startScreen.style.display = 'none';
            initLevel();
            updateDisplay();
            gameLoop();
        }

        function updateDisplay() {
            goldDisplay.textContent = gold;
            livesDisplay.textContent = lives;
            levelDisplay.textContent = level;
        }

        function checkCollision(rect1, rect2) {
            return rect1.x < rect2.x + rect2.width &&
                   rect1.x + rect1.width > rect2.x &&
                   rect1.y < rect2.y + rect2.height &&
                   rect1.y + rect1.height > rect2.y;
        }

        function gameLoop() {
            if (!gameRunning) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (keys.ArrowLeft) {
                player.x -= player.speed;
                player.direction = -1;
            }
            if (keys.ArrowRight) {
                player.x += player.speed;
                player.direction = 1;
            }
            if (keys.ArrowUp) player.y -= player.speed;
            if (keys.ArrowDown) player.y += player.speed;

            player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));
            player.y = Math.max(0, Math.min(canvas.height - player.height, player.y));

            enemies.forEach(enemy => {
                enemy.x += enemy.speed * enemy.directionX;
                enemy.y += enemy.speed * enemy.directionY;
                if (enemy.x <= 0 || enemy.x >= canvas.width - enemy.width) enemy.directionX *= -1;
                if (enemy.y <= 0 || enemy.y >= canvas.height - enemy.height) enemy.directionY *= -1;

                walls.forEach(wall => {
                    if (checkCollision(enemy, wall)) {
                        enemy.directionX *= -1;
                        enemy.directionY *= -1;
                    }
                });

                if (checkCollision(enemy, player)) {
                    lives--;
                    updateDisplay();
                    if (lives <= 0) {
                        gameOver();
                    } else {
                        player.x = 50;
                        player.y = 300;
                    }
                }
            });

            for (let i = coins.length - 1; i >= 0; i--) {
                const coin = coins[i];
                const distance = Math.sqrt(
                    Math.pow(player.x + player.width/2 - coin.x, 2) + 
                    Math.pow(player.y + player.height/2 - coin.y, 2)
                );
                if (distance < player.width/2 + coin.radius) {
                    gold += coin.value;
                    coins.splice(i, 1);
                    updateDisplay();
                }
            }

            if (coins.length === 0) {
                level++;
                updateDisplay();
                initLevel();
            }

            walls.forEach(wall => {
                ctx.fillStyle = wall.color;
                ctx.fillRect(wall.x, wall.y, wall.width, wall.height);
            });

            coins.forEach(coin => {
                ctx.beginPath();
                ctx.arc(coin.x, coin.y, coin.radius, 0, Math.PI * 2);
                ctx.fillStyle = coin.color;
                ctx.fill();
                ctx.closePath();
            });

            enemies.forEach(enemy => {
                ctx.fillStyle = enemy.color;
                ctx.fillRect(enemy.x, enemy.y, enemy.width, enemy.height);
            });

            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);

            ctx.fillStyle = 'white';
            const eyeX = player.direction > 0 ? player.x + player.width - 10 : player.x + 10;
            ctx.beginPath();
            ctx.arc(eyeX, player.y + 15, 5, 0, Math.PI * 2);
            ctx.fill();
            ctx.closePath();

            requestAnimationFrame(gameLoop);
        }

        function gameOver() {
            gameRunning = false;
            startScreen.style.display = 'flex';
            startScreen.innerHTML = `
                <h1>Game Over</h1>
                <p>You collected ${gold} gold and reached level ${level}!</p>
                <button id="restartButton">Play Again</button>
            `;
            document.getElementById('restartButton').addEventListener('click', startGame);
        }

        const keys = {
            ArrowUp: false,
            ArrowDown: false,
            ArrowLeft: false,
            ArrowRight: false
        };

        window.addEventListener('keydown', e => {
            if (keys.hasOwnProperty(e.key)) {
                keys[e.key] = true;
            }
        });

        window.addEventListener('keyup', e => {
            if (keys.hasOwnProperty(e.key)) {
                keys[e.key] = false;
            }
        });

        startButton.addEventListener('click', startGame);
    </script>
</body>
</html>
l…]()
