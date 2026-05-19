<!DOCTYPE html>
<html>
<head>
<title>Ultimate Road Game</title>
<style>
body { margin:0; overflow:hidden; background:#000; font-family:Arial; }
canvas { display:block; }

/* UI */
#ui {
  position:absolute;
  top:10px;
  left:10px;
  color:white;
  font-size:16px;
}

/* SHOP */
#shop {
  position:absolute;
  right:10px;
  top:10px;
  background:rgba(0,0,0,0.7);
  color:white;
  padding:10px;
  border-radius:10px;
  width:160px;
}

button { width:100%; margin-top:5px; cursor:pointer; }

/* GAME OVER */
#over {
  position:absolute;
  top:0;
  left:0;
  width:100%;
  height:100%;
  display:none;
  background:rgba(0,0,0,0.8);
  color:white;
  justify-content:center;
  align-items:center;
  flex-direction:column;
}
#over button {
  padding:10px 20px;
  font-size:18px;
}
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="ui">
Score: <span id="score">0</span><br>
Coins: <span id="coins">0</span>
</div>

<div id="shop">
🏪 SHOP<br>
<button onclick="buy('blue')">Blue (Free)</button>
<button onclick="buy('green')">Green (5)</button>
<button onclick="buy('gold')">Gold (10)</button>
<hr>
Skin: <span id="skin">blue</span>
</div>

<div id="over">
<h1>💥 Game Over</h1>
<p id="final"></p>
<button onclick="restart()">🔄 Try Again</button>
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

canvas.width=innerWidth;
canvas.height=innerHeight;

// 🔊 SOUND
const audio=new (window.AudioContext||window.webkitAudioContext)();
function sound(type){
  let o=audio.createOscillator();
  let g=audio.createGain();
  o.connect(g); g.connect(audio.destination);

  if(type==="coin"){o.frequency.value=900; g.gain.value=0.1;}
  if(type==="crash"){o.frequency.value=100; g.gain.value=0.2;}

  o.start();
  o.stop(audio.currentTime+0.1);
}

// GAME DATA
let car,obs,coins,road,score,coinCount,skin,run;

// INIT
function init(){
  car={x:canvas.width/2,y:canvas.height-140,w:50,h:80,vy:0,j:false};
  obs=[]; coins=[]; road=[];
  score=0; run=true;

  for(let i=0;i<20;i++) road.push(i*80);
}

coinCount=parseInt(localStorage.getItem("coins"))||0;
skin=localStorage.getItem("skin")||"blue";

// CONTROLS (WASD + jump)
document.addEventListener("keydown",e=>{
  if(!run) return;

  if(e.key==="a") car.x-=40;
  if(e.key==="d") car.x+=40;

  if((e.key==="w"||e.key===" ")&&!car.j){
    car.vy=-18;
    car.j=true;
  }
});

// OBSTACLES
setInterval(()=>{
  if(run)
  obs.push({x:canvas.width/2+(Math.random()*200-100),y:-60});
},1200);

// COINS
setInterval(()=>{
  if(run)
  coins.push({x:canvas.width/2+(Math.random()*200-100),y:-40});
},900);

// SHOP
function buy(type){
  if(type==="blue") skin="blue";

  if(type==="green" && coinCount>=5){
    coinCount-=5;
    skin="green";
  }

  if(type==="gold" && coinCount>=10){
    coinCount-=10;
    skin="gold";
  }

  localStorage.setItem("coins",coinCount);
  localStorage.setItem("skin",skin);
}

// 🚗 CAR GRAPHICS
function drawCar(x,y){
  let c="blue";
  if(skin==="green") c="green";
  if(skin==="gold") c="gold";

  ctx.fillStyle=c;
  ctx.fillRect(x,y,50,80);

  // glass
  ctx.fillStyle="#87cefa";
  ctx.fillRect(x+10,y+10,30,20);

  // wheels
  ctx.fillStyle="#111";
  ctx.fillRect(x-5,y+10,10,20);
  ctx.fillRect(x-5,y+50,10,20);
  ctx.fillRect(x+45,y+10,10,20);
  ctx.fillRect(x+45,y+50,10,20);
}

// GAME OVER
function gameOver(){
  run=false;
  document.getElementById("over").style.display="flex";
  document.getElementById("final").innerText="Score: "+score;
  sound("crash");
}

// TRY AGAIN
function restart(){
  document.getElementById("over").style.display="none";
  init();
}

// LOOP
function loop(){
  ctx.fillStyle="#222";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  // road
  ctx.fillStyle="#444";
  ctx.fillRect(canvas.width/2-120,0,240,canvas.height);

  // road lines
  ctx.fillStyle="white";
  for(let i=0;i<road.length;i++){
    road[i]+=10;
    if(road[i]>canvas.height) road[i]=0;
    ctx.fillRect(canvas.width/2-5,road[i],10,40);
  }

  if(!run){requestAnimationFrame(loop);return;}

  // physics
  car.y+=car.vy;
  car.vy+=1.2;

  if(car.y>=canvas.height-140){
    car.y=canvas.height-140;
    car.j=false;
  }

  drawCar(car.x,car.y);

  // obstacles
  ctx.fillStyle="yellow";
  for(let i=0;i<obs.length;i++){
    obs[i].y+=8;
    ctx.fillRect(obs[i].x,obs[i].y,50,50);

    if(car.x<obs[i].x+50&&car.x+car.w>obs[i].x&&car.y<obs[i].y+50&&car.y+car.h>obs[i].y){
      gameOver();
    }

    if(obs[i].y>canvas.height){
      score++;
      obs.splice(i,1);
      i--;
    }
  }

  // coins
  ctx.fillStyle="gold";
  for(let i=0;i<coins.length;i++){
    coins[i].y+=8;

    ctx.beginPath();
    ctx.arc(coins[i].x,coins[i].y,10,0,Math.PI*2);
    ctx.fill();

    if(car.x<coins[i].x+10&&car.x+car.w>coins[i].x&&car.y<coins[i].y+10&&car.y+car.h>coins[i].y){
      coinCount++;
      sound("coin");
      coins.splice(i,1);
      i--;
    }
  }

  localStorage.setItem("coins",coinCount);

  // UI
  document.getElementById("score").innerText=score;
  document.getElementById("coins").innerText=coinCount;
  document.getElementById("skin").innerText=skin;

  requestAnimationFrame(loop);
}

init();
loop();
</script>

</body>
</html>
