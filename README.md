# Snek-game
O.G snake game
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Snake Game</title>
  <link rel="manifest" href="manifest.json" />
  <style>
    body {
      margin: 0;
      background: #000;
      color: white;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      overflow: hidden;
      font-family: sans-serif;
    }

    #score, #highscore {
      font-size: 24px;
    }

    canvas {
      background: #111;
      touch-action: none;
      margin-top: 10px;
    }

    #play-again {
      display: none;
      margin-top: 10px;
      padding: 10px 20px;
      font-size: 18px;
      background: hotpink;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div id="score">Score: 0</div>
  <div id="highscore">High Score: 0</div>
  <canvas id="game"></canvas>
  <button id="play-again">Play Again</button>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const highScoreEl = document.getElementById('highscore');
    const playAgainBtn = document.getElementById('play-again');

    const screenSize = Math.min(window.innerWidth, window.innerHeight) * 0.9;
    canvas.width = canvas.height = screenSize;

    const scale = 20;
    const rows = Math.floor(canvas.height / scale);
    const columns = Math.floor(canvas.width / scale);

    let snake;
    let foods = [];
    const foodCount = 3;
    let score = 0;
    let gameOver = false;
    let highScore = localStorage.getItem('snakeHighScore') || 0;

    class Snake {
      constructor() {
        this.body = [{ x: 5, y: 5 }];
        this.xDir = 1;
        this.yDir = 0;
      }

      draw() {
        for (let i = 0; i < this.body.length; i++) {
          ctx.fillStyle = i === 0 ? 'hotpink' : '#0f0';
          const part = this.body[i];
          ctx.fillRect(part.x * scale, part.y * scale, scale, scale);
        }
      }

      update() {
        const head = { ...this.body[0] };
        head.x += this.xDir;
        head.y += this.yDir;

        if (
          head.x < 0 || head.x >= columns ||
          head.y < 0 || head.y >= rows ||
          this.body.some(p => p.x === head.x && p.y === head.y)
        ) {
          if (score > highScore) {
            highScore = score;
            localStorage.setItem('snakeHighScore', highScore);
          }
          gameOver = true;
          playAgainBtn.style.display = 'block';
          return;
        }

        this.body.unshift(head);

        let ate = false;
        for (let i = foods.length - 1; i >= 0; i--) {
          if (head.x === foods[i].x && head.y === foods[i].y) {
            foods.splice(i, 1);
            foods.push(generateFood());
            score++;
            updateScore();
            ate = true;
          }
        }

        if (!ate) {
          this.body.pop();
        }
      }

      changeDirection(x, y) {
        if (x === -this.xDir && y === -this.yDir) return;
        this.xDir = x;
        this.yDir = y;
      }
    }

    function generateFood() {
      let food;
      do {
        food = {
          x: Math.floor(Math.random() * columns),
          y: Math.floor(Math.random() * rows)
        };
      } while (snake.body.some(p => p.x === food.x && p.y === food.y));
      return food;
    }

    function drawFoods() {
      ctx.fillStyle = 'red';
      for (let f of foods) {
        ctx.fillRect(f.x * scale, f.y * scale, scale, scale);
      }
    }

    function resetFoods() {
      foods = [];
      for (let i = 0; i < foodCount; i++) {
        foods.push(generateFood());
      }
    }

    function updateScore() {
      scoreEl.innerText = `Score: ${score}`;
      highScoreEl.innerText = `High Score: ${highScore}`;
    }

    function resetGame() {
      snake = new Snake();
      score = 0;
      gameOver = false;
      resetFoods();
      updateScore();
      playAgainBtn.style.display = 'none';
    }

    function gameLoop() {
      if (gameOver) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawFoods();
      snake.update();
      snake.draw();
    }

    // Touch swipe support
    let touchStartX = 0;
    let touchStartY = 0;

    canvas.addEventListener('touchstart', (e) => {
      const touch = e.touches[0];
      touchStartX = touch.clientX;
      touchStartY = touch.clientY;
    });

    canvas.addEventListener('touchend', (e) => {
      const touch = e.changedTouches[0];
      const dx = touch.clientX - touchStartX;
      const dy = touch.clientY - touchStartY;

      if (Math.abs(dx) > Math.abs(dy)) {
        dx > 0 ? snake.changeDirection(1, 0) : snake.changeDirection(-1, 0);
      } else {
        dy > 0 ? snake.changeDirection(0, 1) : snake.changeDirection(0, -1);
      }
    });

    playAgainBtn.addEventListener('click', () => {
      resetGame();
    });

    // PWA: register service worker
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js')
        .then(() => console.log('Service Worker Registered'));
    }

    // Start game
    resetGame();
    setInterval(gameLoop, 150);
  </script>
</body>
</html>
{
  "name": "Snake Game",
  "short_name": "Snake",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#000000",
  "theme_color": "#111111",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
const CACHE_NAME = 'snake-game-v1';
const urlsToCache = [
  'index.html',
  'manifest.json',
  'icon-192.png',
  'icon-512.png'
];

self.addEventListener('install', (e) => {
  e.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', (e) => {
  e.respondWith(
    caches.match(e.request).then((res) => res || fetch(e.request))
  );
});
