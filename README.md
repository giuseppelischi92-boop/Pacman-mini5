
<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pac-Man con Chat a Destra</title>
<style>
  body { 
    background:#000; 
    color:white; 
    font-family: Arial; 
    margin:0; 
    display:flex; 
    flex-direction: row; 
  }
  canvas { background:#000; margin:20px; border:2px solid #444; }
  #score { font-size: 20px; margin:20px; }
  #startBtn {
    margin:20px;
    padding:10px 20px;
    font-size:18px;
    background:#28a745;
    border:none;
    color:white;
    border-radius:10px;
    cursor:pointer;
  }
  #startBtn:hover { background:#218838; }

  /* Chat laterale DESTRA */
  .chat {
    width: 280px;
    background: #111;
    border-left: 2px solid #444;
    display:flex;
    flex-direction: column;
  }
  .chat-header {
    background:#222; padding:10px; text-align:center; font-weight:bold;
  }
  .messages {
    flex:1; overflow-y:auto; padding:10px; font-size:14px;
    display:flex; flex-direction:column;
  }
  .message {
    max-width: 75%; margin:5px; padding:8px 12px;
    border-radius:15px; line-height:1.3;
  }
  .user-msg {
    background:#4af; align-self:flex-end; color:#fff;
  }
  .other-msg {
    background:#333; align-self:flex-start; color:#eee;
  }
  .chat-input {
    display:flex; border-top:1px solid #333;
  }
  .chat-input input {
    flex:1; padding:10px; border:none; outline:none; font-size:14px;
  }
  .chat-input button {
    padding:10px; background:#28a745; border:none; color:white; cursor:pointer;
  }
  .chat-input button:hover { background:#218838; }
  .online-counter {
    text-align:center; padding:5px; font-size:13px; background:#1a1a1a;
    border-top:1px solid #333; color:#4af;
  }

  /* Controlli touch */
  .controls {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    opacity: 0.8;
  }
  .row { display: flex; gap: 10px; }
  button.ctrl {
    width: 70px; height: 70px;
    font-size: 24px;
    border-radius: 50%;
    border: none;
    background: rgba(50,50,50,0.7);
    color: white;
    cursor: pointer;
  }
  button.ctrl:active { background: rgba(100,100,100,0.9); }
</style>
</head>
<body>

<!-- Gioco Pac-Man -->
<div>
  <h1 style="margin:20px;">Pac-Man Fluido</h1>
  <button id="startBtn" onclick="startGame()">▶ Start</button>
  <div id="score">Punteggio: 0</div>
  <canvas id="gameCanvas" width="400" height="400"></canvas>
</div>

<!-- Chat laterale -->
<div class="chat">
  <div class="chat-header">💬 Chat</div>
  <div class="messages" id="messages"></div>
  <div class="chat-input">
    <input type="text" id="username" placeholder="Nome">
    <input type="text" id="message" placeholder="Scrivi...">
    <button onclick="sendMessage()">➤</button>
  </div>
  <div class="online-counter" id="onlineCounter">👥 Online: 1</div>
</div>

<!-- Controlli touch -->
<div class="controls">
  <button class="ctrl" onclick="setDirection(0,-1)">▲</button>
  <div class="row">
    <button class="ctrl" onclick="setDirection(-1,0)">◀</button>
    <button class="ctrl" onclick="setDirection(1,0)">▶</button>
  </div>
  <button class="ctrl" onclick="setDirection(0,1)">▼</button>
</div>

<script>
/* ======== CHAT ======== */
let onlineUsers = Math.floor(Math.random()*20)+1; // finto contatore
document.getElementById('onlineCounter').textContent = "👥 Online: "+onlineUsers;

function sendMessage(){
  const user = document.getElementById('username').value || 'Tu';
  const msg = document.getElementById('message').value.trim();
  if(msg==="") return;
  const messagesDiv = document.getElementById('messages');
  const div = document.createElement('div');
  div.classList.add('message');
  if(user==="Tu"){
    div.classList.add('user-msg');
  } else {
    div.classList.add('other-msg');
  }
  div.innerHTML = msg;
  messagesDiv.appendChild(div);
  messagesDiv.scrollTop = messagesDiv.scrollHeight;
  document.getElementById('message').value="";
}

/* ======== PAC-MAN ======== */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const tileSize = 20;
const rows = canvas.height / tileSize;
const cols = canvas.width / tileSize;

let pacman = { x: 1, y: 1, dx: 0, dy: 0 };
let score = 0;
let lives = 3;
let running = false;

let ghosts = [
  {x: 18, y: 1, color:'red'},
  {x: 18, y: 5, color:'pink'}
];

let map = [
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
  [1,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,0,1],
  [1,0,1,1,1,0,1,1,1,1,1,1,1,0,1,1,1,0,0,1],
  [1,0,1,0,0,0,1,0,0,0,0,0,1,0,0,0,1,0,2,1],
  [1,0,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,0,0,1],
  [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
  [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
];

function draw() {
  ctx.clearRect(0,0,canvas.width,canvas.height);
  for(let r=0;r<rows;r++){
    for(let c=0;c<cols;c++){
      if(map[r] && map[r][c] === 1){
        ctx.fillStyle='blue';
        ctx.fillRect(c*tileSize, r*tileSize, tileSize, tileSize);
      } else if(map[r] && map[r][c] === 2){
        ctx.fillStyle='yellow';
        ctx.beginPath();
        ctx.arc(c*tileSize + tileSize/2, r*tileSize + tileSize/2, tileSize/6, 0, Math.PI*2);
        ctx.fill();
      }
    }
  }
  ctx.fillStyle='yellow';
  ctx.beginPath();
  ctx.arc(pacman.x*tileSize + tileSize/2, pacman.y*tileSize + tileSize/2, tileSize/2, 0.25*Math.PI, 1.75*Math.PI);
  ctx.lineTo(pacman.x*tileSize + tileSize/2, pacman.y*tileSize + tileSize/2);
  ctx.fill();

  ghosts.forEach(g => {
    ctx.fillStyle = g.color;
    ctx.beginPath();
    ctx.arc(g.x*tileSize + tileSize/2, g.y*tileSize + tileSize/2, tileSize/2, 0, Math.PI*2);
    ctx.fill();
  });
}

function movePacman() {
  let nx = pacman.x + pacman.dx;
  let ny = pacman.y + pacman.dy;
  if(map[ny] && map[ny][nx] !== 1){
    pacman.x = nx;
    pacman.y = ny;
    if(map[pacman.y][pacman.x] === 2){
      map[pacman.y][pacman.x] = 0;
      score += 10;
      document.getElementById('score').textContent = "Punteggio: " + score;
    }
  }
}

function moveGhosts() {
  ghosts.forEach(g => {
    let dirs = [{dx:0,dy:-1},{dx:0,dy:1},{dx:-1,dy:0},{dx:1,dy:0}];
    let d = dirs[Math.floor(Math.random()*4)];
    let nx = g.x+d.dx, ny = g.y+d.dy;
    if(map[ny] && map[ny][nx] !== 1){ g.x=nx; g.y=ny; }
  });
}

function checkCollision() {
  ghosts.forEach(g => {
    if(g.x===pacman.x && g.y===pacman.y){
      lives--;
      if(lives<=0){
        alert("Hai perso! Punteggio finale: "+score);
        running = false;
      } else {
        pacman.x=1; pacman.y=1;
        ghosts[0].x=18; ghosts[0].y=1;
        ghosts[1].x=18; ghosts[1].y=5;
        alert("Hai perso una vita! Vite rimaste: "+lives);
      }
    }
  });
}

// Controlli
function setDirection(dx,dy){ pacman.dx=dx; pacman.dy=dy; }
document.addEventListener('keydown', e=>{
  if(e.key==='ArrowUp' || e.key==='w'){ setDirection(0,-1); }
  if(e.key==='ArrowDown' || e.key==='s'){ setDirection(0,1); }
  if(e.key==='ArrowLeft' || e.key==='a'){ setDirection(-1,0); }
  if(e.key==='ArrowRight' || e.key==='d'){ setDirection(1,0); }
});

// Loop fluido
function gameLoop(){
  if(!running) return;
  movePacman();
  moveGhosts();
  checkCollision();
  draw();
  let remaining = map.flat().filter(v=>v===2).length;
  if(remaining===0){
    alert("Hai vinto! Punteggio finale: "+score);
    running = false;
  }
  requestAnimationFrame(gameLoop);
}

function startGame(){
  running = true;
  document.getElementById("startBtn").style.display="none";
  gameLoop();
}
</script>

</body>
</html>
