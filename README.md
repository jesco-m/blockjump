# blockjump
Block Jumper
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dinosaur Game</title>
    <style>
        body {
            background-color: #333;
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }
        #gameCanvas {
            position: relative;
            width: 100%;
            height: 400px;
            background: #555;
            display: none;
            overflow: hidden;
        }
        #score {
            position: absolute;
            right: 20px;
            top: 20px;
            color: white;
            font-size: 20px;
        }
        #jumpButton {
            position: absolute;
            bottom: 20px;
            left: calc(50% - 50px);
            padding: 10px 20px;
            background: red;
            color: white;
            border: none;
            font-size: 20px;
            cursor: pointer;
            display: none;
        }
        .cactus {
            position: absolute;
            bottom: 0;
            width: 20px;
            height: 40px;
            background-color: #39ff14;
        }
        .dinosaur {
            position: absolute;
            bottom: 0;
            left: 50px;
            width: 40px;
            height: 60px;
            background-color: white;
        }
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            color: white;
            font-size: 40px;
            display: none;
        }
        #tryAgain {
            margin-top: 20px;
            padding: 10px 20px;
            background: red;
            color: white;
            border: none;
            font-size: 20px;
            cursor: pointer;
        }
        #startButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 10px 20px;
            background-color: red;
            color: white;
            border: none;
            font-size: 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="gameCanvas">
        <div id="score">Distance: <span id="distance">0</span> px</div>
        <div class="dinosaur" id="dino"></div>
    </div>
    <button id="startButton">START GAME</button>
    <button id="jumpButton">JUMP</button>
    <div id="gameOver">
        GAME OVER
        <br>
        <button id="tryAgain">TRY AGAIN</button>
    </div>

    <script>
        let distance = 0;
        let isJumping = false;
        let gameActive = false;
        let gameSpeed = 5;
        let startTime;
        let gameLoop;
        let cactusSpawnInterval;
        let dinoBottom = 0;
        let dinoJumpSpeed = 10;
        let dinoGravity = 0.8;
        let dinoVelocity = 0;
        const maxCactusDistance = 150;
        const cactiPositions = [];
        const dino = document.getElementById('dino');
        const jumpButton = document.getElementById('jumpButton');
        const scoreDisplay = document.getElementById('distance');
        const gameOverDisplay = document.getElementById('gameOver');
        const tryAgainButton = document.getElementById('tryAgain');
        const startButton = document.getElementById('startButton');
        const gameCanvas = document.getElementById('gameCanvas');

        function randomColor() {
            return '#' + Math.floor(Math.random()*16777215).toString(16);
        }

        function checkCactusDistance(cactiPositions, newPosition, maxDistance) {
            const nearbyCacti = cactiPositions.filter(pos => Math.abs(pos - newPosition) <= maxDistance);
            return nearbyCacti.length < 2;
        }

        function createCactusPosition() {
            let xPosition = gameCanvas.offsetWidth;
            while (true) {
                const newPosition = xPosition + Math.floor(Math.random() * 51) + 30;
                if (checkCactusDistance(cactiPositions, newPosition, maxCactusDistance)) {
                    cactiPositions.push(newPosition);
                    return newPosition;
                }
                xPosition = newPosition;
            }
        }

        function createCactus() {
            if (!gameActive) return;
            const xPosition = createCactusPosition();
            const cactus = document.createElement('div');
            cactus.classList.add('cactus');
            cactus.style.left = xPosition + 'px';
            gameCanvas.appendChild(cactus);
        }

        function spawnCactiGroup() {
            if (!gameActive) return;
            const groupSize = Math.floor(Math.random() * 2) + 1;
            for (let i = 0; i < groupSize; i++) {
                createCactus();
            }
        }

        startButton.onclick = () => {
            startGame();
            startButton.style.display = 'none';
            jumpButton.style.display = 'block';
            gameCanvas.style.display = 'block';
            startTime = Date.now();
            spawnCactiGroup();
            cactusSpawnInterval = setInterval(spawnCactiGroup, 2000);
            gameLoop = setInterval(updateGame, 20);
        };

        jumpButton.onclick = jump;
        document.addEventListener('keydown', (event) => {
            if (event.code === 'Space') jump();
        });

        function startGame() {
            distance = 0;
            isJumping = false;
            gameActive = true;
            gameSpeed = 5;
            dinoBottom = 0;
            dinoVelocity = 0;
            scoreDisplay.innerText = distance;
            gameOverDisplay.style.display = 'none';
            jumpButton.disabled = false;
            dino.style.bottom = '0px';
        }

        function jump() {
            if (gameActive && (dinoBottom === 0 || dinoBottom === 40)) {
                isJumping = true;
                dinoVelocity = dinoJumpSpeed;
                dino.style.backgroundColor = randomColor();
            }
        }

        function updateDinoPosition() {
            dinoBottom += dinoVelocity;
            dinoVelocity -= dinoGravity;

            if (dinoBottom <= 0) {
                dinoBottom = 0;
                dinoVelocity = 0;
                isJumping = false;
            }

            dino.style.bottom = dinoBottom + 'px';
        }

        function updateGame() {
            if (!gameActive) return;

            updateGameSpeed();
            moveCacti();
            updateDinoPosition();
            updateDistance();
            checkCollisions();
        }

        function moveCacti() {
            const cacti = document.querySelectorAll('.cactus');
            cacti.forEach(cactus => {
                let cactusPosition = parseInt(cactus.style.left);
                if (cactusPosition < -20) {
                    cactus.remove();
                    cactiPositions.shift();
                } else {
                    cactus.style.left = (cactusPosition - gameSpeed) + 'px';
                }
            });
        }

        function updateDistance() {
            distance += gameSpeed / 5;
            scoreDisplay.innerText = Math.floor(distance);
        }

        function updateGameSpeed() {
            const currentTime = Date.now();
            const elapsedMinutes = (currentTime - startTime) / 60000;
            
            if (elapsedMinutes >= 4) {
                gameSpeed = 20;
            } else if (elapsedMinutes >= 2) {
                gameSpeed = 10;
            } else {
                gameSpeed = 5 + (elapsedMinutes / 2) * 5;
            }
        }

        function checkCollisions() {
            const cacti = document.querySelectorAll('.cactus');
            const dinoRect = dino.getBoundingClientRect();

            cacti.forEach(cactus => {
                const cactusRect = cactus.getBoundingClientRect();
                if (dinoRect.right > cactusRect.left && 
                    dinoRect.left < cactusRect.right) {
                    if (dinoRect.bottom > cactusRect.top && dinoRect.top < cactusRect.top) {
                        gameOver();
                    } else if (dinoRect.bottom <= cactusRect.top && dinoRect.bottom > cactusRect.top - 10) {
                        dinoBottom = 40;
                        dino.style.bottom = dinoBottom + 'px';
                        dinoVelocity = 0;
                        isJumping = false;
                    }
                }
            });
        }

        function gameOver() {
            gameActive = false;
            gameOverDisplay.style.display = 'block';
            jumpButton.disabled = true;
            clearInterval(gameLoop);
            clearInterval(cactusSpawnInterval);
        }

        tryAgainButton.onclick = () => {
            document.querySelectorAll('.cactus').forEach(cactus => cactus.remove());
            cactiPositions.length = 0;
            startGame();
            gameOverDisplay.style.display = 'none';
            jumpButton.style.display = 'block';
            startTime = Date.now();
            spawnCactiGroup();
            cactusSpawnInterval = setInterval(spawnCactiGroup, 2000);
            gameLoop = setInterval(updateGame, 20);
        };
    </script>
    <div class="common bottom">
        Copyright &copy; 2024 
    </div>
</body>
</html>
