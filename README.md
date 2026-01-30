[index.html](https://github.com/user-attachments/files/24973900/index.html)
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>X-O Pro | نورت يا صعيدي</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root {
            --primary: #6c5ce7;
            --secondary: #00ffcc;
            --bg: #0f0c29;
            --text: #ffffff;
        }

        body {
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', sans-serif;
            background: linear-gradient(to bottom, #0f0c29, #302b63, #24243e);
            color: var(--text);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .app-container {
            width: 90%;
            max-width: 400px;
            text-align: center;
            background: rgba(255, 255, 255, 0.05);
            padding: 25px;
            border-radius: 30px;
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }

        .welcome-msg {
            font-size: 28px;
            font-weight: bold;
            color: var(--secondary);
            margin-bottom: 5px;
            text-shadow: 0 0 10px rgba(0, 255, 204, 0.5);
        }

        h1 { font-size: 18px; margin-bottom: 20px; opacity: 0.8; }

        .scoreboard {
            display: flex;
            justify-content: space-around;
            margin-bottom: 20px;
            background: rgba(0,0,0,0.3);
            padding: 15px;
            border-radius: 15px;
        }

        .score-box { font-size: 14px; }
        #status { margin-bottom: 15px; font-size: 20px; font-weight: bold; color: var(--secondary); min-height: 30px; }

        .board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            margin-bottom: 20px;
        }

        .cell {
            aspect-ratio: 1;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 45px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.3s;
            border: 1px solid rgba(255,255,255,0.05);
        }

        .cell:active { transform: scale(0.9); }
        .cell.win { background: var(--secondary); color: #000; }

        button {
            width: 100%;
            padding: 15px;
            border: none;
            border-radius: 15px;
            background: var(--primary);
            color: white;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 5px 15px rgba(108, 92, 231, 0.4);
        }
    </style>
</head>
<body>

<div class="app-container">
    <div class="welcome-msg">نورت يا صعيدي 👋</div>
    <h1>لعبة التحدي والذكاء</h1>
    
    <div class="scoreboard">
        <div>👤 أنت: <span id="scoreX">0</span></div>
        <div>🤖 الذكاء: <span id="scoreO">0</span></div>
    </div>

    <div id="status">دورك يا بطل (X)</div>

    <div class="board" id="board"></div>

    <button onclick="resetGame()">إعادة الجولة</button>
</div>

<script>
    const HUMAN = 'X', AI = 'O';
    let board = Array(9).fill('');
    let isGameOver = false;
    let scores = { X: 0, O: 0 };

    const clickSound = new Audio('https://actions.google.com/sounds/v1/foley/button_click.ogg');

    function createBoard() {
        const boardDiv = document.getElementById('board');
        boardDiv.innerHTML = '';
        board.forEach((cell, i) => {
            const div = document.createElement('div');
            div.className = 'cell';
            div.textContent = cell;
            div.onclick = () => play(i);
            boardDiv.appendChild(div);
        });
    }

    function play(i) {
        if (board[i] || isGameOver) return;
        clickSound.play();
        board[i] = HUMAN;
        updateUI();

        if (checkWin(board, HUMAN)) return end('X');
        if (!board.includes('')) return end('draw');

        document.getElementById('status').textContent = "الكمبيوتر يخطط...";
        setTimeout(aiPlay, 600);
    }

    function aiPlay() {
        if (isGameOver) return;
        let bestMove = minimax(board, AI).index;
        board[bestMove] = AI;
        updateUI();
        if (checkWin(board, AI)) return end('O');
        if (!board.includes('')) return end('draw');
        document.getElementById('status').textContent = "دورك يا بطل (X)";
    }

    function minimax(newBoard, player) {
        let avail = newBoard.map((v, i) => v === '' ? i : null).filter(v => v !== null);
        if (checkWin(newBoard, HUMAN)) return {score: -10};
        if (checkWin(newBoard, AI)) return {score: 10};
        if (avail.length === 0) return {score: 0};

        let moves = [];
        for (let i of avail) {
            let move = {index: i};
            newBoard[i] = player;
            move.score = minimax(newBoard, player === AI ? HUMAN : AI).score;
            newBoard[i] = '';
            moves.push(move);
        }

        let best;
        if (player === AI) {
            let max = -Infinity;
            moves.forEach((m, i) => { if (m.score > max) { max = m.score; best = i; } });
        } else {
            let min = Infinity;
            moves.forEach((m, i) => { if (m.score < min) { min = m.score; best = i; } });
        }
        return moves[best];
    }

    function checkWin(b, p) {
        const wins = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
        return wins.find(c => c.every(i => b[i] === p));
    }

    function end(winner) {
        isGameOver = true;
        const status = document.getElementById('status');
        if (winner === 'draw') status.textContent = "تعادل يا صعيدي! 🤝";
        else {
            status.textContent = winner === 'X' ? "عاش يا وحش! فزت 🎉" : "حظ أوفر المرة الجاية 🤖";
            if (winner === 'X') confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 } });
            scores[winner]++;
            document.getElementById(`score${winner}`).textContent = scores[winner];
            highlight(checkWin(board, winner));
        }
    }

    function highlight(c) { if(c) c.forEach(i => document.querySelectorAll('.cell')[i].classList.add('win')); }
    function updateUI() { document.querySelectorAll('.cell').forEach((c, i) => c.textContent = board[i]); }
    function resetGame() { board = Array(9).fill(''); isGameOver = false; document.getElementById('status').textContent = "دورك يا بطل (X)"; createBoard(); }

    createBoard();
</script>
</body>
</html>
