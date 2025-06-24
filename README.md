<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Space Station Tycoon 2D</title>
<style>
  body, html {
    margin: 0; padding: 0; overflow: hidden; background: #87ceeb; font-family: Arial, sans-serif;
  }
  #ui {
    position: absolute;
    top: 10px; left: 10px;
    background: rgba(0,0,0,0.75);
    color: white;
    padding: 15px;
    border-radius: 10px;
    box-shadow: 0 0 10px #00ffcc;
    font-weight: bold;
    font-size: 1.1em;
    width: 220px;
    z-index: 10;
  }
  canvas {
    display: block;
    background: #444;
  }
  /* Touch controls */
  #touchControls {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 10px;
    z-index: 11;
  }
  .btn {
    width: 60px;
    height: 60px;
    background: rgba(0,255,204,0.7);
    border-radius: 50%;
    text-align: center;
    line-height: 60px;
    font-size: 30px;
    user-select: none;
    touch-action: none;
  }
</style>
</head>
<body>

<div id="ui">
  <h1>🚀 Space Station Tycoon 2D</h1>
  <div>💰 Credits: <span id="credits">200</span></div>
  <div>🏗️ Objekte gekauft: <span id="objects">0</span></div>
  <div>🔋 Energie (Solarpanels): <span id="solar">0</span></div>
  <div>🏪 Handelsstationen: <span id="handel">0</span></div>
</div>

<canvas id="game" width="800" height="600"></canvas>

<div id="touchControls">
  <div class="btn" id="btnLeft">◀</div>
  <div class="btn" id="btnUp">▲</div>
  <div class="btn" id="btnDown">▼</div>
  <div class="btn" id="btnRight">▶</div>
</div>

<script>
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  // Spielfeld-Größe 1000x1000 (Skaliert im Canvas auf 800x600)
  const worldSize = 1000;
  const scaleX = canvas.width / worldSize;
  const scaleY = canvas.height / worldSize;

  // Spieler
  const player = {
    x: worldSize / 2,
    y: worldSize / 2,
    size: 20,
    color: '#ff5500',
    speed: 4
  };

  // Kaufkreise
  const buyTriggers = [];
  const buyCount = 1500;

  // Preise & State
  let credits = 200;
  let solar = 0;
  let handel = 0;
  let objectsBought = 0;

  let priceSolar = 500;
  let priceHandel = 1000;
  let priceHouseObj = 300;
  const priceSolarInc = 300;
  const priceHandelInc = 500;
  const priceHouseObjInc = 200;

  // Kaufobjekte erstellen
  function createBuyTriggers() {
    for(let i=0; i<buyCount; i++) {
      const x = Math.random() * worldSize;
      const y = Math.random() * worldSize;
      const types = ['solar', 'handel', 'houseObj'];
      const type = types[Math.floor(Math.random()*types.length)];
      buyTriggers.push({x, y, type, bought:false});
    }
  }
  createBuyTriggers();

  // Sound
  function playSound() {
    const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
    audio.volume = 0.3;
    audio.play();
  }

  // UI update
  function updateUI() {
    document.getElementById('credits').innerText = Math.floor(credits);
    document.getElementById('solar').innerText = solar;
    document.getElementById('handel').innerText = handel;
    document.getElementById('objects').innerText = objectsBought;
  }

  // Bewegung
  const keys = {};
  document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
  document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

  // Touch Buttons
  const btnLeft = document.getElementById('btnLeft');
  const btnRight = document.getElementById('btnRight');
  const btnUp = document.getElementById('btnUp');
  const btnDown = document.getElementById('btnDown');

  const touchState = {left:false, right:false, up:false, down:false};

  function touchDownHandler(dir) {
    touchState[dir] = true;
  }
  function touchUpHandler(dir) {
    touchState[dir] = false;
  }

  btnLeft.addEventListener('touchstart', e => { e.preventDefault(); touchDownHandler('left'); });
  btnLeft.addEventListener('touchend', e => { e.preventDefault(); touchUpHandler('left'); });
  btnRight.addEventListener('touchstart', e => { e.preventDefault(); touchDownHandler('right'); });
  btnRight.addEventListener('touchend', e => { e.preventDefault(); touchUpHandler('right'); });
  btnUp.addEventListener('touchstart', e => { e.preventDefault(); touchDownHandler('up'); });
  btnUp.addEventListener('touchend', e => { e.preventDefault(); touchUpHandler('up'); });
  btnDown.addEventListener('touchstart', e => { e.preventDefault(); touchDownHandler('down'); });
  btnDown.addEventListener('touchend', e => { e.preventDefault(); touchUpHandler('down'); });

  // Spieler bewegen
  function movePlayer() {
    if(keys['w'] || touchState.up) player.y -= player.speed;
    if(keys['s'] || touchState.down) player.y += player.speed;
    if(keys['a'] || touchState.left) player.x -= player.speed;
    if(keys['d'] || touchState.right) player.x += player.speed;

    // Begrenzung
    player.x = Math.max(0, Math.min(worldSize, player.x));
    player.y = Math.max(0, Math.min(worldSize, player.y));
  }

  // Kauf-Check
  function checkBuy() {
    for(let bt of buyTriggers) {
      if(bt.bought) continue;
      const dx = player.x - bt.x;
      const dy = player.y - bt.y;
      const dist = Math.sqrt(dx*dx + dy*dy);

      if(dist < player.size) {
        let cost = 0;
        if(bt.type === 'solar') cost = priceSolar;
        else if(bt.type === 'handel') cost = priceHandel;
        else cost = priceHouseObj;

        if(credits >= cost) {
          credits -= cost;
          bt.bought = true;
          objectsBought++;

          if(bt.type === 'solar') {
            solar++;
            priceSolar += priceSolarInc;
          } else if(bt.type === 'handel') {
            handel++;
            priceHandel += priceHandelInc;
          } else {
            priceHouseObj += priceHouseObjInc;
          }
          playSound();
          updateUI();
        }
      }
    }
  }

  // Einkommen pro Sekunde
  setInterval(() => {
    credits += solar * 100 + handel * 150;
    updateUI();
  }, 1000);

  // Zeichnen
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Zeichne Kaufkreise
    for(let bt of buyTriggers) {
      if(bt.bought) continue;
      let color;
      if(bt.type === 'solar') color = 'yellow';
      else if(bt.type === 'handel') color = 'magenta';
      else color = 'cyan';

      ctx.beginPath();
      ctx.arc(bt.x * scaleX, bt.y * scaleY, 10, 0, Math.PI * 2);
      ctx.fillStyle = color;
      ctx.globalAlpha = 0.5;
      ctx.fill();
      ctx.globalAlpha = 1;

      // Beschriftung mit Preis
      ctx.fillStyle = 'black';
      ctx.font = 'bold 10px Arial';
      let price = 0;
      if(bt.type === 'solar') price = priceSolar;
      else if(bt.type === 'handel') price = priceHandel;
      else price = priceHouseObj;
      ctx.fillText(price + 'c', bt.x * scaleX - 12, bt.y * scaleY + 4);
    }

    // Spieler zeichnen
    ctx.fillStyle = player.color;
    ctx.fillRect(
      player.x * scaleX - player.size/2,
      player.y * scaleY - player.size/2,
      player.size,
      player.size
    );
  }

  // Game Loop
  function loop() {
    movePlayer();
    checkBuy();
    draw();
    requestAnimationFrame(loop);
  }

  updateUI();
  loop();

  // Resize Canvas (optional, fixed size hier)
  // window.addEventListener('resize', () => { ... });
</script>

</body>
</html>
