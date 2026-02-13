
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>Neon Pro Clicker</title>
<style>
body{
  font-family: 'Arial', sans-serif;
  margin:0;
  padding-top:100px;
  background:#0a0a0f;
  color:#0ff;
  text-align:center;
  overflow-x:hidden;
}

/* ===== Header ===== */
.header{
  position:fixed;
  top:0;
  width:100%;
  background:#05050a;
  padding:12px 0;
  box-shadow:0 0 20px #0ff;
  z-index:100;
}

.header-row{
  display:flex;
  justify-content:space-around;
  align-items:center;
  font-size:14px;
  color:#0ff;
  text-shadow:0 0 5px #0ff;
}

.rank{
  padding:5px 12px;
  border-radius:20px;
  font-size:13px;
  font-weight:bold;
  text-shadow:0 0 8px #0ff;
}

/* ===== Progress Bar ===== */
.progress{
  width:80%;
  height:12px;
  background:#111;
  margin:8px auto 0;
  border-radius:10px;
  box-shadow:0 0 10px #0ff inset;
}

.progress-bar{
  height:100%;
  width:0%;
  background:#0ff;
  border-radius:10px;
  box-shadow:0 0 15px #0ff;
  transition:width 0.3s;
}

/* ===== Buttons ===== */
button{
  padding:10px 20px;
  margin:5px;
  font-size:14px;
  border:none;
  border-radius:8px;
  cursor:pointer;
  text-shadow:0 0 5px #0ff;
  box-shadow:0 0 10px #0ff;
  background:#111;
  color:#0ff;
  transition:0.2s;
}
button:hover{
  box-shadow:0 0 20px #0ff;
  text-shadow:0 0 10px #0ff;
}

/* ===== Box ===== */
#box{
  position:absolute;
  display:none;
  border-radius:10px;
  cursor:pointer;
  transition:0.1s;
}
#box:active{
  transform: scale(0.9);
  filter: drop-shadow(0 0 25px #0ff);
}

/* ===== Score & Timer ===== */
#score, #timer{
  font-size:20px;
  text-shadow:0 0 8px #0ff;
  transition:0.2s;
}
.glow{
  color:#ff0;
  text-shadow:0 0 12px #ff0, 0 0 20px #ff0;
}
</style>
</head>
<body>

<div class="header">
  <div class="header-row">
    <div>Level: <span id="level">1</span></div>
    <div>Coins: <span id="coins">0</span></div>
    <div id="rank" class="rank">مبتدئ</div>
  </div>
  <div class="progress">
    <div id="xpBar" class="progress-bar"></div>
  </div>
</div>

<h2>🎮 Neon Pro Clicker</h2>
<p id="score">النقاط: 0</p>
<p id="timer">الوقت: 30</p>

<button onclick="startGame()">ابدأ</button>
<button onclick="stopGame()">إيقاف</button>

<div id="box"></div>

<script>
// ===== Player Data =====
let level=parseInt(localStorage.getItem("level"))||1;
let xp=parseInt(localStorage.getItem("xp"))||0;
let coins=parseInt(localStorage.getItem("coins"))||0;
let rank="";

// ===== Game Data =====
let score=0;
let timeLeft=30;
let gameRunning=false;
let timerInterval;
let boxTimeout;
let comboCount=0;
let comboTimer;

// ===== DOM Elements =====
let box=document.getElementById("box");
let scoreEl=document.getElementById("score");

// ===== Update UI =====
function updateUI(){
  document.getElementById("level").innerText=level;
  document.getElementById("coins").innerText=coins;
  let needed=level*100;
  document.getElementById("xpBar").style.width=(xp/needed*100)+"%";
  updateRank();
}

// ===== Update Rank & Box Glow =====
function updateRank(){
  let rankEl=document.getElementById("rank");
  let color="#0ff";
  if(level>=50){ rank="👑 ملك السرعة"; color="gold"; rankEl.style.background="linear-gradient(45deg, gold, orange)";}
  else if(level>=35){ rank="🔴 أسطورة"; color="#ff0033"; rankEl.style.background=color;}
  else if(level>=20){ rank="🟡 خبير"; color="#ffcc00"; rankEl.style.background=color;}
  else if(level>=10){ rank="🟣 محترف"; color="#9900ff"; rankEl.style.background=color;}
  else if(level>=5){ rank="🔵 لاعب"; color="#00ccff"; rankEl.style.background=color;}
  else{ rank="🟢 مبتدئ"; color="#00ff66"; rankEl.style.background=color; rankEl.style.color="#000";}
  rankEl.innerText=rank;
  box.style.boxShadow=`0 0 25px ${color}, 0 0 50px ${color}`;
}

// ===== Start Game =====
function startGame(){
  if(gameRunning) return;
  score=0;
  timeLeft=30;
  gameRunning=true;
  comboCount=0;
  document.getElementById("score").innerText="النقاط: 0";
  document.getElementById("timer").innerText="الوقت: 30";
  timerInterval=setInterval(()=>{
    timeLeft--;
    document.getElementById("timer").innerText="الوقت: "+timeLeft;
    if(timeLeft<=0) endGame();
  },1000);
  showBox();
}

// ===== Stop Game =====
function stopGame(){
  if(!gameRunning) return;
  gameRunning=false;
  clearInterval(timerInterval);
  clearTimeout(boxTimeout);
  clearTimeout(comboTimer);
  box.style.display="none";
}

// ===== Show Box =====
function showBox(){
  if(!gameRunning) return;
  let size=Math.random()*40+40;
  box.style.width=size+"px";
  box.style.height=size+"px";
  box.style.left=Math.random()*(window.innerWidth-size)+"px";
  box.style.top=Math.random()*(window.innerHeight-size-100)+"px";
  box.style.background=getRandomColor();
  box.style.display="block";
  let speed=2000-(level*50);
  if(speed<500) speed=500;
  boxTimeout=setTimeout(()=>{
    box.style.display="none";
    showBox();
  },speed);
}

// ===== Box Click =====
box.onclick=function(){
  if(!gameRunning) return;

  score++;
  scoreEl.innerText="النقاط: "+score;
  scoreEl.classList.add("glow");
  clearTimeout(comboTimer);
  comboTimer=setTimeout(()=>{ scoreEl.classList.remove("glow"); },200);

  // Neon Combo
  comboCount++;
  if(comboCount>=5){ box.style.transform="scale(1.2)"; setTimeout(()=>{box.style.transform="scale(1)";},150); comboCount=0;}

  box.style.display="none";
  clearTimeout(boxTimeout);
  showBox();
}

// ===== End Game =====
function endGame(){
  gameRunning=false;
  clearInterval(timerInterval);
  clearTimeout(boxTimeout);
  clearTimeout(comboTimer);
  box.style.display="none";

  let earnedXP=score*5;
  let earnedCoins=score*2;
  xp+=earnedXP; coins+=earnedCoins;
  while(xp>=level*100){ xp-=level*100; level++; }

  localStorage.setItem("level",level);
  localStorage.setItem("xp",xp);
  localStorage.setItem("coins",coins);

  updateUI();
}

// ===== Random Neon Color =====
function getRandomColor(){
  let letters="0123456789ABCDEF";
  let color="#";
  for(let i=0;i<6;i++){ color+=letters[Math.floor(Math.random()*16)]; }
  return color;
}

updateUI();
</script>
</body>
</html>
