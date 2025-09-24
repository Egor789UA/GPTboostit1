<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Мульти-Аркада</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0;background:#111;overflow:hidden;font-family:sans-serif;color:white;}
canvas{display:block;}
#hud{position:absolute;top:5px;left:10px;font-size:16px;}
#settings{position:absolute;top:40px;left:10px;}
#menu{position:absolute;top:90px;left:10px;}
#mobile-controls{position:absolute;bottom:10px;width:100%;display:flex;justify-content:space-around;}
button.control{width:60px;height:60px;font-size:24px;opacity:0.3;background:#333;color:white;border:none;border-radius:10px;}
button{margin:2px;}
#nicknameModal{position:absolute;top:40%;left:50%;transform:translate(-50%,-50%);
background:#222;padding:20px;border-radius:10px;display:none;color:white;text-align:center;}
#nicknameModal input{padding:5px;margin:5px;}
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="hud">
  Нік: <span id="nickDisplay">?</span> | 
  FPS: <span id="fpsCounter">0</span> | 
  Score: <span id="scoreCounter">0</span> | 
  Health: <span id="healthCounter">3</span>
</div>

<div id="settings">
  <label>Max FPS: <input type="number" id="maxFPSInput" value="144" min="30" max="240"></label>
  <button onclick="togglePause()">Пауза</button>
  <button onclick="resetGame()">Скинути</button>
</div>

<div id="menu">
  <button onclick="switchGame('flappy')">Flappy Bird</button>
  <button onclick="switchGame('road')">Безкінечна дорога</button>
  <button onclick="switchGame('autogun')">Автострілялка</button>
  <button onclick="switchGame('blocks')">Падаючі блоки</button>
  <button onclick="switchGame('breakout')">Breakout</button>
  <button onclick="switchGame('sound')">Sound Control</button>
</div>

<div id="mobile-controls">
  <button class="control" id="left">◀</button>
  <button class="control" id="jump">▲</button>
  <button class="control" id="right">▶</button>
</div>

<div id="nicknameModal">
  <h3>Введіть свій нік</h3>
  <input type="text" id="nicknameInput" placeholder="Ваш нік">
  <button onclick="setNickname()">OK</button>
</div>

<script>
const canvas=document.getElementById('game');
const ctx=canvas.getContext('2d');
canvas.width=window.innerWidth;
canvas.height=window.innerHeight;

let fps=0,frames=0,lastTime=performance.now(),lastFrameTime=performance.now();
let maxFPS=144,frameInterval=1000/maxFPS;
let score=0,health=3,paused=false,currentGame='flappy';
const keys={};
let nickname = localStorage.getItem("arcade_nick") || "";

// ======= Події =======
document.addEventListener('keydown', e=>keys[e.code]=true);
document.addEventListener('keyup', e=>keys[e.code]=false);
document.getElementById('maxFPSInput').addEventListener('change',(e)=>{
  let val=parseInt(e.target.value); if(val>=30&&val<=240){maxFPS=val;frameInterval=1000/maxFPS;}
});
document.getElementById('left').addEventListener('touchstart',()=>keys['ArrowLeft']=true);
document.getElementById('left').addEventListener('touchend',()=>keys['ArrowLeft']=false);
document.getElementById('right').addEventListener('touchstart',()=>keys['ArrowRight']=true);
document.getElementById('right').addEventListener('touchend',()=>keys['ArrowRight']=false);
document.getElementById('jump').addEventListener('touchstart',()=>{if(currentGame==='flappy') flappyJump();if(currentGame==='sound') soundTrigger();});
document.addEventListener('mousedown',()=>{if(currentGame==='flappy') flappyJump();if(currentGame==='sound') soundTrigger();});

// ======= Нік =======
function setNickname(){
  const input=document.getElementById("nicknameInput").value.trim();
  if(input){
    nickname=input;
    localStorage.setItem("arcade_nick", nickname);
    document.getElementById("nicknameModal").style.display="none";
    document.getElementById("nickDisplay").innerText=nickname;
  }
}
window.onload=()=>{
  if(!nickname){
    document.getElementById("nicknameModal").style.display="block";
  } else {
    document.getElementById("nickDisplay").innerText=nickname;
  }
};

// ======= HUD =======
function updateHUD(){
  document.getElementById('fpsCounter').innerText=fps;
  document.getElementById('scoreCounter').innerText=score;
  document.getElementById('healthCounter').innerText=health;
  document.getElementById('nickDisplay').innerText=nickname||"?";
}
function togglePause(){paused=!paused;alert(paused?"Гра на паузі":"Гра відновлена");}
function resetGame(){score=0;health=3;initGame();}

// ======= Меню =======
function switchGame(name){currentGame=name;score=0;health=3;initGame();}
function initGame(){
  if(currentGame==='flappy') initFlappy();
  if(currentGame==='road') initRoad();
  if(currentGame==='autogun') initAutoGun();
  if(currentGame==='blocks') initBlocks();
  if(currentGame==='breakout') initBreakout();
  if(currentGame==='sound') initSound();
}

// ======= Loop =======
function loop(now){
  requestAnimationFrame(loop);
  if(paused) return;
  const delta=now-lastFrameTime; if(delta<frameInterval) return; lastFrameTime=now;
  frames++; if(now-lastTime>=1000){fps=frames;frames=0;lastTime=now;}
  ctx.clearRect(0,0,canvas.width,canvas.height);

  if(currentGame==='flappy'){updateFlappy();drawFlappy();}
  if(currentGame==='road'){updateRoad();drawRoad();}
  if(currentGame==='autogun'){updateAutoGun();drawAutoGun();}
  if(currentGame==='blocks'){updateBlocks();drawBlocks();}
  if(currentGame==='breakout'){updateBreakout();drawBreakout();}
  if(currentGame==='sound'){updateSound();drawSound();}

  updateHUD();
  if(health<=0){alert("Game Over! Score: "+score);resetGame();}
}
initGame();
loop(performance.now());

// ======= ІГРИ =======
// Flappy Bird
let birdY,velocity,pipes;
function initFlappy(){birdY=canvas.height/2;velocity=0;pipes=[];}
function flappyJump(){velocity=-8;}
function updateFlappy(){velocity+=0.5;birdY+=velocity;if(birdY>canvas.height)health=0;
  if(Math.random()<0.02)pipes.push({x:canvas.width,y:Math.random()*canvas.height/2+50});
  pipes.forEach(p=>p.x-=5);
  pipes=pipes.filter(p=>p.x>-50);
  pipes.forEach(p=>{if(p.x<60&&birdY<p.y-100||p.x<60&&birdY>p.y)health=0;});
  score++;
}
function drawFlappy(){ctx.fillStyle="yellow";ctx.fillRect(50,birdY,30,30);
  ctx.fillStyle="green";pipes.forEach(p=>{ctx.fillRect(p.x,0,50,p.y-100);ctx.fillRect(p.x,p.y,50,canvas.height);});
}

// Road
let carX,enemies;
function initRoad(){carX=canvas.width/2;enemies=[];}
function updateRoad(){if(keys['ArrowLeft'])carX-=5;if(keys['ArrowRight'])carX+=5;
  if(Math.random()<0.03)enemies.push({x:Math.random()*canvas.width,y:0});
  enemies.forEach(e=>e.y+=5);
  enemies=enemies.filter(e=>e.y<canvas.height);
  enemies.forEach(e=>{if(Math.abs(e.x-carX)<30&&e.y>canvas.height-100)health--;});
  score++;
}
function drawRoad(){ctx.fillStyle="blue";ctx.fillRect(carX,canvas.height-80,40,80);
  ctx.fillStyle="red";enemies.forEach(e=>ctx.fillRect(e.x,e.y,40,40));
}

// AutoGun
let playerX,bullets,targets;
function initAutoGun(){playerX=canvas.width/2;bullets=[];targets=[];}
function updateAutoGun(){if(keys['ArrowLeft'])playerX-=5;if(keys['ArrowRight'])playerX+=5;
  if(Math.random()<0.05)targets.push({x:Math.random()*canvas.width,y:0});
  if(Math.random()<0.2)bullets.push({x:playerX,y:canvas.height-50});
  bullets.forEach(b=>b.y-=10);
  targets.forEach(t=>t.y+=3);
  bullets.forEach(b=>targets.forEach(t=>{if(Math.abs(b.x-t.x)<20&&Math.abs(b.y-t.y)<20){score+=10;t.y=canvas.height+100;}}));
  targets=targets.filter(t=>t.y<canvas.height);
}
function drawAutoGun(){ctx.fillStyle="white";ctx.fillRect(playerX,canvas.height-40,40,40);
  ctx.fillStyle="yellow";bullets.forEach(b=>ctx.fillRect(b.x,b.y,5,10));
  ctx.fillStyle="red";targets.forEach(t=>ctx.fillRect(t.x,t.y,30,30));
}

// Blocks
let falling;
function initBlocks(){falling=[];}
function updateBlocks(){if(Math.random()<0.05)falling.push({x:Math.random()*canvas.width,y:0});
  falling.forEach(f=>f.y+=5);
  falling.forEach(f=>{if(f.y>canvas.height-20){health--;f.y=canvas.height+100;}});
  falling=falling.filter(f=>f.y<canvas.height);
  score++;
}
function drawBlocks(){ctx.fillStyle="lime";falling.forEach(f=>ctx.fillRect(f.x,f.y,30,30));}

// Breakout
let paddleX,ballX,ballY,dx,dy,bricks;
function initBreakout(){paddleX=canvas.width/2;ballX=canvas.width/2;ballY=canvas.height/2;dx=4;dy=-4;bricks=[];
  for(let i=0;i<5;i++)for(let j=0;j<8;j++)bricks.push({x:80*j+20,y:30*i+20,alive:true});
}
function updateBreakout(){if(keys['ArrowLeft'])paddleX-=7;if(keys['ArrowRight'])paddleX+=7;
  ballX+=dx;ballY+=dy;
  if(ballX<0||ballX>canvas.width)dx=-dx;if(ballY<0)dy=-dy;
  if(ballY>canvas.height){health--;ballY=canvas.height/2;}
  if(ballX>paddleX&&ballX<paddleX+100&&ballY>canvas.height-30)dy=-dy;
  bricks.forEach(b=>{if(b.alive&&ballX>b.x&&ballX<b.x+70&&ballY>b.y&&ballY<b.y+20){b.alive=false;dy=-dy;score+=5;}});
}
function drawBreakout(){ctx.fillStyle="white";ctx.fillRect(paddleX,canvas.height-20,100,10);
  ctx.beginPath();ctx.arc(ballX,ballY,10,0,Math.PI*2);ctx.fill();
  ctx.fillStyle="orange";bricks.forEach(b=>{if(b.alive)ctx.fillRect(b.x,b.y,70,20);});
}

// Sound Control
let soundActive=false;
function initSound(){}
function updateSound(){if(soundActive){score++;soundActive=false;}}
function drawSound(){ctx.fillStyle="cyan";ctx.fillText("Клацни або торкнись, щоб заробити очки!",50,100);}
function soundTrigger(){soundActive=true;}
</script>
</body>
</html>
