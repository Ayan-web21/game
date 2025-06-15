<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Flappy Bird Temple</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background: #000;
    }
    canvas {
      display: block;
      background: #70c5ce;
    }
  </style>
</head>
<body>
<canvas id="game"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

// Fullscreen
function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

// Constants
const GRAVITY = 0.4;
const FLAP = -8;
const GAP = 200;
const PIPE_WIDTH = 80;
const BIRD_WIDTH = 40;
const BIRD_HEIGHT = 30;

// Game state
let birdY = canvas.height / 2, birdVel = 0, score = 0, gameOver = false;
let pipes = [];

// Load images
const loadImage = (src) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.src = src;
    img.onload = () => resolve(img);
  });
};

// Use Imgur images for pillars that work
const assets = {
  bg: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/background-day.png',
  base: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/base.png',
  bird: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/yellowbird-midflap.png',
  pillar: 'https://i.imgur.com/2RXXdYt.png'  // Reliable temple column image
};

let images = {};

async function loadAssets() {
  for (let key in assets) {
    images[key] = await loadImage(assets[key]);
  }
  startGame();
}

function resetGame() {
  birdY = canvas.height / 2;
  birdVel = 0;
  pipes = [];
  score = 0;
  gameOver = false;
}

function update() {
  if (gameOver) return;

  birdVel += GRAVITY;
  birdY += birdVel;

  if (birdY + BIRD_HEIGHT > canvas.height || birdY < 0) {
    gameOver = true;
  }

  // Update pipes
  for (let pipe of pipes) {
    pipe.x -= 4;
    // Collision
    if (
      100 + BIRD_WIDTH > pipe.x &&
      100 < pipe.x + PIPE_WIDTH &&
      (birdY < pipe.y + images.pillar.height ||
       birdY + BIRD_HEIGHT > pipe.y + images.pillar.height + GAP)
    ) {
      gameOver = true;
    }

    if (!pipe.passed && pipe.x + PIPE_WIDTH < 100) {
      pipe.passed = true;
      score++;
    }
  }

  // Spawn pipes
  if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 300) {
    const y = -Math.floor(Math.random() * (images.pillar.height - 100));
    pipes.push({ x: canvas.width, y: y, passed: false });
  }

  pipes = pipes.filter(p => p.x + PIPE_WIDTH > 0);
}

function draw() {
  // Background
  ctx.drawImage(images.bg, 0, 0, canvas.width, canvas.height);

  // Pipes
  for (let p of pipes) {
    // Top pillar (flipped)
    ctx.save();
    ctx.translate(p.x + PIPE_WIDTH / 2, p.y + images.pillar.height);
    ctx.scale(1, -1);
    ctx.drawImage(images.pillar, -PIPE_WIDTH / 2, 0, PIPE_WIDTH, images.pillar.height);
    ctx.restore();

    // Bottom pillar
    ctx.drawImage(
      images.pillar,
      p.x,
      p.y + images.pillar.height + GAP,
      PIPE_WIDTH,
      images.pillar.height
    );
  }

  // Ground (base)
  ctx.drawImage(images.base, 0, canvas.height - images.base.height, canvas.width, images.base.height);

  // Bird
  ctx.drawImage(images.bird, 100, birdY, BIRD_WIDTH, BIRD_HEIGHT);

  // Score
  ctx.fillStyle = 'white';
  ctx.font = '30px Arial';
  ctx.fillText(`Score: ${score}`, 20, 40);

  if (gameOver) {
    ctx.fillText('Game Over! Click to restart', 50, canvas.height / 2);
  }
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

function startGame() {
  resetGame();
  loop();
}

canvas.addEventListener('click', () => {
  if (gameOver) resetGame();
  birdVel = FLAP;
});

loadAssets();
</script>
</body>
</html>
