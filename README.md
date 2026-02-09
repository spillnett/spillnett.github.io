<html lang="no">
<head>
<meta charset="UTF-8">
<title>Neon Runner</title>
<style>
  body { margin:0; background:#000; display:flex; justify-content:center; align-items:center; height:100vh; }
  canvas { background:#0b1020; border:2px solid #4af; touch-action:none; }
  .hint { position:fixed; bottom:10px; color:#9cf; font-family:Arial; font-size:14px; }
</style>
</head>
<body>
<canvas id="game" width="480" height="640"></canvas>
<div class="hint">← → / A D eller touch • Hopp: ↑ / W / touch</div>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

let player = { x:180, y:520, w:28, h:28, vx:0, vy:0, onGround:false };
let platforms = [];
let gravity = 0.6;
let score = 0;
let gameOver = false;

function reset() {
  player.x = canvas.width/2 - player.w/2; player.y = 520; player.vx = 0; player.vy = 0;
  platforms = [];
  // Startplattform (dekker hele bunnen)
  platforms.push({ x:0, y:600, w:canvas.width, h:16 });
  // Vanlige plattformer over hele bredden
  for (let i=1;i<6;i++) {
    platforms.push({ x:Math.random()*(canvas.width-100), y:600-i*120, w:80+Math.random()*80, h:12 });
  }
  score = 0; gameOver = false;
}
reset();

// Tastatur
let keys = {};
addEventListener('keydown', e=> keys[e.key]=true);
addEventListener('keyup', e=> keys[e.key]=false);

// Touch
let touchX = null;
canvas.addEventListener('touchstart', e=>{
  const t = e.touches[0];
  touchX = t.clientX;
  if (player.onGround) { player.vy = -12; player.onGround = false; }
});
canvas.addEventListener('touchmove', e=>{
  const t = e.touches[0];
  touchX = t.clientX;
});
canvas.addEventListener('touchend', ()=> touchX = null);

function update() {
  if (gameOver) return;

  // Input
  if (keys['ArrowLeft'] || keys['a']) player.vx = -4;
  else if (keys['ArrowRight'] || keys['d']) player.vx = 4;
  else if (touchX!==null) {
    const rect = canvas.getBoundingClientRect();
    const x = touchX - rect.left;
    player.vx = x < canvas.width/2 ? -4 : 4;
  } else player.vx = 0;

  if ((keys['ArrowUp'] || keys['w']) && player.onGround) {
    player.vy = -12; player.onGround = false;
  }

  // Fysikk
  player.vy += gravity;
  player.x += player.vx;
  player.y += player.vy;

  // Vegger
  if (player.x<0) player.x=0;
  if (player.x+player.w>canvas.width) player.x=canvas.width-player.w;

  // Plattform-kollisjon
  player.onGround = false;
  for (let p of platforms) {
    if (player.x < p.x+p.w && player.x+player.w > p.x &&
        player.y+player.h < p.y+p.h && player.y+player.h+player.vy >= p.y) {
      player.y = p.y-player.h;
      player.vy = 0;
      player.onGround = true;
    }
  }

  // Scroll
  if (player.y < 300) {
    const dy = 300 - player.y;
    player.y = 300;
    for (let p of platforms) p.y += dy;
    score += Math.floor(dy);
  }

  // Nye plattformer
  platforms = platforms.filter(p=>p.y<700);
  while (platforms.length < 6) {
    platforms.push({
      x: Math.random() * (canvas.width - 120),
      y: -Math.random() * 140,
      w: 80 + Math.random() * 120,
      h: 12
    });
  });
  }

  if (player.y > canvas.height) gameOver = true;
}

function draw() {
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Player
  ctx.fillStyle = '#4af';
  ctx.fillRect(player.x, player.y, player.w, player.h);

  // Plattformer
  ctx.fillStyle = '#8ff';
  for (let p of platforms) ctx.fillRect(p.x, p.y, p.w, p.h);

  // UI
  ctx.fillStyle = '#fff';
  ctx.font = '16px Arial';
  ctx.fillText('Score: '+score, 10, 20);

  if (gameOver) {
    ctx.fillStyle = '#f66';
    ctx.font = '24px Arial';
    ctx.fillText('Game Over', 110, 300);
    ctx.font = '16px Arial';
    ctx.fillText('Trykk R for restart', 100, 330);
  }
}

addEventListener('keydown', e=>{ if (e.key==='r') reset(); });

function loop() {
  update(); draw(); requestAnimationFrame(loop);
}
loop();
</script>
</body>
</html>
