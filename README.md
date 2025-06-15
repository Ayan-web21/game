<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Flappy Bird Temple</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; margin: auto; background: #87CEEB; }
  </style>
</head>
<body>
<canvas id="game" width="288" height="512"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const GRAVITY = 0.25;
const FLAP = -4.6;
const GAP = 120;
const BIRD_WIDTH = 34, BIRD_HEIGHT = 24;
const PIPE_WIDTH = 52;

let birdY = 150, birdVelocity = 0, pipes = [], score = 0, gameOver = false;

const images = {
  bg: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/background-day.png',
  base: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/base.png',
  bird: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/yellowbird-midflap.png',
  pillarTop: 'https://i.imgur.com/WnDSKoq.png',   // Temple-style column (top)
  pillarBottom: 'https://i.imgur.com/WnDSKoq.png' // Same image, flipped
};

const sprites = {};
let loaded = 0;
for (let key in images) {
  const img = new Image();
  img.src = images[key];
  img.onload = () => {
    if (++loaded === Object.keys(images).length) gameLoop();
  };
  sprites[key] = img;
}

function resetGame() {
  birdY = 150;
  birdVelocity = 0;
  pipes = [];
  score = 0;
  gameOver = false;
}

function draw() {
  ctx.drawImage(sprites.bg, 0, 0);

  for (let p of pipes) {
    // Draw top pillar (flipped)
    ctx.save();
    ctx.translate(p.x + PIPE_WIDTH / 2, p.y + 320);
    ctx.scale(1, -1);
    ctx.drawImage(sprites.pillarTop, -PIPE_WIDTH / 2, 0, PIPE_WIDTH, 320);
    ctx.restore();

    // Draw bottom pillar
    ctx.drawImage(sprites.pillarBottom, p.x, p.y + 320 + GAP, PIPE_WIDTH, 320);
  }

  ctx.drawImage(sprites.base, 0, canvas.height - sprites.base.height);
  ctx.drawImage(sprites.bird, 50, birdY, BIRD_WIDTH, BIRD_HEIGHT);

  ctx.fillStyle = "white";
  ctx.font = "20px Arial";
  ctx.fillText("Score: " + score, 10, canvas.height - 20);

  if (gameOver) {
    ctx.fillText("Game Over! Click to restart", 40, canvas.height / 2);
  }
}

function update() {
  if (gameOver) return;

  birdVelocity += GRAVITY;
  birdY += birdVelocity;

  if (birdY + BIRD_HEIGHT > canvas.height - sprites.base.height || birdY < 0) {
    gameOver = true;
  }

  for (let p of pipes) {
    p.x -= 2;

    if (
      50 + BIRD_WIDTH > p.x &&
      50 < p.x + PIPE_WIDTH &&
      (birdY < p.y + 320 || birdY + BIRD_HEIGHT > p.y + 320 + GAP)
    ) {
      gameOver = true;
    }

    if (!p.passed && p.x + PIPE_WIDTH < 50) {
      p.passed = true;
      score++;
    }
  }

  if (pipes.length === 0 || pipes[pipes.length - 1].x < 160) {
    const y = -Math.floor(Math.random() * 150) - 50;
    pipes.push({ x: canvas.width, y: y, passed: false });
  }

  pipes = pipes.filter(p => p.x + PIPE_WIDTH > 0);
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

canvas.addEventListener("click", () => {
  if (gameOver) resetGame();
  else birdVelocity = FLAP;
});
</script>
</body>
</html>
