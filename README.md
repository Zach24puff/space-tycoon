<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<title>Space Station Tycoon 2D</title>
<style>
  body, html {
    margin: 0; padding: 0; overflow: hidden;
    background: #1a1a2e;
    font-family: Arial, sans-serif;
    user-select: none;
  }
  #ui {
    position: fixed;
    top: 10px; left: 10px;
    background: rgba(0,0,0,0.7);
    color: white;
    padding: 12px 18px;
    border-radius: 10px;
    box-shadow: 0 0 10px #00ffcc;
    width: 220px;
    font-size: 1.1em;
    z-index: 10;
  }
</style>
</head>
<body>
  <div id="ui">
    <h1>🚀 Space Station Tycoon 2D</h1>
    <div>💰 Credits: <span id="credits">200</span></div>
    <div>🔬 Labor: <span id="labor">0</span></div>
    <div>🏪 Handelsstation: <span id="handel">0</span></div>
    <div>🔋 Solarpanels: <span id="solar">0</span></div>
    <div>🪐 Plattformen: <span id="platform">1</span></div>
    <div style="margin-top:10px; font-size:0.9em; color:#ccc;">
      Steuerung: WASD oder Pfeiltasten
    </div>
  </div>
  <canvas id="gameCanvas"></canvas>
<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  // Bildschirmgröße anpassen
  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  // Spielvariablen
  let credits = 200;
  let labor = 0;
  let handel = 0;
  let solar = 0;
  let platformCount = 1;

  let priceSolar = 100;
  let priceLabor = 100;
  let priceHandel = 250;
  let priceHouseObj = 50;
  let priceSolarInc = 100;
  let priceHouseObjInc = 50;

  // Spieler
  const player = {
    x: canvas.width/2,
    y: canvas.height/2,
    radius: 15,
    speed: 4,
  };

  // Kaufzonen (Beispiele)
  const buyZones = [
    {x: 150, y: 200, radius: 30, type: 'solar', bought: false},
    {x: 350, y: 150, radius: 30, type: 'labor', bought: false},
    {x: 550, y: 300, radius: 30, type: 'handel', bought: false},
    {x: 450, y: 400, radius: 30, type: 'houseObj', bought: false},
  ];

  // Farben pro Typ
  function zoneColor(type, bought) {
    if(bought) return 'rgba(100,100,100,0.6)';
    switch(type) {
      case 'solar': return 'rgba(0,255,255,0.7)';
      case 'labor': return 'rgba(255,255,0,0.7)';
      case 'handel': return 'rgba(255,0,255,0.7)';
      case 'houseObj': return 'rgba(139,69,19,0.7)';
      default: return 'rgba(0,255,204,0.6)';
    }
  }

  // Bezeichnungen pro Typ
  function zoneLabel(type) {
    switch(type) {
      case 'solar': return 'Solarpanel';
      case 'labor': return 'Labor';
      case 'handel': return 'Handelsstation';
      case 'houseObj': return 'Hausobjekt';
      default: return '';
    }
  }

  // Preislabels
  function priceLabel(type) {
    switch(type) {
      case 'solar': return priceSolar + ' Cr';
      case 'labor': return priceLabor + ' Cr';
      case 'handel': return priceHandel + ' Cr';
      case 'houseObj': return priceHouseObj + ' Cr';
      default: return '';
    }
  }

  // Steuerung
  const keys = {};
  window.addEventListener('keydown', e => {
    keys[e.key.toLowerCase()] = true;
    if(['arrowup','arrowdown','arrowleft','arrowright'].includes(e.key.toLowerCase())) {
      keys[e.key.toLowerCase()] = true;
    }
  });
  window.addEventListener('keyup', e => {
    keys[e.key.toLowerCase()] = false;
    if(['arrowup','arrowdown','arrowleft','arrowright'].includes(e.key.toLowerCase())) {
      keys[e.key.toLowerCase()] = false;
    }
  });

  function movePlayer() {
    if(keys['w'] || keys['arrowup']) player.y -= player.speed;
    if(keys['s'] || keys['arrowdown']) player.y += player.speed;
    if(keys['a'] || keys['arrowleft']) player.x -= player.speed;
    if(keys['d'] || keys['arrowright']) player.x += player.speed;

    // Begrenzung auf Canvas
    if(player.x < player.radius) player.x = player.radius;
    if(player.x > canvas.width - player.radius) player.x = canvas.width - player.radius;
    if(player.y < player.radius) player.y = player.radius;
    if(player.y > canvas.height - player.radius) player.y = canvas.height - player.radius;
  }

  // UI Update
  function updateUI() {
    document.getElementById('credits').innerText = credits;
    document.getElementById('labor').innerText = labor;
    document.getElementById('handel').innerText = handel;
    document.getElementById('solar').innerText = solar;
    document.getElementById('platform').innerText = platformCount;
  }

  // Kaufen prüfen
  function checkBuy() {
    for(let i=0; i<buyZones.length; i++) {
      const z = buyZones[i];
      if(z.bought) continue;
      const dx = player.x - z.x;
      const dy = player.y - z.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      if(dist < player.radius + z.radius) {
        // Kaufen wenn genug Credits
        let price = 0;
        switch(z.type) {
          case 'solar': price = priceSolar; break;
          case 'labor': price = priceLabor; break;
          case 'handel': price = priceHandel; break;
          case 'houseObj': price = priceHouseObj; break;
        }
        if(credits >= price) {
          credits -= price;
          z.bought = true;
          switch(z.type) {
            case 'solar': solar++; priceSolar += priceSolarInc; break;
            case 'labor': labor++; break;
            case 'handel': handel++; break;
            case 'houseObj': platformCount++; priceHouseObj += priceHouseObjInc; break;
          }
          updateUI();
          playSound();
        }
      }
    }
  }

  // Sound
  function playSound() {
    const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
    audio.volume = 0.3;
    audio.play();
  }

  // Einnahmen pro Sekunde
  setInterval(() => {
    credits += solar * 10 + labor * 5 + handel * 15;
    updateUI();
  }, 1000);

  // Zeichnen
  function draw() {
    ctx.clearRect(0,0,canvas.width, canvas.height);

    // Plattform (als graues Rechteck)
    ctx.fillStyle = '#444';
    ctx.fillRect(100, 100, 600, 400);

    // Kaufkreise mit Label & Preis
    buyZones.forEach(z => {
      ctx.fillStyle = zoneColor(z.type, z.bought);
      ctx.beginPath();
      ctx.arc(z.x, z.y, z.radius, 0, Math.PI*2);
      ctx.fill();

      ctx.fillStyle = 'white';
      ctx.font = 'bold 16px Arial';
      ctx.textAlign = 'center';
      ctx.fillText(zoneLabel(z.type), z.x, z.y - z.radius - 10);

      ctx.font = '14px Arial';
      ctx.fillText(priceLabel(z.type), z.x, z.y + z.radius + 16);
    });

    // Spieler
    ctx.fillStyle = '#ff5500';
    ctx.beginPath();
    ctx.arc(player.x, player.y, player.radius, 0, Math.PI*2);
    ctx.fill();

    requestAnimationFrame(draw);
  }

  // Haupt-Loop
  function gameLoop() {
    movePlayer();
    checkBuy();
    requestAnimationFrame(gameLoop);
  }

  updateUI();
  draw();
  gameLoop();
</script>
</body>
</html>
