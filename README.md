<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>貪食蛇</title>
    <style>
        body {
            background-color: #f0f0f0;
            font-family: 'Microsoft JhengHei', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            flex-direction: column;
        }
        #game-container {
            position: relative;
            border: 2px solid #333;
        }
        #gameCanvas {
            background-color: #fff;
            display: block;
        }
        #forbidden-box {
            position: absolute;
            top: 0;
            right: 0;
            width: 60px; /* 3 grid units */
            height: 60px; /* 3 grid units */
            border: 2px solid black;
            background-color: #eee;
            box-sizing: border-box;
            font-size: 40px;
            display: flex;
            justify-content: center;
            align-items: center;
            color: black;
            font-weight: bold;
        }
        #controls {
            margin-top: 20px;
            text-align: center;
        }
        button {
            font-size: 18px;
            padding: 10px 20px;
            margin: 5px;
            cursor: pointer;
            border-radius: 5px;
            border: 1px solid #ccc;
            background-color: #fff;
        }
        #startBtn {
            background-color: #4CAF50;
            color: white;
            border-color: #4CAF50;
        }
        .direction-btn-group {
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas" width="400" height="400"></canvas>
        <div id="forbidden-box">柯</div>
    </div>
    <div id="controls">
        <button id="startBtn">開始遊戲</button>
        <div class="direction-btn-group">
            <button id="upBtn">上</button>
        </div>
        <div>
            <button id="leftBtn">左</button>
            <button id="downBtn">下</button>
            <button id="rightBtn">右</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startBtn = document.getElementById('startBtn');

        // 方向控制按鈕
        const upBtn = document.getElementById('upBtn');
        const downBtn = document.getElementById('downBtn');
        const leftBtn = document.getElementById('leftBtn');
        const rightBtn = document.getElementById('rightBtn');

        const gridSize = 20;
        const tileCount = canvas.width / gridSize;

        // 障礙物「柯」的位置和大小
        const forbiddenBox = {
            x: tileCount - 3,
            y: 0,
            width: 3,
            height: 3
        };

        let snake = [{ x: 10, y: 10 }];
        let food = {};
        let direction = { x: 0, y: 0 };
        let score = 0;
        let gameInterval;
        let isGameRunning = false;

        function generateFood() {
            food = {
                x: Math.floor(Math.random() * tileCount),
                y: Math.floor(Math.random() * tileCount)
            };

            // 確保食物不會生成在蛇身上或障礙物上
            let onSnake = snake.some(part => part.x === food.x && part.y === food.y);
            let onForbiddenBox = food.x >= forbiddenBox.x && food.x < forbiddenBox.x + forbiddenBox.width &&
                                 food.y >= forbiddenBox.y && food.y < forbiddenBox.y + forbiddenBox.height;

            if (onSnake || onForbiddenBox) {
                generateFood();
            }
        }

        function draw() {
            // 清除畫布
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 畫蛇
            snake.forEach((part, index) => {
                if (index === 0) { // 蛇頭
                    ctx.fillStyle = 'red';
                    ctx.font = `${gridSize}px Arial`;
                    ctx.fillText('蔥', part.x * gridSize, (part.y + 1) * gridSize - (gridSize/4));
                } else { // 蛇身 (這裡也用蔥)
                    ctx.fillStyle = 'darkred';
                    ctx.font = `${gridSize}px Arial`;
                    ctx.fillText('蔥', part.x * gridSize, (part.y + 1) * gridSize - (gridSize/4));
                }
            });

            // 畫食物
            ctx.fillStyle = 'green';
            ctx.font = `${gridSize}px Arial`;
            ctx.fillText('草', food.x * gridSize, (food.y + 1) * gridSize - (gridSize/4));
        }

        function update() {
            const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

            // 碰撞檢測 - 牆壁
            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                gameOver();
                return;
            }

            // 碰撞檢測 - 障礙物「柯」
            if (head.x >= forbiddenBox.x && head.x < forbiddenBox.x + forbiddenBox.width &&
                head.y >= forbiddenBox.y && head.y < forbiddenBox.y + forbiddenBox.height) {
                gameOver();
                return;
            }

            // 碰撞檢測 - 蛇身
            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    gameOver();
                    return;
                }
            }

            snake.unshift(head);

            // 吃到食物
            if (head.x === food.x && head.y === food.y) {
                score++;
                generateFood();
            } else {
                snake.pop();
            }

            draw();
        }

        function gameOver() {
            clearInterval(gameInterval);
            isGameRunning = false;
            alert(`遊戲結束！你的分數是：${score}`);
            startBtn.disabled = false;
        }

        function changeDirection(event) {
            const { keyCode } = event;
            const goingUp = direction.y === -1;
            const goingDown = direction.y === 1;
            const goingLeft = direction.x === -1;
            const goingRight = direction.x === 1;

            if (keyCode === 37 && !goingRight) { // 左
                direction = { x: -1, y: 0 };
            } else if (keyCode === 38 && !goingDown) { // 上
                direction = { x: 0, y: -1 };
            } else if (keyCode === 39 && !goingLeft) { // 右
                direction = { x: 1, y: 0 };
            } else if (keyCode === 40 && !goingUp) { // 下
                direction = { x: 0, y: 1 };
            }
        }
        
        // 按鈕控制
        upBtn.addEventListener('click', () => { if (direction.y !== 1) direction = { x: 0, y: -1 }; });
        downBtn.addEventListener('click', () => { if (direction.y !== -1) direction = { x: 0, y: 1 }; });
        leftBtn.addEventListener('click', () => { if (direction.x !== 1) direction = { x: -1, y: 0 }; });
        rightBtn.addEventListener('click', () => { if (direction.x !== -1) direction = { x: 1, y: 0 }; });

        function startGame() {
            if (isGameRunning) return;
            isGameRunning = true;
            startBtn.disabled = true;

            snake = [{ x: 10, y: 10 }];
            direction = { x: 0, y: 0 };
            score = 0;
            generateFood();
            
            gameInterval = setInterval(update, 150);
        }

        document.addEventListener('keydown', changeDirection);
        startBtn.addEventListener('click', startGame);

    </script>
</body>
</html>
