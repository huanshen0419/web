<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Motion Challenge Game</title>
    <style>
        * {
            box-sizing: border-box;
        }
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #d2e0fb, #fef6ff);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        h1 {
            font-size: 2rem;
            margin-bottom: 20px;
            color: #333;
        }
        .grid {
            display: grid;
            grid-template-columns: repeat(5, 80px);
            grid-template-rows: repeat(5, 80px);
            gap: 6px;
            margin-bottom: 20px;
            background-color: #ccc;
            padding: 6px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }
        .cell {
            width: 80px;
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            font-weight: bold;
            border-radius: 10px;
            transition: transform 0.2s ease;
            background-color: white;
        }
        .cell:hover {
            transform: scale(1.05);
        }
        .cell > div {
            width: 60%;
            height: 60%;
        }
        .ball {
            background-color: #ff4d4d;
            border-radius: 50%;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        .hole {
            background-color: #333;
            border-radius: 50%;
        }
        .block {
            background-color: #888;
            border-radius: 8px;
            width: 80%;
            height: 80%;
        }
        .info {
            font-size: 18px;
            color: #555;
            background-color: #fff;
            padding: 10px 16px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        }
        table {
            margin-top: 20px;
            border-collapse: collapse;
            width: 80%;
        }
        table, th, td {
            border: 1px solid #ccc;
        }
        th, td {
            padding: 10px;
            text-align: center;
        }
        th {
            background-color: #f4f4f4;
        }
    </style>
</head>
<body tabindex="0">
    <h1>🎯 Motion Challenge Game</h1>

    <!-- 表單區域 -->
    <form id="playerForm">
        <label for="playerName">Player Name:</label>
        <input type="text" id="playerName" name="playerName" required>
        <button type="submit">Start Game</button>
    </form>

    <!-- 遊戲網格 -->
    <div class="grid" id="gameBoard"></div>

    <!-- 步數顯示 -->
    <div class="info">Moves: <span id="moveCount">0</span></div>

    <!-- 排行榜 -->
    <table id="scoreTable">
        <thead>
            <tr>
                <th>Level</th>
                <th>Best Moves</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1</td>
                <td id="level1Best">-</td>
            </tr>
            <tr>
                <td>2</td>
                <td id="level2Best">-</td>
            </tr>
            <tr>
                <td>3</td>
                <td id="level3Best">-</td>
            </tr>
            <tr>
                <td>4</td>
                <td id="level4Best">-</td>
            </tr>
            <tr>
                <td>5</td>
                <td id="level5Best">-</td>
            </tr>
        </tbody>
    </table>

    <script>
        let moveCount = 0;
        let currentLevel = 0;
        const bestMoves = [null, null, null, null, null]; // 儲存每關最佳步數

        const levels = [
            [
                ['', '', 'B', '', ''],
                ['', '', '', '', ''],
                ['', 'B', 'O', '', 'X'],
                ['', '', '', '', ''],
                ['B', '', '', '', '']
            ],
            [
                ['O', 'B', '', '', ''],
                ['', 'B', '', '', 'B'],
                ['', '', '', 'B', ''],
                ['B', '', '', '', ''],
                ['', '', 'X', '', '']
            ],
            [
                ['B', 'O', 'B', '', 'B'],
                ['B', '', '', '', 'B'],
                ['', 'B', '', 'B', ''],
                ['', '', '', '', 'X'],
                ['B', '', 'B', '', 'B']
            ],
            [
                ['B', 'O', 'B', '', 'B'],
                ['B', '', '', 'B', 'B'],
                ['', 'B', '', '', ''],
                ['B', '', '', 'B', 'X'],
                ['B', '', 'B', '', 'B']
            ],
            [
                ['O', 'B', 'B', '', 'B'],
                ['', '', '', '', 'B'],
                ['B', '', 'B', '', 'B'],
                ['', '', 'B', '', 'B'],
                ['', '', '', '', 'X']
            ]
        ];

        function loadLevel(levelIndex) {
            moveCount = 0;
            document.getElementById("moveCount").textContent = moveCount;
            grid.splice(0, grid.length, ...levels[levelIndex].map(row => row.slice()));
            renderGrid();
        }

        const grid = [];

        function renderGrid() {
            const board = document.getElementById("gameBoard");
            board.innerHTML = "";
            grid.forEach((row, y) => {
                row.forEach((cell, x) => {
                    const div = document.createElement("div");
                    div.classList.add("cell");
                    const inner = document.createElement("div");
                    if (cell === 'O') {
                        inner.classList.add("ball");
                    } else if (cell === 'X') {
                        inner.classList.add("hole");
                    } else if (cell === 'B') {
                        inner.classList.add("block");
                    }
                    div.appendChild(inner);
                    board.appendChild(div);
                });
            });
        }

        function findBallPosition() {
            for (let y = 0; y < grid.length; y++) {
                for (let x = 0; x < grid[y].length; x++) {
                    if (grid[y][x] === 'O') {
                        return { x, y };
                    }
                }
            }
            return null;
        }

        function moveBall(dx, dy) {
            const pos = findBallPosition();
            if (!pos) return;
            const newX = pos.x + dx;
            const newY = pos.y + dy;

            // 檢查是否超出邊界或撞到障礙物
            if (
                newX < 0 || newX >= grid[0].length ||
                newY < 0 || newY >= grid.length ||
                grid[newY][newX] === 'B'
            ) {
                return;
            }

            // 更新球的位置
            if (grid[newY][newX] === 'X') {
                grid[pos.y][pos.x] = '';
                grid[newY][newX] = 'O';
                moveCount++;
                document.getElementById("moveCount").textContent = moveCount;
                renderGrid();
                setTimeout(() => {
                    alert(`🎉 You completed Level ${currentLevel + 1}! Steps taken: ${moveCount}`);
                    if (!bestMoves[currentLevel] || moveCount < bestMoves[currentLevel]) {
                        bestMoves[currentLevel] = moveCount;
                        document.getElementById(`level${currentLevel + 1}Best`).textContent = moveCount;
                    }
                    currentLevel++;
                    if (currentLevel < levels.length) {
                        loadLevel(currentLevel);
                    } else {
                        alert("🎉 Congratulations! You completed all levels!");
                        location.reload(); // 重整遊戲
                    }
                }, 100);
                return;
            }

            grid[pos.y][pos.x] = '';
            grid[newY][newX] = 'O';
            moveCount++;
            document.getElementById("moveCount").textContent = moveCount;
            renderGrid();
        }

        document.addEventListener("keydown", (event) => {
            if (event.key === "ArrowUp") moveBall(0, -1);
            if (event.key === "ArrowDown") moveBall(0, 1);
            if (event.key === "ArrowLeft") moveBall(-1, 0);
            if (event.key === "ArrowRight") moveBall(1, 0);
        });

        document.getElementById("playerForm").addEventListener("submit", (event) => {
            event.preventDefault();
            const playerName = document.getElementById("playerName").value;
            alert(`Welcome, ${playerName}! Let's start the game!`);
            document.getElementById("playerForm").style.display = "none";
            loadLevel(currentLevel);
        });

        document.addEventListener("DOMContentLoaded", () => {
            document.body.focus();
        });
    </script>
</body>
</html>
