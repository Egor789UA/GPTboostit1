<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>GPTboostit Аркада</title>
  <style>
    body { margin:0; background:#000; color:#fff; font-family:sans-serif; text-align:center; }
    canvas { background:#111; display:block; margin:0 auto; }
    #hud { padding:10px; font-size:14px; }
    #settings { margin:10px; }
    button { margin:3px; padding:5px 10px; cursor:pointer; }
    #nicknameModal {
      position:absolute;top:40%;left:50%;transform:translate(-50%,-50%);
      background:#222;padding:20px;border-radius:10px;color:white;z-index:1000;
    }
    #leaderboard { background:#111; padding:10px; margin-top:10px; }
    .touchControls {position:fixed;bottom:20px;left:0;right:0;display:flex;justify-content:space-between;pointer-events:none;}
    .touchBtn {width:60px;height:60px;background:#444a;pointer-events:auto;border-radius:10px;}
  </style>
</head>
<body>

<div id="hud">
  FPS: <span id="fpsCounter">0</span> | 
  Score: <span id="scoreCounter">0</span> | 
  Health: <span id="healthCounter">3</span> | 
  Нік: <span id="nickHUD"></span>
</div>

<div id="settings">
  <label>Max FPS: <input type="number" id="maxFPSInput" value="144" min="30" max="240"></label>
  <button onclick="togglePause()">Пауза</button>
  <button onclick="resetGame()">Скинути гру</button>
</div>

<div>
  <button onclick="switchGame('flappy')">Flappy Bird</button>
  <button onclick="switchGame('road')">Безкінечна дорога</button>
  <button onclick="switchGame('autogun')">Автострілялка</button>
  <button onclick="switchGame('blocks')">Падаючі блоки</button>
  <button onclick="switchGame('breakout')">Breakout</button>
  <button onclick="switchGame('sound')">Звуковий контроль</button>
  <button onclick="showLeaders()">Показати лідерів</button>
</div>

<canvas id="gameCanvas" width="480" height="640"></canvas>

<div id="leaderboard" style="display:none;"></div>

<!-- Вікно для ніку -->
<div id="nicknameModal">
  <h3>Введіть свій нік</h3>
  <input type="text" id="nicknameInput" placeholder="Ваш нік">
  <button onclick="setNickname()">OK</button>
</div>

<!-- Мобільні кнопки -->
<div class="touchControls">
  <div class="touchBtn" id="leftBtn"></div>
  <div class="touchBtn" id="upBtn"></div>
  <div class="touchBtn" id="rightBtn"></div>
</div>

<script>
const canvas=document.getElementById("gameCanvas"),ctx=canvas.getContext("2d");

// --- Нікнейм ---
let nickname=localStorage.getItem("arcade_nick")||"";
function setNickname(){
  const val=document.getElementById("nicknameInput").value.trim();
  if(val){
    nickname=val;
    localStorage.setItem("arcade_nick",nickname);
    document.getElementById("nicknameModal").style.display="none";
    document.getElementById("nickHUD").innerText=nickname;
  }
}
window.onload=()=>{
  if(!nickname){
    document.getElementById("nicknameModal").style.display="block";
  } else {
    document.getElementById("nicknameModal").style.display="none";
    document.getElementById("nickHUD").innerText=nickname;
  }
};

// --- HUD змінні ---
let fps=0,score=0,health=3;
let frames=0,lastTime=0,lastFrameTime=0;
let maxFPS=144,frameInterval=1000/maxFPS,paused=false;

// --- Глобальні ---
let currentGame="flappy";
let leaders=JSON.parse(localStorage.getItem("leaders")||"{}");

// --- HUD функції ---
function updateHUD(){
  document.getElementById('fpsCounter').innerText=fps;
  document.getElementById('scoreCounter').innerText=score;
  document.getElementById('healthCounter').innerText=health;
  document.getElementById('nickHUD').innerText=nickname;
}

function togglePause(){ paused=!paused; }
function resetGame(){ score=0; health=3; initGame(); }

// --- Збереження ---
function saveScore(){
  if(!leaders[currentGame]) leaders[currentGame]=[];
  leaders[currentGame].push({nick:nickname,score:score});
  leaders[currentGame].sort((a,b)=>b.score-a.score);
  leaders[currentGame]=leaders[currentGame].slice(0,10);
  localStorage.setItem("leaders",JSON.stringify(leaders));
}
function showLeaders(){
  let box=document.getElementById("leaderboard");
  box.innerHTML="<h3>Лідери</h3>";
  for(const g in leaders){
    box.innerHTML+=`<h4>${g}</h4><ol>`+leaders[g].map(l=>`<li>${l.nick}: ${l.score}</li>`).join("")+"</ol>";
  }
  box.style.display="block";
}

// --- Ігри (спрощені приклади) ---
function initGame(){}
function updateFlappy(){score++;ctx.fillStyle="yellow";ctx.fillRect(200,200,30,30);}
function drawFlappy(){}
function updateRoad(){score++;ctx.fillStyle="red";ctx.fillRect(200,300,40,40);}
function drawRoad(){}
function updateAutoGun(){score++;ctx.fillStyle="blue";ctx.fillRect(100,100,20,20);}
function drawAutoGun(){}
function updateBlocks(){score++;ctx.fillStyle="green";ctx.fillRect(250,250,30,30);}
function drawBlocks(){}
function updateBreakout(){score++;ctx.fillStyle="orange";ctx.fillRect(150,150,60,20);}
function drawBreakout(){}
function updateSound(){score++;ctx.fillStyle="purple";ctx.fillRect(50,400,40,40);}
function drawSound(){}

function switchGame(g){ currentGame=g; resetGame(); }

// --- Цикл ---
function loop(now){
  requestAnimationFrame(loop);
  if(paused) return;
  const delta=now-lastFrameTime;
  if(delta<frameInterval) return;
  lastFrameTime=now;

  frames++;
  if(now-lastTime>=1000){ fps=frames; frames=0; lastTime=now; }

  ctx.clearRect(0,0,canvas.width,canvas.height);
  if(currentGame==="flappy"){updateFlappy();drawFlappy();}
  else if(currentGame==="road"){updateRoad();drawRoad();}
  else if(currentGame==="autogun"){updateAutoGun();drawAutoGun();}
  else if(currentGame==="blocks"){updateBlocks();drawBlocks();}
  else if(currentGame==="breakout"){updateBreakout();drawBreakout();}
  else if(currentGame==="sound"){updateSound();drawSound();}

  updateHUD();
  if(health<=0){alert("Game Over! Score:"+score);saveScore();resetGame();}
}
loop(performance.now());

// --- FPS контроль ---
document.getElementById('maxFPSInput').addEventListener('change',e=>{
  let val=parseInt(e.target.value);
  if(val>=30&&val<=240){ maxFPS=val; frameInterval=1000/maxFPS; }
});

// --- Мобільне керування ---
document.getElementById("leftBtn").addEventListener("touchstart",()=>{ /* left */ });
document.getElementById("upBtn").addEventListener("touchstart",()=>{ /* jump */ });
document.getElementById("rightBtn").addEventListener("touchstart",()=>{ /* right */ });
</script>
</body>
</html>
