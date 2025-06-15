<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Flappy Bird Temple - Working Spacebar + Pillars</title>
  <style>
    html, body {
      margin: 0; padding: 0; overflow: hidden; background: #70c5ce;
      height: 100%;
      user-select: none;
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
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  // Resize canvas to full window
  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  // Constants
  const GRAVITY = 0.4;
  const FLAP = -8;
  const PIPE_GAP = 180;
  const PIPE_WIDTH = 80;
  const BIRD_WIDTH = 40;
  const BIRD_HEIGHT = 30;
  const BIRD_X = 100;

  // Game variables
  let birdY = canvas.height / 2;
  let birdVel = 0;
  let pipes = [];
  let score = 0;
  let gameOver = false;

  // Load images (reliable GH assets)
  const images = {};
  const assets = {
    bg: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/background-day.png',
    base: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/base.png',
    bird: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/yellowbird-midflap.png',
    pipe: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/pipe-green.png',
  };

  function loadImage(src) {
    return new Promise((resolve) => {
      const img = new Image();
      img.src = src;
      img.onload = () => resolve(img);
    });
  }

  async function loadAllImages() {
    for (const key in assets) {
      images[key] = await loadImage(assets[key]);
    }
  }

  // Reset game state
  function reset() {
    birdY = canvas.height / 2;
    birdVel = 0;
    pipes = [];
    score = 0;
    gameOver = false;
  }

  // Create new pipe with random height
  function createPipe() {
    // y is the top pipe's bottom y coordinate (negative because we draw flipped)
    const maxPipeHeight = images.pipe.height;
    const y = -Math.floor(Math.random() * (maxPipeHeight - 100));
    pipes.push({ x: canvas.width, y: y, passed: false });
  }

  // Update game state
  function update() {
    if (gameOver) return;

    birdVel += GRAVITY;
    birdY += birdVel;

    // Check for hitting floor or ceiling
    if (birdY + BIRD_HEIGHT > canvas.height - images.base.height || birdY < 0) {
      gameOver = true;
    }

    // Move pipes and check collisions
    for (const pipe of pipes) {
      pipe.x -= 4;

      // Collision detection with pipes
      if (
        BIRD_X + BIRD_WIDTH > pipe.x &&
        BIRD_X < pipe.x + PIPE_WIDTH &&
        (
          birdY < pipe.y + images.pipe.height ||
          birdY + BIRD_HEIGHT > pipe.y + images.pipe.height + PIPE_GAP
        )
      ) {
        gameOver = true;
      }

      // Score if pipe passed
      if (!pipe.passed && pipe.x + PIPE_WIDTH < BIRD_X) {
        pipe.passed = true;
        score++;
      }
    }

    // Remove offscreen pipes
    pipes = pipes.filter(pipe => pipe.x + PIPE_WIDTH > 0);

    // Add pipes every ~300px gap
    if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 300) {
      createPipe();
    }
  }

  // Draw everything
  function draw() {
    // Background scaled to canvas size
    ctx.drawImage(images.bg, 0, 0, canvas.width, canvas.height);

    // Draw pipes
    for (const pipe of pipes) {
      // Top pipe (flipped)
      ctx.save();
      ctx.translate(pipe.x + PIPE_WIDTH / 2, pipe.y + images.pipe.height);
      ctx.scale(1, -1);
      ctx.drawImage(images.pipe, -PIPE_WIDTH / 2, 0, PIPE_WIDTH, images.pipe.height);
      ctx.restore();

      // Bottom pipe
      ctx.drawImage(images.pipe, pipe.x, pipe.y + images.pipe.height + PIPE_GAP, PIPE_WIDTH, images.pipe.height);
    }

    // Draw base on bottom (scaled width)
    ctx.drawImage(images.base, 0, canvas.height - images.base.height, canvas.width, images.base.height);

    // Draw bird
    ctx.drawImage(images.bird, BIRD_X, birdY, BIRD_WIDTH, BIRD_HEIGHT);

    // Draw score
    ctx.fillStyle = 'white';
    ctx.font = '40px Arial';
    ctx.fillText(`Score: ${score}`, 20, 50);

    if (gameOver) {
      ctx.fillStyle = 'yellow';
      ctx.font = '50px Arial';
      ctx.fillText('Game Over! Press Space to Restart', 50, canvas.height / 2);
    }
  }

  // Game loop
  function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
  }

  // Spacebar controls
  window.addEventListener('keydown', e => {
    if (e.code === 'Space') {
      if (gameOver) reset();
      birdVel = FLAP;
    }
  });

  // Start everything
  loadAllImages().then(() => {
    reset();
    loop();
  });
})();
</script>
</body>
</html>
