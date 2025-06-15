<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Flappy Bird Enhanced</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
    }
    canvas {
      display: block;
      background: skyblue;
    }
  </style>
</head>
<body>
<canvas id="game" width="360" height="640"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const GRAVITY = 0.5;
const FLAP = -8;
const PIPE_WIDTH = 52;
const PIPE_GAP = 140;
const BIRD_SIZE = 34;

let birdY = 250;
let birdVelocity = 0;
let pipes = [];
let score = 0;
let gameOver = false;

const bgImg = new Image();
const birdImg = new Image();
const pipeTopImg = new Image();
const pipeBottomImg = new Image();
bgImg.src = "https://i.imgur.com/1cXjJpH.png";
birdImg.src = "https://i.imgur.com/jM3H8dB.png";
pipeTopImg.src = "https://i.imgur.com/bFzUeJt.png";
pipeBottomImg.src = "https://i.imgur.com/NXxKSl7.png";

function reset() {
  birdY = 250;
  birdVelocity = 0;
  pipes = [];
  score = 0;
  gameOver = false;
}

function drawBackground() {
  ctx.drawImage(bgImg, 0, 0, canvas.width, canvas.height);
}

function drawBird() {
  ctx.drawImage(birdImg, 60, birdY, BIRD_SIZE, BIRD_SIZE);
}

function drawPipes() {
  pipes.forEach(pipe => {
    ctx.drawImage(pipeTopImg, pipe.x, pipe.top - pipeTopImg.height);
    ctx.drawImage(pipeBottomImg, pipe.x, pipe.top + PIPE_GAP);
  });
}

function update() {
  if (gameOver) return;

  birdVelocity += GRAVITY;
  birdY += birdVelocity;

  if (birdY > canvas.height || birdY < 0) {
    gameOver = true;
  }

  pipes.forEach(pipe => {
    pipe.x -= 2;

    if (
      60 + BIRD_SIZE > pipe.x &&
      60 < pipe.x + PIPE_WIDTH &&
      (birdY < pipe.top || birdY + BIRD_SIZE > pipe.top + PIPE_GAP)
    ) {
      gameOver = true;
    }

    if (!pipe.passed && pipe.x + PIPE_WIDTH < 60) {
      pipe.passed = true;
      score++;
    }
  });

  pipes = pipes.filter(pipe => pipe.x + PIPE_WIDTH > 0);

  if (pipes.length === 0 || pipes[pipes.length - 1].x < 200) {
    const top = Math.random() * (canvas.height - PIPE_GAP - 100) + 50;
    pipes.push({ x: canvas.width, top: top, passed: false });
  }
}

function draw() {
  drawBackground();
  drawPipes();
  drawBird();

  ctx.fillStyle = "white";
  ctx.font = "28px Arial";
  ctx.fillText("Score: " + score, 10, 40);

  if (gameOver) {
    ctx.fillText("Game Over! Click to restart", 40, canvas.height / 2);
  }
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

canvas.addEventListener("click", () => {
  if (gameOver) {
    reset();
  } else {
    birdVelocity = FLAP;
  }
});

window.onload = () => {
  loop();
};
</script>
</body>
</html>
