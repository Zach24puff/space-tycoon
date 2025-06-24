<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon.io</title>
  <style>
    html, body {
      margin: 0; padding: 0;
      overflow: hidden;
      font-family: Arial, sans-serif;
      background: #000;
      color: white;
      user-select: none;
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
    }
    .btn:hover {
      background: #00ffcc;
      color: #000;
    }
    .stats {
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas"></canvas>

  <div class="ui">
    <h1>🚀 Space Station Tycoon</h1>
    <div class="stats">
      💰 Credits: <span id="credits">0</span>
    </div>
    <button class="btn" onclick="buyModule()">🛰️ Modul kaufen (100 Credits)</button>
    <div>Module: <span id="moduleCount">0</span></div>
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
      buyButton.x = width - 60; // Update Button Position on resize
      buyButton.y = height - 60;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Game state
    let credits = 200; // Startkapital 200 Credits
    let modules = 0;

    const player = {
      x: 100,
      y: 300,
      width: 40,
      height: 60,
      color: '#00ffcc',
      dx: 0,
      dy: 0,
      onGround: true,
    };

    const platform = {
      x: 0,
      y: 400,
      width: 10000,
      height: 50,
      color: '#444',
    };

    // WASD controls
    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    // Kauf-Kreis Button
    const buyButton = {
      x: window.innerWidth - 60,
      y: window.innerHeight - 60,
      radius: 30,
      color: '#00ffcc',
      hoverColor: '#00aaaa',
      isHover: false,
    };

    // Mausposition tracken
    const mouse = { x: 0, y: 0 };
    window.addEventListener('mousemove', e => {
      mouse.x = e.clientX;
      mouse.y = e.clientY;
      const dx = mouse.x - buyButton.x;
      const dy = mouse.y - buyButton.y;
      buyButton.isHover = (dx*dx + dy*dy) <= (buyButton.radius * buyButton.radius);
      canvas.style.cursor = buyButton.isHover ? 'pointer' : 'default';
    });

    // Klick auf Canvas für Kaufbutton
    canvas.addEventListener('click', () => {
      if (buyButton.isHover) {
        buyModule();
      }
    });

    // Funktion Module kaufen
    function buyModule() {
      if (credits >= 100) {
        credits -= 100;
        modules++;
        updateUI();
      }
    }

    // Geld pro Sekunde dazu
    function addCredits() {
      let base = 10;
      if (modules > 1) {
        base = 10 + (modules - 1) * 50;
      }
      credits += base;
      updateUI();
    }
    setInterval(addCredits, 1000);

    // UI aktualisieren
    function updateUI() {
      document.getElementById('credits').innerText = credits;
      document.getElementById('moduleCount').innerText = modules;
    }

    // Spiel Logik & Zeichnung
    function update() {
      // Bewegung Spieler
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
      if (player.y + player.height > platform.y) {
        player.y = platform.y - player.height;
        player.dy = 0;
        player.onGround = true;
      }

      // Hintergrund löschen
      ctx.clearRect(0, 0, width, height);

      // Plattform zeichnen
      ctx.fillStyle = platform.color;
      ctx.fillRect(platform.x - player.x + width / 2 - player.width / 2, platform.y, platform.width, platform.height);

      // Module als kleine Quadrate
      for (let i = 0; i < modules; i++) {
        ctx.fillStyle = "#00ffcc";
        ctx.fillRect(width / 2 - player.width / 2 + i * 60, platform.y - 40, 40, 40);
      }

      // Spieler zeichnen
      ctx.fillStyle = player.color;
      ctx.fillRect(width / 2 - player.width / 2, player.y, player.width, player.height);

      // Kauf-Button zeichnen
      drawBuyButton();

      requestAnimationFrame(update);
    }

    // Kauf-Button zeichnen (Kreis mit Plus)
    function drawBuyButton() {
      ctx.save();
      ctx.beginPath();
      ctx.fillStyle = buyButton.isHover ? buyButton.hoverColor : buyButton.color;
      ctx.shadowColor = buyButton.color;
      ctx.shadowBlur = 15;
      ctx.arc(buyButton.x, buyButton.y, buyButton.radius, 0, Math.PI * 2);
      ctx.fill();

      // Plus Zeichen
      ctx.strokeStyle = 'black';
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.moveTo(buyButton.x - 10, buyButton.y);
      ctx.lineTo(buyButton.x + 10, buyButton.y);
      ctx.moveTo(buyButton.x, buyButton.y - 10);
      ctx.lineTo(buyButton.x, buyButton.y + 10);
      ctx.stroke();
      ctx.restore();
    }

    updateUI();
    update();
  </script>
</body>
</html>
