<!DOCTYPE html>
<html>
<head>
<title>Ultimate Fighting Game</title>

<style>
body { margin:0; overflow:hidden; font-family:sans-serif; }

/* STORY SCREEN */
#story {
  position:absolute;
  width:100%;
  height:100%;
  background:black;
  color:white;
  display:flex;
  align-items:center;
  justify-content:center;
  flex-direction:column;
}

/* GAME */
#game {
  display:none;
  width:100vw;
  height:100vh;
  position:relative;
  background-size:cover;
}

/* PLAYER */
.player {
  position:absolute;
  width:80px;
  height:120px;
  background-size:cover;
}

/* FIREBALL */
.fireball {
  position:absolute;
  width:25px;
  height:25px;
  background:orange;
  border-radius:50%;
}

/* UI */
.barBox {
  position:absolute;
  top:10px;
  width:40%;
  height:20px;
  background:#444;
}
#h1 { left:10px; }
#h2 { right:10px; }
.bar { height:100%; background:lime; }

/* MESSAGE */
#message {
  position:absolute;
  top:40%;
  width:100%;
  text-align:center;
  font-size:40px;
  color:red;
  display:none;
}

/* CONTROLS */
.controls {
  position:absolute;
  bottom:0;
  width:50%;
  display:flex;
  flex-wrap:wrap;
}
button {
  width:33%;
  height:60px;
  font-size:18px;
}
#leftControls { left:0; }
#rightControls { right:0; }

</style>
</head>

<body>

<!-- STORY -->
<div id="story">
  <h1>🔥 Warrior Story 🔥</h1>
  <p>Defeat all enemies and final boss 👹</p>
  <button onclick="startGame()">START</button>
</div>

<!-- GAME -->
<div id="game">

  <div id="p1" class="player"></div>
  <div id="p2" class="player"></div>

  <div id="h1" class="barBox"><div id="bar1" class="bar"></div></div>
  <div id="h2" class="barBox"><div id="bar2" class="bar"></div></div>

  <div id="message"></div>

  <!-- MOBILE CONTROLS -->
  <div id="leftControls" class="controls">
    <button ontouchstart="keys.a=true" ontouchend="keys.a=false">←</button>
    <button ontouchstart="keys.d=true" ontouchend="keys.d=false">→</button>
    <button ontouchstart="keys.w=true">↑</button>
    <button ontouchstart="keys.f=true" ontouchend="keys.f=false">👊</button>
    <button ontouchstart="keys.g=true" ontouchend="keys.g=false">🔥</button>
  </div>

  <div id="rightControls" class="controls">
    <button ontouchstart="keys.left=true" ontouchend="keys.left=false">←</button>
    <button ontouchstart="keys.right=true" ontouchend="keys.right=false">→</button>
    <button ontouchstart="keys.up=true">↑</button>
    <button ontouchstart="keys.slash=true" ontouchend="keys.slash=false">👊</button>
  </div>

</div>

<script>
let level = 1;
let maxLevel = 3;

let p1, p2, keys = {}, fireballs = [], gameOver = false;

// START GAME
function startGame(){
  document.getElementById("story").style.display="none";
  document.getElementById("game").style.display="block";

  // PLAYER RESET
  p1 = {x:100,y:0,hp:100,vy:0,dir:1};
  p2 = {x:800,y:0,hp:100,vy:0,dir:-1,lastAttack:0};

  // IMAGES
  document.getElementById("p1").style.backgroundImage =
    "url('https://i.imgur.com/6X12F5G.png')";
  document.getElementById("p2").style.backgroundImage =
    "url('https://i.imgur.com/Y3aXbQW.png')";

  // BACKGROUND
  document.getElementById("game").style.backgroundImage =
    "url('https://i.imgur.com/3e5ZQwR.jpg')";

  spawnEnemy();
  update();
}

// SPAWN ENEMY
function spawnEnemy(){
  if(level === 3){
    p2.hp = 200; // BOSS
    showMessage("👹 BOSS FIGHT!");
  } else {
    p2.hp = 100 + level * 20;
    showMessage("Level " + level);
  }
}

// MESSAGE
function showMessage(text){
  let m = document.getElementById("message");
  m.innerText = text;
  m.style.display = "block";
  setTimeout(()=>m.style.display="none",1500);
}

// CONTROLS
document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

// FIREBALL
function createFireball(p){
  let f = document.createElement("div");
  f.className = "fireball";
  document.getElementById("game").appendChild(f);

  fireballs.push({el:f,x:p.x,y:p.y+50,dir:p.dir,owner:p});
}

// GAME LOOP
function update(){
  if(!p1 || !p2) return;
  if(gameOver) return;

  // GRAVITY
  [p1,p2].forEach(p=>{
    p.vy -= 1;
    p.y += p.vy;
    if(p.y < 0){ p.y = 0; p.vy = 0; }
  });

  // PLAYER MOVE
  if(keys.a){p1.x -= 5; p1.dir=-1;}
  if(keys.d){p1.x += 5; p1.dir=1;}
  if(keys.w && p1.y===0) p1.vy = 15;

  // ENEMY AI MOVE
  if(p2.x > p1.x) p2.x -= 2;
  else p2.x += 2;

  // ATTACK (PLAYER)
  if(keys.f && Math.abs(p1.x-p2.x)<80) p2.hp -= 2;

  if(keys.g){
    createFireball(p1);
    keys.g=false;
  }

  // ENEMY ATTACK (COOLDOWN FIX)
  let now = Date.now();
  if(Math.abs(p1.x - p2.x) < 70 && now - p2.lastAttack > 1000){
    p1.hp -= (level===3 ? 2 : 1);
    p2.lastAttack = now;
  }

  // FIREBALL MOVE
  fireballs.forEach((f,i)=>{
    f.x += 10 * f.dir;
    f.el.style.left = f.x + "px";
    f.el.style.bottom = f.y + "px";

    if(f.owner===p1 && Math.abs(f.x-p2.x)<50){
      p2.hp -= 5;
      f.el.remove();
      fireballs.splice(i,1);
    }
  });

  // UI
  document.getElementById("bar1").style.width = p1.hp + "%";
  document.getElementById("bar2").style.width = p2.hp + "%";

  // LOSE
  if(p1.hp <= 0){
    showMessage("💀 YOU LOSE");
    gameOver = true;
  }

  // WIN / NEXT LEVEL
  if(p2.hp <= 0){
    level++;
    if(level > maxLevel){
      showMessage("🏆 YOU WIN!");
      gameOver = true;
    } else {
      p1.hp = 100;
      spawnEnemy();
    }
  }

  // POSITION UPDATE
  let p1El = document.getElementById("p1");
  let p2El = document.getElementById("p2");

  p1El.style.left = p1.x + "px";
  p1El.style.bottom = p1.y + "px";

  p2El.style.left = p2.x + "px";
  p2El.style.bottom = p2.y + "px";

  requestAnimationFrame(update);
}
</script>

</body>
</html>
