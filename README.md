<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon.io</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
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
      top: 10px;
      left: 10px;
      background: rgba(0, 0, 0, 0.75);
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
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Game state
    let credits = 0;
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

    function update() {
      if (keys['a']) player.dx = -2;
      else if (keys['d']) player.dx = 2;
      else player.dx = 0;

      if (keys['w'] && player.onGround) {
        player.dy = -8;
        player.onGround = false;
      }

      player.x += player.dx;
      player.dy += 0.4;
      player.y += player.dy;

      if (player.y + player.height > platform.y) {
        player.y = platform.y - player.height;
        player.dy = 0;
        player.onGround = true;
      }

      ctx.clearRect(0, 0, width, height);

      // Plattform
      ctx.fillStyle = platform.color;
      ctx.fillRect(platform.x - player.x + width / 2 - player.width / 2, platform.y, platform.width, platform.height);

      // Module
      for (let i = 0; i < modules; i++) {
        ctx.fillStyle = "#00ffcc";
        ctx.fillRect(width / 2 - player.width / 2 + i * 60, platform.y - 40, 40, 40);
      }

      // Spieler
      ctx.fillStyle = player.color;
      ctx.fillRect(width / 2 - player.width / 2, player.y, player.width, player.height);

      requestAnimationFrame(update);
    }

    update();

    function addCredits() {
      let base = 10;
      if (modules > 1) {
        base = 10 + (modules - 1) * 50;
      }
      credits += base;
      document.getElementById('credits').innerText = credits;
    }
    setInterval(addCredits, 1000);

    function buyModule() {
      if (credits >= 100) {
        credits -= 100;
        modules++;
        document.getElementById('credits').innerText = credits;
        document.getElementById('moduleCount').innerText = modules;
      }
    }
  </script>
</body>
</html>
