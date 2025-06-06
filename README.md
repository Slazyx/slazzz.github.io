<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Block Puzzle Fusion</title>
  <Link rel="shortcut icon" href ="512x512.png" type="image/x-icon">
  <style>
    @keyframes pop {
      0% { transform: scale(1); }
      50% { transform: scale(1.3); }
      100% { transform: scale(1); }
    }

    @keyframes flash {
      0% { background-color: #f1c40f; }
      100% { background-color: #fbfcfd; }
    }

    body {
      font-family: sans-serif;
      background: #003369;
      margin: 0;
      color: #3affd4;
    }
    .main-menu {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
    }
    .main-menu button {
      padding: 15px 30px;
      font-size: 24px;
      cursor: pointer;
      background-color: #3affd4;
      color: rgb(255, 255, 255);
      border: none;
      border-radius: 10px;
    }
    .main-menu button:hover {
      background-color: #3affd4;
    }
    .game-container {
      display: none;
      flex-direction: column;
      align-items: center;
      position: relative;
    }
    .score {
      font-size: 24px;
      margin-bottom: 7px;
      font-weight: bold;
    }
    .best-score {
      position: absolute;
      top: -2px;
      left: 733px;
      font-size: 25px;
      font-weight: bold;
      display: flex;
      align-items: center;
      color: gold;
    }
    .best-score span {
      margin-left: 5px;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(10, 30px);
      gap: 2px;
      margin-bottom: 20px;
    }
    .cell {
      width: 30px;
      height: 30px;
      background: #fbfcfd;
      border: 1px solid #3affd4;
      transition: background 0.2s;
    }
    .cell.filled {
      animation: pop 0.3s ease;
    }
    .cell.clearing {
      animation: flash 0.5s ease;
    }
    .blocks {
      display: flex;
      gap: 20px;
      margin-bottom: 20px;
    }
    .block {
      display: grid;
      grid-template-columns: repeat(4, 20px);
      grid-template-rows: repeat(4, 20px);
      gap: 2px;
      cursor: pointer;
    }
    .block-cell {
      width: 20px;
      height: 20px;
      background: #003369;
    }
    .block-cell.active {
      background: #3affd4;
    }
    .block.selected {
      outline: 3px solid yellow;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
      background-color: #3affd4;
      color: white;
      border: none;
      border-radius: 5px;
    }
    button:hover {
      background-color: #3affd4;
    }
    .game-over {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 30px;
      border-radius: 10px;
      text-align: center;
      font-size: 24px;
      display: none;
    }
    .settings-menu {
      position: absolute;
      top: 10;
      right: 10px;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }
    .settings-panel {
      display: none;
      position: absolute;
      top: 20px;
      left: 20px;
      background: #001f3f;
      color: white;
    }
  </style>
</head>
<body>
  <div class="main-menu" id="mainMenu">
    <img src="512x512.png" alt="" class="logo" />
    <h1>Block Puzzle Fusion</h1>
    <button onclick="startGame()">Jouer</button>
  </div>

  <div class="game-container" id="gameContainer">
    <div class="">
    
    </div>
    <div class="best-score">👑 <span id="bestScore">Best: 0</span></div>
    <div class="score" id="score">Score: 0</div>
    <div class="grid" id="grid"></div>
    <div class="blocks" id="blocks"></div>
    <button onclick="resetGame()">Réinitialiser</button>
  </div>

  <div class="game-over" id="gameOver">
    Partie terminée !<br/>
    <button onclick="resetGame()">Rejouer</button>
  </div>

  <div class="settings-panel" id="settingsPanel">
    <h3>Paramètres</h3>
    <button onclick="toggleSound()" id="soundButton">Couper le son</button
  </div>

  <audio id="placeSound" src="https://www.myinstants.com/media/sounds/click.mp3"></audio>
  <audio id="clearSound" src="https://www.myinstants.com/media/sounds/coin_1.mp3"></audio>
  <audio id="gameOverSound" src="https://www.myinstants.com/media/sounds/sad-trombone.mp3"></audio>

<script>
    const grid = document.getElementById("grid");
    const blocksContainer = document.getElementById("blocks");
    const placeSound = document.getElementById("placeSound");
    const clearSound = document.getElementById("clearSound");
    const gameOverSound = document.getElementById("gameOverSound");
    const scoreDisplay = document.getElementById("score");
    const bestScoreDisplay = document.getElementById("bestScore");
    const gameOverDisplay = document.getElementById("gameOver");
    const gameContainer = document.getElementById("gameContainer");
    const mainMenu = document.getElementById("mainMenu");
    const soundBtn = document.getElementById("soundBtn");
    const cells = [];
    let selectedBlock = null;
    let score = 0;
    let bestScore = localStorage.getItem("bestScore") || 0;
    let soundOn = true;
    bestScoreDisplay.textContent = `Best: ${bestScore}`;

    const blockShapes = [
      [ [1, 1], [1, 1] ],
      [ [1, 1, 1, 1] ],
      [ [1, 0], [1, 0], [1, 1] ],
      [ [0, 1], [0, 1], [1, 1] ],
      [ [1, 1, 0], [0, 1, 1] ]
    ];

    function startGame() {
      mainMenu.style.display = "none";
      gameContainer.style.display = "flex";
    }

    function goHome() {
      gameContainer.style.display = "none";
      mainMenu.style.display = "flex";
    }

    function toggleSound() {
      soundOn = !soundOn;
      soundBtn.textContent = soundOn ? "🔊 Son: Activé" : "🔇 Son: Désactivé";
    }

    function playSound(sound) {
      if (soundOn) {
        sound.currentTime = 0;
        sound.play();
      }
    }

    function randomColor() {
      const colors = ["#e74c3c", "#3498db", "#2ecc71", "#f1c40f", "#7f00ff"];
      return colors[Math.floor(Math.random() * colors.length)];
    }

    function createGrid() {
      for (let i = 0; i < 100; i++) {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        cell.addEventListener("click", () => placeBlock(i));
        grid.appendChild(cell);
        cells.push(cell);
      }
    }

    function generateBlocks() {
      blocksContainer.innerHTML = "";
      let hasValid = false;

      for (let i = 0; i < 3; i++) {
        const shape = blockShapes[Math.floor(Math.random() * blockShapes.length)];
        const color = randomColor();
        const block = document.createElement("div");
        block.classList.add("block");
        const flat = [];
        for (let y = 0; y < 4; y++) {
          for (let x = 0; x < 4; x++) {
            const cell = document.createElement("div");
            cell.classList.add("block-cell");
            if (shape[y]?.[x]) {
              cell.classList.add("active");
              cell.style.background = color;
              flat.push({ x, y, color });
            }
            block.appendChild(cell);
          }
        }
        block.dataset.shape = JSON.stringify(flat);
        block.addEventListener("click", () => {
          selectedBlock = flat;
          Array.from(blocksContainer.children).forEach(b => b.classList.remove("selected"));
          block.classList.add("selected");
        });
        blocksContainer.appendChild(block);

        if (!hasValid && isBlockPlaceable(flat)) {
          hasValid = true;
        }
      }

      if (!hasValid) {
        gameOver();
      }
    }

    function isBlockPlaceable(block) {
      for (let baseY = 0; baseY < 10; baseY++) {
        for (let baseX = 0; baseX < 10; baseX++) {
          if (canPlaceBlock(baseX, baseY, block)) {
            return true;
          }
        }
      }
      return false;
    }

    function placeBlock(index) {
      if (!selectedBlock) return;
      const baseX = index % 10;
      const baseY = Math.floor(index / 10);

      if (canPlaceBlock(baseX, baseY, selectedBlock)) {
        selectedBlock.forEach(({ x, y, color }) => {
          const idx = (baseY + y) * 10 + (baseX + x);
          cells[idx].classList.add("filled");
          cells[idx].style.background = color;
        });
        playSound(placeSound);
        score += 50;
        updateScore();
        selectedBlock = null;
        Array.from(blocksContainer.children).forEach(b => b.classList.remove("selected"));
        checkFullLines();
        generateBlocks();
      }
    }

    function canPlaceBlock(baseX, baseY, shape) {
      return shape.every(({ x, y }) => {
        const tx = baseX + x;
        const ty = baseY + y;
        if (tx < 0 || tx >= 10 || ty < 0 || ty >= 10) return false;
        const idx = ty * 10 + tx;
        return !cells[idx].classList.contains("filled");
      });
    }

    function animateAndClear(line) {
      line.forEach(cell => {
        cell.classList.add("clearing");
      });
      setTimeout(() => {
        line.forEach(cell => {
          cell.classList.remove("filled", "clearing");
          cell.style.background = "#fff";
        });
      }, 500);
    }

    function checkFullLines() {
      let cleared = 0;
      for (let row = 0; row < 10; row++) {
        const line = cells.slice(row * 10, row * 10 + 10);
        if (line.every(c => c.classList.contains("filled"))) {
          animateAndClear(line);
          playSound(clearSound);
          cleared++;
        }
      }

      for (let col = 0; col < 10; col++) {
        const column = [];
        for (let row = 0; row < 10; row++) {
          column.push(cells[row * 10 + col]);
        }
        if (column.every(c => c.classList.contains("filled"))) {
          animateAndClear(column);
          playSound(clearSound);
          cleared++;
        }
      }

      if (cleared > 0) {
        score += cleared * 850;
        updateScore();
      }
    }

    function updateScore() {
      scoreDisplay.textContent = `Score: ${score}`;
      if (score > bestScore) {
        bestScore = score;
        localStorage.setItem("bestScore", bestScore);
        bestScoreDisplay.textContent = `Best: ${bestScore}`;
      }
    }

    function resetGame() {
      cells.forEach(cell => {
        cell.classList.remove("filled", "clearing");
        cell.style.background = "#fff";
      });
      score = 0;
      updateScore();
      gameOverDisplay.style.display = "none";
      generateBlocks();
    }

    function gameOver() {
      gameOverDisplay.style.display = "block";
      playSound(gameOverSound);
    }

    createGrid();
    generateBlocks();
  </script>
</body>
</html>
