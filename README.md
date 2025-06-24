<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon.io</title>
  <style>
    html, body {
      margin: 0; padding: 0; overflow: hidden;
      font-family: Arial, sans-serif;
      background: #000;
      color: white;
    }
    canvas {
      display: block;
      background: linear-gradient(to bottom, #000010, #001130);
    }
    .ui {
      position: absolute;
      top: 10px; left: 10px;
      background: rgba(0,0,0,0.75);
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ffcc;
      z-index: 10;
      width: 280px;
      user-select: none;
    }
    .ui h1 {
      color: #00ffcc;
      font-size: 20px;
      margin: 0 0 10px 0;
    }
    .btn {
      display: block;
      margin: 5px 0;
      padding: 8px;
      background: #111;
      border: 1px solid #00ffcc;
      border-radius: 6px;
      color: white;
      cursor: pointer;
      transition: 0.3s;
      user-select: none;
    }
    .btn:hover {
      background: #00ffcc;
      color: #000;
    }
    .stats {
      margin-bottom: 10px;
      font-size: 14px;
    }
    .moduleCount {
      font-weight: bold;
      color: #00ffcc;
    }
    .upgrade {
      font-size: 12px;
      margin-left: 8px;
      padding: 3px 6px;
      border: 1px solid #00ffcc;
      border-radius: 4px;
      background: transparent;
      cursor: pointer;
      color: #00ffcc;
      user-select: none;
      transition: 0.3s;
    }
    .upgrade:hover {
      background: #00ffcc;
      color: #000;
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas"></canvas>

  <div class="ui" role="region" aria-label="Spielsteuerung und Ressourcen">
    <h1>🚀 Space Station Tycoon</h1>
    <div class="stats" aria-live="polite" aria-atomic="true">
      💰 Credits: <span id="credits">200</span><br />
      ⚡ Energie: <span id="energy">0</span><br />
      🔬 Forschung: <span id="research">0</span>
    </div>

    <button class="btn" onclick="buyModule('solar')">🔋 Solarpanel kaufen (100 Credits)</button>
    <button class="btn" onclick="buyModule('labor')">🧪 Labor kaufen (100 Credits)</button>
    <button class="btn" onclick="buyModule('handel')">🏪 Handelsstation kaufen (250 Energie)</button>

    <div style="margin-top:10px;">
      Solarpanels: <span class="moduleCount" id="count-solar">0</span>
      <button class="upgrade" onclick="upgradeModule('solar')" title="Upgrade +25% Produktion (100 Forschung)">Upgrade</button><br />
      Labore: <span class="moduleCount" id="count-labor">0</span>
      <button class="upgrade" onclick="upgradeModule('labor')" title="Upgrade +25% Produktion (100 Forschung)">Upgrade</button><br />
      Handelsstationen: <span class="moduleCount" id="count-handel">0</span>
      <button class="upgrade" onclick="upgradeModule('handel')" title="Upgrade +25% Produktion (100 Forschung)">Upgrade</button>
    </div>

    <div style="margin-top:15px;">
      <button class="btn" onclick="saveGame()">💾 Spiel speichern</button>
      <button class="btn" onclick="loadGame()">📂 Spiel laden</button>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    let width, height;

    function resizeCanvas() {
      width = window.innerWidth;
      height = window.innerHeight;
      canvas.width = width;
      canvas.height = height;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Ressourcen
    let credits = 200;
    let energy = 0;
    let research = 0;

    // Module zählen
    let solarPanels = 0;
    let labore = 0;
    let handelsstationen = 0;

    // Upgrade-Level (1 = 100%)
    let solarUpgrade = 1;
    let laborUpgrade = 1;
    let handelUpgrade = 1;

    // Spieler (für Plattform)
    const player = {
      x: 0,
      y: 0,
      width: 40,
      height: 60,
      color: '#00ffcc',
      dx: 0,
      dy: 0,
      onGround: false,
    };

    const platform = {
      y: 400,
      height: 50,
      color: '#444',
    };

    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    function resetPlayer() {
      player.x = 0;
      player.y = platform.y - player.height;
      player.dx = 0;
      player.dy = 0;
      player.onGround = true;
    }

    function update() {
      // Steuerung
      if (keys['a']) player.dx = -2;
      else if (keys['d']) player.dx = 2;
      else player.dx = 0;

      if (keys['w'] && player.onGround) {
        player.dy = -8;
        player.onGround = false;
      }

      player.x += player.dx;
      player.dy += 0.4; // Gravitation
      player.y += player.dy;

      // Plattform-Kollision
      if (player.y + player.height >= platform.y) {
        player.y = platform.y - player.height;
        player.dy = 0;
        player.onGround = true;
      }

      ctx.clearRect(0, 0, width, height);
      const screenX = width / 2 - player.width / 2;

      // Plattform
      ctx.fillStyle = platform.color;
      ctx.fillRect(0, platform.y, width, platform.height);

      // Module als Rechtecke oberhalb Plattform, horizontal zentriert
      function drawModules(count, yOffset, color) {
        ctx.fillStyle = color;
        for (let i = 0; i < count; i++) {
          const mX = screenX + (i - Math.floor(count / 2)) * 60;
          ctx.fillRect(mX, platform.y - yOffset, 40, 40);
        }
      }
      drawModules(solarPanels, 40, '#00ffcc');
      drawModules(labore, 90, '#ffcc00');
      drawModules(handelsstationen, 140, '#cc00ff');

      // Spieler zeichnen
      ctx.fillStyle = player.color;
      ctx.fillRect(screenX, player.y, player.width, player.height);

      requestAnimationFrame(update);
    }

    function addResources() {
      energy += solarPanels * 5 * solarUpgrade;
      research += labore * 5 * laborUpgrade;
      credits += handelsstationen * 10 * handelUpgrade;
      updateUI();
    }

    function updateUI() {
      document.getElementById('credits').innerText = Math.floor(credits);
      document.getElementById('energy').innerText = Math.floor(energy);
      document.getElementById('research').innerText = Math.floor(research);
      document.getElementById('count-solar').innerText = solarPanels;
      document.getElementById('count-labor').innerText = labore;
      document.getElementById('count-handel').innerText = handelsstationen;
    }

    // Soundeffekt beim Kauf (einfacher Klickton)
    const clickSound = new Audio();
    clickSound.src = "data:audio/wav;base64,UklGRiQAAABXQVZFZm10IBAAAAABAAEAQB8AAIA+AAACABAAZGF0YQAAAAA="; 
    // minimaler leerer Sound, kann durch eigenen Sound ersetzt werden

    function buyModule(type) {
      if(type === 'solar') {
        if(credits >= 100) {
          credits -= 100;
          solarPanels++;
          clickSound.play();
          updateUI();
        } else alert('Nicht genug Credits für Solarpanel!');
      }
      else if(type === 'labor') {
        if(credits >= 100) {
          credits -= 100;
          labore++;
          clickSound.play();
          updateUI();
        } else alert('Nicht genug Credits für Labor!');
      }
      else if(type === 'handel') {
        if(energy >= 250) {
          energy -= 250;
          handelsstationen++;
          clickSound.play();
          updateUI();
        } else alert('Nicht genug Energie für Handelsstation!');
      }
    }

    function upgradeModule(type) {
      const upgradeCost = 100;
      if(research < upgradeCost) {
        alert('Nicht genug Forschung für Upgrade!');
        return;
      }
      research -= upgradeCost;
      if(type === 'solar') {
        solarUpgrade *= 1.25;
      } else if(type === 'labor') {
        laborUpgrade *= 1.25;
      } else if(type === 'handel') {
        handelUpgrade *= 1.25;
      }
      updateUI();
    }

    // Speicher-Funktionen
    function saveGame() {
      const saveData = {
        credits, energy, research,
        solarPanels, labore, handelsstationen,
        solarUpgrade, laborUpgrade, handelUpgrade,
        playerX: player.x,
        playerY: player.y
      };
      localStorage.setItem('spaceTycoonSave', JSON.stringify(saveData));
      alert('Spiel gespeichert!');
    }

    function loadGame() {
      const saveStr = localStorage.getItem('spaceTycoonSave');
      if(!saveStr) {
        alert('Kein Spielstand gefunden!');
        return;
      }
      const saveData = JSON.parse(saveStr);
      credits = saveData.credits || 0;
      energy = saveData.energy || 0;
      research = saveData.research || 0;
      solarPanels = saveData.solarPanels || 0;
      labore = saveData.labore || 0;
      handelsstationen = saveData.handelsstationen || 0;
      solarUpgrade = saveData.solarUpgrade || 1;
      laborUpgrade = saveData.laborUpgrade || 1;
      handelUpgrade = saveData.handelUpgrade || 1;
      player.x = saveData.playerX || 0;
      player.y = saveData.playerY || platform.y - player.height;
      updateUI();
      alert('Spiel geladen!');
    }

    resetPlayer();
    update();
    updateUI();
    setInterval(addResources, 1000);
  </script>
</body>
</html>
