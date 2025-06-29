<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Jogo de Damas</title>
  <style>
    body {
      background: #222;
      color: #eee;
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
    }
    h1 { margin: 10px; }
    #board {
      display: grid;
      grid-template-columns: repeat(8, 60px);
      grid-template-rows: repeat(8, 60px);
      border: 3px solid #fff;
    }
    .cell {
      width: 60px;
      height: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .dark { background-color: #444; }
    .light { background-color: #ddd; }
    .piece {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 22px;
      color: #fff;
      cursor: pointer;
    }
    .red { background-color: red; }
    .black { background-color: black; }
    .selected { outline: 3px solid yellow; }
    #status {
      margin-top: 10px;
      font-weight: bold;
    }
    button, select {
      margin-top: 12px;
      padding: 8px 12px;
      font-size: 16px;
    }
  </style>
</head>
<body>

  <h1>Jogo de Damas</h1>
  <div id="setup">
    <label>
      Escolha sua cor:
      <select id="playerColor">
        <option value="red">Vermelho</option>
        <option value="black">Preto</option>
      </select>
    </label>
    <button onclick="startGame()">Iniciar</button>
  </div>

  <div id="board" style="display:none;"></div>
  <div id="status"></div>
  <button onclick="startGame()" style="display:none;" id="restartBtn">Reiniciar</button>

  <script>
    const boardEl = document.getElementById("board");
    const statusEl = document.getElementById("status");
    const restartBtn = document.getElementById("restartBtn");
    let board = [];
    let selected = null;
    let playerColor = "red";
    let aiColor = "black";
    let turn = "red";

    const SIZE = 8;

    function startGame() {
      playerColor = document.getElementById("playerColor").value;
      aiColor = playerColor === "red" ? "black" : "red";
      turn = "red"; // vermelho sempre começa
      document.getElementById("setup").style.display = "none";
      boardEl.style.display = "grid";
      restartBtn.style.display = "block";
      initBoard();
      renderBoard();
      if (turn !== playerColor) setTimeout(aiMove, 500);
    }

    function initBoard() {
      board = Array.from({ length: SIZE }, () => Array(SIZE).fill(null));
      // Vermelho sempre embaixo (linhas 5 a 7)
      for (let r = 5; r < 8; r++) {
        for (let c = 0; c < SIZE; c++) {
          if ((r + c) % 2 === 1) board[r][c] = { color: "red", queen: false };
        }
      }
      // Preto sempre em cima (linhas 0 a 2)
      for (let r = 0; r < 3; r++) {
        for (let c = 0; c < SIZE; c++) {
          if ((r + c) % 2 === 1) board[r][c] = { color: "black", queen: false };
        }
      }
    }

    function renderBoard() {
      boardEl.innerHTML = "";
      for (let r = 0; r < SIZE; r++) {
        for (let c = 0; c < SIZE; c++) {
          const cell = document.createElement("div");
          cell.className = "cell " + ((r + c) % 2 === 0 ? "light" : "dark");
          cell.dataset.row = r;
          cell.dataset.col = c;

          const piece = board[r][c];
          if (piece) {
            const el = document.createElement("div");
            el.className = "piece " + piece.color;
            el.innerHTML = piece.queen ? "👑" : "";
            if (selected && selected.row === r && selected.col === c)
              el.classList.add("selected");
            cell.appendChild(el);
          }

          if ((r + c) % 2 === 1) {
            cell.addEventListener("click", () => onCellClick(r, c));
          }

          boardEl.appendChild(cell);
        }
      }
      updateStatus();
    }

    function updateStatus() {
      if (getAllPlayerMoves(playerColor).length === 0) {
        statusEl.textContent = "Você perdeu!";
      } else if (getAllPlayerMoves(aiColor).length === 0) {
        statusEl.textContent = "Você venceu!";
      } else {
        statusEl.textContent = `Vez de ${turn === playerColor ? "você" : "IA"} (${turn})`;
      }
    }

    function onCellClick(r, c) {
      if (turn !== playerColor) return;

      const piece = board[r][c];
      if (piece && piece.color === playerColor) {
        selected = { row: r, col: c };
        renderBoard();
        return;
      }

      if (selected) {
        const moves = getValidMoves(selected.row, selected.col);
        const move = moves.find(m => m.row === r && m.col === c);
        if (move) {
          makeMove(selected.row, selected.col, move.row, move.col, move.capture);
          selected = null;
          renderBoard();
          setTimeout(aiMove, 500);
        }
      }
    }

    function makeMove(fromR, fromC, toR, toC, capture) {
      const piece = board[fromR][fromC];
      board[toR][toC] = piece;
      board[fromR][fromC] = null;
      if (capture) board[capture.row][capture.col] = null;

      // promoção pra dama
      if (piece.color === "red" && toR === 0) piece.queen = true;
      if (piece.color === "black" && toR === 7) piece.queen = true;

      turn = turn === "red" ? "black" : "red";
    }

    function getValidMoves(r, c) {
      const piece = board[r][c];
      const dirs = piece.queen
        ? [[1,1],[1,-1],[-1,1],[-1,-1]]
        : (piece.color === "red" ? [[-1,-1],[-1,1]] : [[1,-1],[1,1]]);

      const moves = [];

      for (const [dr, dc] of dirs) {
        const nr = r + dr;
        const nc = c + dc;
        if (inBounds(nr, nc) && !board[nr][nc]) {
          moves.push({ row: nr, col: nc });
        }
        // captura
        const jr = r + dr;
        const jc = c + dc;
        const er = r + dr * 2;
        const ec = c + dc * 2;
        if (
          inBounds(er, ec) &&
          board[jr][jc] &&
          board[jr][jc].color !== piece.color &&
          !board[er][ec]
        ) {
          moves.push({ row: er, col: ec, capture: { row: jr, col: jc } });
        }
      }

      return moves;
    }

    function getAllPlayerMoves(color) {
      const moves = [];
      for (let r = 0; r < SIZE; r++) {
        for (let c = 0; c < SIZE; c++) {
          const piece = board[r][c];
          if (piece && piece.color === color) {
            const valid = getValidMoves(r, c);
            if (valid.length) moves.push({ from: { r, c }, moves: valid });
          }
        }
      }
      return moves;
    }

    function aiMove() {
      if (turn !== aiColor) return;

      const all = getAllPlayerMoves(aiColor);
      if (all.length === 0) {
        updateStatus();
        return;
      }

      // capturas primeiro
      for (const obj of all) {
        const cap = obj.moves.find(m => m.capture);
        if (cap) {
          makeMove(obj.from.r, obj.from.c, cap.row, cap.col, cap.capture);
          renderBoard();
          return;
        }
      }

      // senão move aleatório
      const randPiece = all[Math.floor(Math.random() * all.length)];
      const randMove = randPiece.moves[Math.floor(Math.random() * randPiece.moves.length)];
      makeMove(randPiece.from.r, randPiece.from.c, randMove.row, randMove.col, randMove.capture);
      renderBoard();
    }

    function inBounds(r, c) {
      return r >= 0 && r < SIZE && c >= 0 && c < SIZE;
    }
  </script>

</body>
</html>
