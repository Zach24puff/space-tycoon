<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Space Station Tycoon 2D Top-Down</title>
<style>
  body, html {
    margin: 0; padding: 0; overflow: hidden; background: #87ceeb;
    font-family: Arial, sans-serif;
  }
  #ui {
    position: absolute;
    top: 10px; left: 10px;
    background: rgba(0,0,0,0.75);
    padding: 15px;
    border-radius: 10px;
    box-shadow: 0 0 10px #00ffcc;
    color: white;
    width: 220px;
    z-index: 10;
  }
  canvas {
    display: block;
    background-color: #888888;
    margin: 0;
  }
</style>
</head>
<body>

<div id="ui">
  <h1>🚀 Space Station Tycoon</h1>
  <div>💰 Credits: <span id="credits">200</span></div>
  <div>🔬 Labor: <span id="labor">0</span></div>
  <div>🏪 Handelsstation: <span id="handel">0</span></div>
  <div>🔋 Solarpanels: <span id="solar">0</span></div>
  <div>🪐 Plattformen: <span id="platform">1</span> (100x100 gehört dir)</div>
</div>

<canvas id="gameCanvas" width="800" height="600"></canvas>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  // Spiel-Variablen
  let credits = 200;
  let labor = 0;
  let handel = 0;
  let solar = 0;
  let platformCount = 1;

  const priceSolar = 100;
  const priceLabor = 100;
  const priceHandel = 250;
  const priceHouseObj = 50;

  // Spielfigur
  const player = {
    x: canvas.width/2,
    y: canvas.height/2,
    size: 20,
    speed: 3
  };

  // Häuser (als Rechtecke)
  const houses = [
    {x: 100, y: 100, width: 40, height: 40, bought: false},
    {x: 300, y: 200, width: 40, height: 40, bought: false},
    {x: 600, y: 400, width: 40, height: 40, bought: false},
  ];

  // Kaufzonen rund um Häuser (Radius 30px)
  const buyZones = houses.map(h => ({
    x: h.x + h.width/2,
    y: h.y + h.height/2,
    radius: 30,
    type: 'houseObj',
    bought: false,
    house: h
  }));

  // Weitere Kaufzonen (Solar, Labor, Handel) zufällig verteilt
  const otherBuyZones = [];
  const types = ['solar', 'labor', 'handel'];
  for(let i=0; i<50; i++) {
    otherBuyZones.push({
      x: Math.random()*(canvas.width-60)+30,
      y: Math.random()*(canvas.height-60)+30,
      radius: 20,
      type: types[Math.floor(Math.random()*types.length)],
      bought: false
    });
  }

  // Bewegungstasten
  const keys = {};
  window.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
  window.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

  function updatePlayerPosition() {
    if(keys['w'] || keys['arrowup']) player.y -= player.speed;
    if(keys['s'] || keys['arrowdown']) player.y += player.speed;
    if(keys['a'] || keys['arrowleft']) player.x -= player.speed;
    if(keys['d'] || keys['arrowright']) player.x += player.speed;

    // Begrenzung im Canvas
    player.x = Math.max(player.size/2, Math.min(canvas.width - player.size/2, player.x));
    player.y = Math.max(player.size/2, Math.min(canvas.height - player.size/2, player.y));
  }

  // Prüfen ob Spieler in Kaufzone ist
  function checkBuyZones() {
    const zones = buyZones.concat(otherBuyZones);

    zones.forEach(zone => {
      if(zone.bought) return;
      const dx = player.x - zone.x;
      const dy = player.y - zone.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      if(dist < zone.radius) {
        // Prüfen ob genug Credits
        if(zone.type === 'houseObj') {
          if(credits >= priceHouseObj) {
            credits -= priceHouseObj;
            zone.bought = true;
            zone.house.bought = true;
            playSound();
          }
        } else if(zone.type === 'solar') {
          if(credits >= priceSolar) {
            credits -= priceSolar;
            solar++;
            zone.bought = true;
            playSound();
          }
        } else if(zone.type === 'labor') {
          if(credits >= priceLabor) {
            credits -= priceLabor;
            labor++;
            zone.bought = true;
            playSound();
          }
        } else if(zone.type === 'handel') {
          if(credits >= priceHandel) {
            credits -= priceHandel;
            handel++;
            zone.bought = true;
            playSound();
          }
        }
      }
    });
  }

  function playSound() {
    const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
    audio.volume = 0.3;
    audio.play();
  }

  // UI aktualisieren
  function updateUI() {
    document.getElementById('credits').innerText = Math.floor(credits);
    document.getElementById('labor').innerText = labor;
    document.getElementById('handel').innerText = handel;
    document.getElementById('solar').innerText = solar;
    document.getElementById('platform').innerText = platformCount;
  }

  // Einnahmen pro Sekunde
  setInterval(() => {
    credits += solar * 10 + labor * 5 + handel * 15;
    updateUI();
  }, 1000);

  // Zeichnen
  function draw() {
    // Hintergrund
    ctx.fillStyle = '#888888';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Plattform-Rand (100x100) in der Mitte (für Demo)
    ctx.strokeStyle = '#004040';
    ctx.lineWidth = 3;
    ctx.strokeRect(canvas.width/2 - 50, canvas.height/2 - 50, 100, 100);

    // Häuser
    houses.forEach(h => {
      ctx.fillStyle = h.bought ? '#8B4513' : 'rgba(139,69,19,0.4)';
      ctx.fillRect(h.x, h.y, h.width, h.height);
      // Fenster (vereinfachte weiße Rechtecke)
      if(h.bought) {
        ctx.fillStyle = '#FFFFCC';
        ctx.fillRect(h.x+10, h.y+10, 8, 8);
        ctx.fillRect(h.x+22, h.y+10, 8, 8);
      }
    });

    // Kaufzonen anzeigen (mit Farbe je Typ)
    function zoneColor(type, bought) {
      if(bought) return 'rgba(0,255,204,0.2)';
      switch(type) {
        case 'solar': return 'rgba(0,255,255,0.5)';
        case 'labor': return 'rgba(255,255,0,0.5)';
        case 'handel': return 'rgba(255,0,255,0.5)';
        case 'houseObj': return 'rgba(0,255,204,0.5)';
        default: return 'rgba(255,255,255,0.3)';
      }
    }
    buyZones.forEach(z => {
      ctx.fillStyle = zoneColor(z.type, z.bought);
      ctx.beginPath();
      ctx.arc(z.x, z.y, z.radius, 0, 2*Math.PI);
      ctx.fill();
    });
    otherBuyZones.forEach(z => {
      ctx.fillStyle = zoneColor(z.type, z.bought);
      ctx.beginPath();
      ctx.arc(z.x, z.y, z.radius, 0, 2*Math.PI);
      ctx.fill();
    });

    // Spieler zeichnen
    ctx.fillStyle = '#ff5500';
    ctx.fillRect(player.x - player.size/2, player.y - player.size/2, player.size, player.size);
  }

  // Haupt-Loop
  function loop() {
    updatePlayerPosition();
    checkBuyZones();
    draw();
    updateUI();
    requestAnimationFrame(loop);
  }

  // Start
  loop();
</script>

</body>
</html>
