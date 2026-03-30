<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Super Mario Style - ULTRA PRO</title>
<style>
  body { margin:0; overflow:hidden; background:#5c94fc; }
  canvas { display:block; }
</style>
</head>
<body>
<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const GRAVITY = 0.5;
const FRICTION = 0.85;

let cameraX = 0;
let score = 0;

// ===== LOAD SPRITES =====
const playerImg = new Image();
playerImg.src = "https://ibb.co.com/21JGtYWT/IMG-20260330-WA0091.jpg";

const enemyImg = new Image();
enemyImg.src = "https://i.imgur.com/QZ6XQ7T.png";

const coinImg = new Image();
coinImg.src = "https://i.imgur.com/8Qf6K2G.png";

// ===== PLAYER =====
class Player {
  constructor() {
    this.x = 100;
    this.y = 0;
    this.w = 40;
    this.h = 50;
    this.dx = 0;
    this.dy = 0;
    this.speed = 0.7;
    this.jump = -12;
    this.grounded = false;
    this.frame = 0;
    this.lives = 3;
  }

  update(platforms, enemies, coins) {
    this.dx *= FRICTION;
    this.dy += GRAVITY;

    this.x += this.dx;
    this.y += this.dy;

    this.grounded = false;

    platforms.forEach(p => {
      if (this.x < p.x+p.w &&
          this.x+this.w > p.x &&
          this.y < p.y+p.h &&
          this.y+this.h > p.y) {

        if (this.dy > 0) {
          this.y = p.y - this.h;
          this.dy = 0;
          this.grounded = true;
        }
      }
    });

    enemies.forEach(e => {
      if (!e.dead &&
          this.x < e.x+e.w &&
          this.x+this.w > e.x &&
          this.y < e.y+e.h &&
          this.y+this.h > e.y) {

        if (this.dy > 0) {
          e.dead = true;
          this.dy = -8;
          score += 50;
        } else {
          this.die();
        }
      }
    });

    coins.forEach(c => {
      if (!c.collected &&
          this.x < c.x+20 &&
          this.x+this.w > c.x &&
          this.y < c.y+20 &&
          this.y+this.h > c.y) {

        c.collected = true;
        score += 10;
      }
    });

    if (this.y > 1000) this.die();
  }

  die() {
    this.lives--;
    this.x = 100;
    this.y = 0;
    this.dx = 0;
    this.dy = 0;

    if (this.lives <= 0) {
      alert("GAME OVER");
      location.reload();
    }
  }

  draw() {
    this.frame += 0.2;
    ctx.drawImage(playerImg,
      (Math.floor(this.frame)%3)*32, 0,
      32, 32,
      this.x - cameraX, this.y,
      this.w, this.h);
  }
}

// ===== PLATFORM =====
class Platform {
  constructor(x,y,w,h){
    this.x=x;this.y=y;this.w=w;this.h=h;
  }

  draw(){
    ctx.fillStyle="#654321";
    ctx.fillRect(this.x-cameraX,this.y,this.w,this.h);
  }
}

// ===== ENEMY =====
class Enemy {
  constructor(x,y){
    this.x=x;
    this.y=y;
    this.w=40;
    this.h=40;
    this.dx=2;
    this.dead=false;
  }

  update(){
    this.x+=this.dx;
    if(this.x<0 || this.x>3000) this.dx*=-1;
  }

  draw(){
    if(!this.dead){
      ctx.drawImage(enemyImg,this.x-cameraX,this.y,this.w,this.h);
    }
  }
}

// ===== COIN =====
class Coin {
  constructor(x,y){
    this.x=x;
    this.y=y;
    this.collected=false;
  }

  draw(){
    if(!this.collected){
      ctx.drawImage(coinImg,this.x-cameraX,this.y,20,20);
    }
  }
}

// ===== LEVEL GENERATION =====
let level = {
  platforms: [],
  enemies: [],
  coins: []
};

for(let i=0;i<60;i++){
  level.platforms.push(new Platform(i*150,500,150,50));

  if(i%4===0){
    level.platforms.push(new Platform(i*150,400,80,20));
    level.enemies.push(new Enemy(i*150+40,460));
    level.coins.push(new Coin(i*150+20,350));
  }
}

let player = new Player();

// ===== INPUT =====
let keys={};
addEventListener("keydown",e=>keys[e.key]=true);
addEventListener("keyup",e=>keys[e.key]=false);

// ===== LOOP =====
function update(){
  if(keys["ArrowRight"]) player.dx+=player.speed;
  if(keys["ArrowLeft"]) player.dx-=player.speed;

  if(keys[" "] && player.grounded){
    player.dy = player.jump;
  }

  player.update(level.platforms, level.enemies, level.coins);

  level.enemies.forEach(e=>e.update());

  cameraX += (player.x - canvas.width/3 - cameraX) * 0.1;
}

function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  ctx.fillStyle="#87CEEB";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  level.platforms.forEach(p=>p.draw());
  level.enemies.forEach(e=>e.draw());
  level.coins.forEach(c=>c.draw());

  player.draw();

  // UI
  ctx.fillStyle="black";
  ctx.font="20px Arial";
  ctx.fillText("Score: "+score,20,30);
  ctx.fillText("Lives: "+player.lives,20,60);
}

function loop(){
  update();
  draw();
  requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
