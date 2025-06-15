<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Flappy Bird Temple</title>
  <style>
    body { margin:0; overflow:hidden; }
    canvas { display:block; margin:auto; background:#87CEEB; }
  </style>
</head>
<body>
<canvas id="game" width="288" height="512"></canvas>
<script>
const canvas = document.getElementById('game'),
      ctx = canvas.getContext('2d');

const GRAVITY=0.25, FLAP=-4.6, GAP=120,
      BWIDTH=34, BHEIGHT=24, PWIDTH=52;

let birdY=150, birdVel=0, pipes=[], score=0, gameOver=false;

// Use reliable assets
const assets={
  bg:'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/background-day.png',
  base:'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/base.png',
  bird:'https://raw.githubusercontent.com/samuelcust/flappy-bird-assets/master/sprites/yellowbird-midflap.png',
  pillarTop:'https://i.imgur.com/WnDSKoq.png',
  pillarBottom:'https://i.imgur.com/WnDSKoq.png'
};

const sprites={}, total=Object.keys(assets).length;
let loaded=0;

// Preload
for(let k in assets){
  sprites[k]=new Image();
  sprites[k].src=assets[k];
  sprites[k].onload=
    ()=>{
      if(++loaded===total) loop();
    };
}

function reset(){
  birdY=150; birdVel=0; pipes=[]; score=0; gameOver=false;
}

function update(){
  if(gameOver)return;
  birdVel+=GRAVITY; birdY+=birdVel;
  if(birdY + BHEIGHT > canvas.height - sprites.base.height || birdY < 0) gameOver=true;

  for(let p of pipes){
    p.x-=2;
    if(!p.passed && p.x + PWIDTH < 50){
      p.passed=true; score++;
    }
    if(50 + BWIDTH > p.x && 50 < p.x+PWIDTH &&
       (birdY < p.y+320 || birdY + BHEIGHT > p.y+320+GAP)){
      gameOver=true;
    }
  }

  const last = pipes[pipes.length-1];
  if(!last || last.x < 150){
    let y = -Math.floor(Math.random()*150) - 50;
    pipes.push({x:canvas.width, y, passed:false});
  }
  pipes = pipes.filter(p=>p.x + PWIDTH > 0);
}

function draw(){
  ctx.drawImage(sprites.bg,0,0);
  for(let p of pipes){
    ctx.save();
    ctx.translate(p.x + PWIDTH/2, p.y + 320);
    ctx.scale(1,-1);
    ctx.drawImage(sprites.pillarTop, -PWIDTH/2, 0, PWIDTH, 320);
    ctx.restore();
    ctx.drawImage(sprites.pillarBottom, p.x, p.y+320+GAP, PWIDTH, 320);
  }
  ctx.drawImage(sprites.base,0,canvas.height - sprites.base.height);
  ctx.drawImage(sprites.bird,50,birdY,BWIDTH,BHEIGHT);

  ctx.fillStyle='white';
  ctx.font='20px Arial';
  ctx.fillText('Score: '+score,10,canvas.height-20);
  if(gameOver) ctx.fillText('Game Over! Click to restart',30,canvas.height/2);
}

function loop(){
  update(); draw();
  requestAnimationFrame(loop);
}

canvas.addEventListener('click', ()=>{
  if(gameOver) reset(); else birdVel=FLAP;
});
</script>
</body>
</html>

