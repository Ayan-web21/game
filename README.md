
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mini Flappy Bird</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: skyblue;
      font-family: sans-serif;
    }
    canvas {
      display: block;
      margin: auto;
      background: #70c5ce;
    }
  </style>
</head>
<body>
<canvas id="game" width="320" height="480"></canvas>

<script>
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  const GRAVITY = 0.5;
  const FLAP = -8;
  const PIPE_WIDTH = 50;
  const PIPE_GAP = 120;
  const BIRD_SIZE = 20;

  let birdY = 150;
  let birdVelocity = 0;
  let pipes = [];
  let score = 0;
  let gameOver = false;

  function reset() {
    birdY = 150;
    birdVelocity = 0;
    pipes = [];
    score = 0;
    gameOver = false;
  }

  function drawBird() {
    ctx.fillStyle = 'yellow';
    ctx.fillRect(50, birdY, BIRD_SIZE, BIRD_SIZE);
  }

  function drawPipes() {
    ctx.fillStyle = 'green';
    pipes.forEach(pipe => {
      ctx.fillRect(pipe.x, 0, PIPE_WIDTH, pipe.top);
      ctx.fillRect(pipe.x, pipe.top + PIPE_GAP, PIPE_WIDTH, canvas.height);
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

      // Collision detection
      if (
        50 + BIRD_SIZE > pipe.x &&
        50 < pipe.x + PIPE_WIDTH &&
        (birdY < pipe.top || birdY + BIRD_SIZE > pipe.top + PIPE_GAP)
      ) {
        gameOver = true;
      }

      // Score update
      if (!pipe.passed && pipe.x + PIPE_WIDTH < 50) {
        pipe.passed = true;
        score++;
      }
    });

    // Remove off-screen pipes
    pipes = pipes.filter(p => p.x + PIPE_WIDTH > 0);

    // Add new pipes
    if (pipes.length === 0 || pipes[pipes.length - 1].x < 200) {
      const top = Math.random() * (canvas.height - PIPE_GAP - 60) + 20;
      pipes.push({ x: canvas.width, top: top, passed: false });
    }
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawBird();
    drawPipes();

    // Score
    ctx.fillStyle = 'white';
    ctx.font = '24px sans-serif';
    ctx.fillText(`Score: ${score}`, 10, 30);

    if (gameOver) {
      ctx.fillText("Game Over! Click to restart", 30, canvas.height / 2);
    }
  }

  function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
  }

  canvas.addEventListener('click', () => {
    if (gameOver) {
      reset();
    } else {
      birdVelocity = FLAP;
    }
  });

  loop();
</script>
</body>
</html>
