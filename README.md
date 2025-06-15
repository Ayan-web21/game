<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Flappy Bird Fullscreen</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      height: 100%;
      background: #70c5ce;
    }
    canvas {
      display: block;
      width: 100vw;
      height: 100vh;
    }
  </style>
</head>
<body>
<canvas id="game"></canvas>
<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  const GRAVITY = 0.4;
  const FLAP = -8;
  const PIPE_GAP = 180;
  const PIPE_WIDTH = 80;
  const BIRD_WIDTH = 40;
  const BIRD_HEIGHT = 30;
  const BIRD_X = 100;

  let birdY = canvas.height / 2;
  let birdVel = 0;
  let pipes = [];
  let score = 0;
  let gameOver = false;

  const images = {};
  const assets = {
    bg: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/background-day.png',
    base: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/base.png',
    bird: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/yellowbird-midflap.png',
    pipe: 'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/pipe-green.png',
  };

  function loadImage(src) {
    return new Promise(resolve => {
      const img = new Image();
      img.src = src;
      img.onload = () => resolve(img);
    });
  }

  async function loadAssets() {
    for (let key in assets) {
      images[key] = await loadImage(assets[key]);
    }
  }

  function reset() {
    birdY = canvas.height / 2;
    birdVel = 0;
    pipes = [];
    score = 0;
    gameOver = false;
  }

  function createPipe() {
    const y = -Math.floor(Math.random() * (images.pipe.height - 100));
    pipes.push({ x: canvas.width, y: y, passed: false });
  }

  function update() {
    if (gameOver) return;

    birdVel += GRAVITY;
    birdY += birdVel;

    if (birdY + BIRD_HEIGHT > canvas.height - images.base.height || birdY < 0) {
      gameOver = true;
    }

    for (let pipe of pipes) {
      pipe.x -= 4;

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

      if (!pipe.passed && pipe.x + PIPE_WIDTH < BIRD_X) {
        pipe.passed = true;
        score++;
      }
    }

    pipes = pipes.filter(p => p.x + PIPE_WIDTH > 0);

    if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 300) {
      createPipe();
    }
  }

  function draw() {
    ctx.drawImage(images.bg, 0, 0, canvas.width, canvas.height);

    for (let p of pipes) {
      // Top pipe (flipped)
      ctx.save();
      ctx.translate(p.x + PIPE_WIDTH / 2, p.y + images.pipe.height);
      ctx.scale(1, -1);
      ctx.drawImage(images.pipe, -PIPE_WIDTH / 2, 0, PIPE_WIDTH, images.pipe.height);
      ctx.restore();

      // Bottom pipe
      ctx.drawImage(images.pipe, p.x, p.y + images.pipe.height + PIPE_GAP, PIPE_WIDTH, images.pipe.height);
    }

    ctx.drawImage(images.base, 0, canvas.height - images.base.height, canvas.width, images.base.height);

    ctx.drawImage(images.bird, BIRD_X, birdY, BIRD_WIDTH, BIRD_HEIGHT);

    ctx.fillStyle = 'white';
    ctx.font = '40px Arial';
    ctx.fillText(`Score: ${score}`, 20, 50);

    if (gameOver) {
      ctx.fillStyle = 'yellow';
      ctx.font = '40px Arial';
      ctx.fillText('Game Over! Tap or Press Space to Restart', canvas.width / 2 - 300, canvas.height / 2);
    }
  }

  function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
  }

  // Controls: spacebar, click, tap
  window.addEventListener('keydown', e => {
    if (e.code === 'Space') {
      if (gameOver) reset();
      birdVel = FLAP;
    }
  });

  window.addEventListener('mousedown', () => {
    if (gameOver) reset();
    birdVel = FLAP;
  });

  window.addEventListener('touchstart', () => {
    if (gameOver) reset();
    birdVel = FLAP;
  });

  loadAssets().then(() => {
    reset();
    loop();
  });
})();
</script>
</body>
</html>
