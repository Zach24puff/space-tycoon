<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Space Station Tycoon 2D</title>
<style>
  body, html {
    margin: 0; padding: 0; overflow: hidden; background: #0b0f2a;
    font-family: Arial, sans-serif;
  }
  #ui {
    position: absolute; top: 10px; left: 10px;
    background: rgba(0,0,0,0.75);
    padding: 15px; border-radius: 10px;
    box-shadow: 0 0 10px #00ffcc;
    color: white; width: 220px; z-index: 10;
  }
  canvas {
    display: block;
    background: #1a1a2e;
  }
</style>
</head>
<body>

<div id="ui">
  <h1>🚀 Space Station Tycoon 2D</h1>
  <div>💰 Credits: <span id="credits">200</span></div>
  <div>🔬 Labor: <span id="labor">0</span></div>
  <div>🏪 Handel: <span id="handel">0</span></div>
  <div>🔋 Solar: <span id="solar">0</span></div>
  <div>🪐 Plattformen: <span id="platform">1</span></div>
</div>

<canvas id="gameCanvas" width="800" height="600"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

let credits = 200;
let labor = 0;
let handel = 0;
let solar = 0;
let platform = 1;

let player = { x: 400, y: 300, radius: 15, speed: 4 };

const buyZones = [];
const otherBuyZones = [];

const priceSolar = 100;
const priceLabor = 100;
const priceHandel = 250;
const priceHouseObj = 50;

const priceSolarInc = 100;
const priceHouseObjInc = 50;

let currentPriceSolar = priceSolar;
let currentPriceLabor = priceLabor;
let currentPriceHandel = priceHandel;
let currentPriceHouseObj = priceHouseObj;

// Erstelle ein paar Kaufzonen
function addBuyZone(x, y, radius, type) {
  buyZones.push({x, y, radius, type, bought: false});
}
// Häuser kaufen (extra Array)
function addHouseBuyZone(x, y, radius) {
  otherBuyZones.push({x, y, radius, type: 'houseObj', bought: false});
}

// Beispiel-Zonen (nicht zu viele, für Demo)
addBuyZone(200, 200, 30, 'solar');
addBuyZone(600, 150, 30, 'labor');
addBuyZone(500, 450, 30, 'handel');

addHouseBuyZone(350, 350, 35);
addHouseBuyZone(450, 300, 35);

// Steuerung mit Pfeiltasten oder WASD
const keys = {};
window.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
window.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

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
function zoneLabel(type) {
  switch(type) {
    case 'solar': return 'Solar';
    case 'labor': return 'Labor';
    case 'handel': return 'Handel';
    case 'houseObj': return 'Haus';
    default: return '';
  }
}

function updateUI() {
  document.getElementById('credits').innerText = credits;
  document.getElementById('labor').innerText = labor;
  document.getElementById('handel').innerText = handel;
  document.getElementById('solar').innerText = solar;
  document.getElementById('platform').innerText = platform;
}

// Bewegung Spieler
function movePlayer() {
  if(keys['arrowup'] || keys['w']) player.y -= player.speed;
  if(keys['arrowdown'] || keys['s']) player.y += player.speed;
  if(keys['arrowleft'] || keys['a']) player.x -= player.speed;
  if(keys['arrowright'] || keys['d']) player.x += player.speed;

  // Begrenzung im Canvas
  if(player.x < player.radius) player.x = player.radius;
  if(player.x > canvas.width - player.radius) player.x = canvas.width - player.radius;
  if(player.y < player.radius) player.y = player.radius;
  if(player.y > canvas.height - player.radius) player.y = canvas.height - player.radius;
}

// Prüfen, ob Spieler in Kaufzone ist
function checkBuy() {
  function buyZone(z) {
    if(z.bought) return;
    let dx = player.x - z.x;
    let dy = player.y - z.y;
    let dist = Math.sqrt(dx*dx + dy*dy);
    if(dist < player.radius + z.radius) {
      // Kaufen je nach Typ
      if(z.type === 'houseObj') {
        if(credits >= currentPriceHouseObj) {
          credits -= currentPriceHouseObj;
          z.bought = true;
          houseBoughtSound();
          currentPriceHouseObj += priceHouseObjInc;
        }
      } else if(z.type === 'solar') {
        if(credits >= currentPriceSolar) {
          credits -= currentPriceSolar;
          solar++;
          z.bought = true;
          currentPriceSolar += priceSolarInc;
          buySound();
        }
      } else if(z.type === 'labor') {
        if(credits >= priceLabor) {
          credits -= priceLabor;
          labor++;
          z.bought = true;
          buySound();
        }
      } else if(z.type === 'handel') {
        if(credits >= priceHandel) {
          credits -= priceHandel;
          handel++;
          z.bought = true;
          buySound();
        }
      }
      updateUI();
    }
  }
  buyZones.forEach(buyZone);
  otherBuyZones.forEach(buyZone);
}

function buySound() {
  const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
  audio.volume = 0.3;
  audio.play();
}
function houseBoughtSound() {
  const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/21/audio_0c4d4f8da6.mp3?filename=achievement-6069.mp3");
  audio.volume = 0.5;
  audio.play();
}

// Haupt-Loop
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Hintergrund
  ctx.fillStyle = '#1a1a2e';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Plattformen anzeigen
  ctx.fillStyle = '#444';
  ctx.fillRect(100, 100, 600, 400);

  // Kaufzonen anzeigen mit Farbe + Beschriftung
  buyZones.forEach(z => {
    ctx.fillStyle = zoneColor(z.type, z.bought);
    ctx.beginPath();
    ctx.arc(z.x, z.y, z.radius, 0, 2*Math.PI);
    ctx.fill();

    // Text-Beschriftung
    ctx.fillStyle = 'white';
    ctx.font = '14px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(zoneLabel(z.type), z.x, z.y - z.radius - 8);
  });
  otherBuyZones.forEach(z => {
    ctx.fillStyle = zoneColor(z.type, z.bought);
    ctx.beginPath();
    ctx.arc(z.x, z.y, z.radius, 0, 2*Math.PI);
    ctx.fill();

    // Text-Beschriftung
    ctx.fillStyle = 'white';
    ctx.font = '14px Arial';
    ctx.textAlign = 'center';
    ctx.fillText(zoneLabel(z.type), z.x, z.y - z.radius - 8);
  });

  // Spieler zeichnen
  ctx.fillStyle = '#ff5500';
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.radius, 0, 2*Math.PI);
  ctx.fill();

  requestAnimationFrame(draw);
}

// Einnahmen pro Sekunde
setInterval(() => {
  credits += solar * 100 + labor * 50 + handel * 150;
  updateUI();
}, 1000);

updateUI();
draw();
setInterval(() => {
  movePlayer();
  checkBuy();
}, 20);
</script>

</body>
</html>
