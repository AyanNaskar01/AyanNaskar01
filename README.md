<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ayan's Snake Game</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; font-family: "Courier New", monospace; }
  body {
    background: #0a0e1a;
    color: #f2f6fc;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 16px;
    padding: 20px;
  }
  h1 { color: #8b5cf6; font-size: 1.4rem; letter-spacing: 1px; }
  #score { font-size: 1.1rem; color: #c4b5fd; }
  canvas {
    background: #131b2e;
    border: 2px solid #8b5cf6;
    border-radius: 8px;
    box-shadow: 0 0 30px rgba(139, 92, 246, 0.25);
  }
  #hint { color: #93a0b8; font-size: 0.85rem; text-align: center; }
  #overlay {
    position: absolute;
    display: none;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    background: rgba(10, 14, 26, 0.9);
    padding: 24px 32px;
    border-radius: 12px;
    border: 1px solid #8b5cf6;
  }
  button {
    background: #8b5cf6;
    color: #0a0e1a;
    border: none;
    padding: 10px 20px;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    font-family: inherit;
  }
  a { color: #8b5cf6; }
</style>
</head>
<body>

  <h1>AYAN'S SNAKE</h1>
  <p id="score">Score: 0</p>
  <div style="position: relative;">
    <canvas id="game" width="360" height="360"></canvas>
    <div id="overlay">
      <p id="overlay-text">Game over</p>
      <button id="restart">Play again</button>
    </div>
  </div>
  <p id="hint">Arrow keys or swipe to move. Eat the dots, don't hit yourself.</p>
  <p><a href="https://github.com/AyanNaskar08">back to profile</a></p>

<script>
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const scoreEl = document.getElementById('score');
  const overlay = document.getElementById('overlay');
  const overlayText = document.getElementById('overlay-text');
  const restartBtn = document.getElementById('restart');

  const gridSize = 18;
  const tileCount = canvas.width / gridSize;

  let snake, direction, nextDirection, food, score, alive, loopId;

  function resetGame() {
    snake = [{ x: 9, y: 9 }];
    direction = { x: 0, y: 0 };
    nextDirection = { x: 0, y: 0 };
    score = 0;
    alive = true;
    placeFood();
    scoreEl.textContent = 'Score: 0';
    overlay.style.display = 'none';
    if (loopId) clearInterval(loopId);
    loopId = setInterval(tick, 110);
  }

  function placeFood() {
    let valid = false;
    while (!valid) {
      food = {
        x: Math.floor(Math.random() * tileCount),
        y: Math.floor(Math.random() * tileCount)
      };
      valid = !snake.some(seg => seg.x === food.x && seg.y === food.y);
    }
  }

  function tick() {
    if (!alive) return;
    direction = nextDirection;
    if (direction.x === 0 && direction.y === 0) {
      draw();
      return;
    }

    const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

    if (
      head.x < 0 || head.x >= tileCount ||
      head.y < 0 || head.y >= tileCount ||
      snake.some(seg => seg.x === head.x && seg.y === head.y)
    ) {
      endGame();
      return;
    }

    snake.unshift(head);

    if (head.x === food.x && head.y === food.y) {
      score += 10;
      scoreEl.textContent = 'Score: ' + score;
      placeFood();
    } else {
      snake.pop();
    }

    draw();
  }

  function draw() {
    ctx.fillStyle = '#131b2e';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = '#ffb86b';
    ctx.beginPath();
    ctx.arc(
      food.x * gridSize + gridSize / 2,
      food.y * gridSize + gridSize / 2,
      gridSize / 2.5, 0, Math.PI * 2
    );
    ctx.fill();

    snake.forEach((seg, i) => {
      ctx.fillStyle = i === 0 ? '#8b5cf6' : '#6d5bd0';
      ctx.fillRect(seg.x * gridSize + 1, seg.y * gridSize + 1, gridSize - 2, gridSize - 2);
    });
  }

  function endGame() {
    alive = false;
    clearInterval(loopId);
    overlayText.textContent = 'Game over — score ' + score;
    overlay.style.display = 'flex';
  }

  document.addEventListener('keydown', (e) => {
    const key = e.key;
    if (key === 'ArrowUp' && direction.y !== 1) nextDirection = { x: 0, y: -1 };
    else if (key === 'ArrowDown' && direction.y !== -1) nextDirection = { x: 0, y: 1 };
    else if (key === 'ArrowLeft' && direction.x !== 1) nextDirection = { x: -1, y: 0 };
    else if (key === 'ArrowRight' && direction.x !== -1) nextDirection = { x: 1, y: 0 };
  });

  let touchStartX = 0, touchStartY = 0;
  canvas.addEventListener('touchstart', (e) => {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
  });
  canvas.addEventListener('touchend', (e) => {
    const dx = e.changedTouches[0].clientX - touchStartX;
    const dy = e.changedTouches[0].clientY - touchStartY;
    if (Math.abs(dx) > Math.abs(dy)) {
      if (dx > 0 && direction.x !== -1) nextDirection = { x: 1, y: 0 };
      else if (dx < 0 && direction.x !== 1) nextDirection = { x: -1, y: 0 };
    } else {
      if (dy > 0 && direction.y !== -1) nextDirection = { x: 0, y: 1 };
      else if (dy < 0 && direction.y !== 1) nextDirection = { x: 0, y: -1 };
    }
  });

  restartBtn.addEventListener('click', resetGame);

  resetGame();
</script>
</body>
</html>
