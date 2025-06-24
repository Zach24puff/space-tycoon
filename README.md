<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon.io</title>
  <style>
    /* Vollbild Canvas für Sterne */
    #starfield {
      position: fixed;
      top: 0; left: 0;
      width: 100vw;
      height: 100vh;
      z-index: 0;
      background: #0b0f2a;
    }

    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      color: white;
      overflow-x: hidden;
      position: relative;
      z-index: 10;
    }

    .container {
      background: rgba(0, 0, 0, 0.75);
      padding: 20px;
      max-width: 700px;
      margin: 40px auto;
      border-radius: 15px;
      box-shadow: 0 0 20px #00ffcc;
      position: relative;
      z-index: 20;
    }

    h1 {
      color: #00ffcc;
      text-align: center;
    }

    .stats {
      text-align: center;
      margin-bottom: 20px;
      font-size: 1.2em;
    }

    .btn {
      display: block;
      width: 100%;
      margin: 10px 0;
      padding: 12px;
      background: #111;
      border: 1px solid #00ffcc;
      color: white;
      font-size: 1em;
      border-radius: 8px;
      cursor: pointer;
      transition: 0.3s;
    }

    .btn:hover {
      background: #00ffcc;
      color: #000;
    }

    .module {
      margin-bottom: 20px;
    }

    .modules-visual {
      display: flex;
      justify-content: center;
      gap: 15px;
      margin-top: 15px;
    }

    .module-icon {
      display: flex;
      flex-direction: column;
      align-items: center;
      font-size: 0.9rem;
      user-select: none;
    }

    .module-icon svg {
      width: 40px;
      height: 40px;
      margin-bottom: 5px;
      fill: #00ffcc;
      filter: drop-shadow(0 0 3px #00ffcc);
    }

    .count {
      font-weight: bold;
      color: #00ffcc;
    }
  </style>
</head>
<body>
  <!-- Sternenhimmel Canvas -->
  <canvas id="starfield"></canvas>

  <div class="container" role="main">
    <h1>🚀 Space Station Tycoon.io</h1>
    <div class="stats" aria-live="polite">
      ⚡ Energie: <span id="energie">1000</span> |
      💰 Credits: <span id="credits">55</span> |
      🔬 Forschung: <span id="forschung">1000</span>
    </div>

    <div class="module">
      <button class="btn" onclick="build('solar')">🔋 Solarpanel (10💰)</button>
      <button class="btn" onclick="build('handel')">🏪 Handelsstation (2500⚡)</button>
      <button class="btn" onclick="build('labor')">🧪 Labor (50💰)</button>
    </div>

    <div class="module">
      <button class="btn" onclick="upgrade('solar')">🔋 Upgrade Solar (100🔬)</button>
      <button class="btn" onclick="upgrade('handel')">🏪 Upgrade Handel (100🔬)</button>
      <button class="btn" onclick="upgrade('labor')">🧪 Upgrade Labor (100🔬)</button>
    </div>

    <div class="module">
      <button class="btn" onclick="saveGame()">💾 Speichern</button>
      <button class="btn" onclick="loadGame()">📂 Laden</button>
    </div>

    <div class="modules-visual" aria-label="Gebäude Visualisierung">
      <div class="module-icon" id="visual-solar" aria-live="polite">
        <svg viewBox="0 0 64 64" aria-hidden="true" focusable="false">
          <circle cx="32" cy="32" r="12" stroke="#00ffcc" stroke-width="2" fill="none"/>
          <rect x="26" y="20" width="12" height="24" fill="#00ffcc"/>
        </svg>
        <span>Solarpanel: <span class="count" id="count-solar">0</span></span>
      </div>
      <div class="module-icon" id="visual-handel" aria-live="polite">
        <svg viewBox="0 0 64 64" aria-hidden="true" focusable="false">
          <rect x="16" y="24" width="32" height="20" fill="#00ffcc" stroke="#00ffcc" stroke-width="2" rx="4" ry="4"/>
          <rect x="28" y="16" width="8" height="8" fill="#00ffcc"/>
        </svg>
        <span>Handelsstation: <span class="count" id="count-handel">0</span></span>
      </div>
      <div class="module-icon" id="visual-labor" aria-live="polite">
        <svg viewBox="0 0 64 64" aria-hidden="true" focusable="false">
          <rect x="22" y="18" width="20" height="28" fill="#00ffcc" stroke="#00ffcc" stroke-width="2" rx="3" ry="3"/>
          <circle cx="32" cy="32" r="6" fill="#000" stroke="#00ffcc" stroke-width="2"/>
          <line x1="32" y1="26" x2="32" y2="38" stroke="#00ffcc" stroke-width="2"/>
          <line x1="26" y1="32" x2="38" y2="32" stroke="#00ffcc" stroke-width="2"/>
        </svg>
        <span>Labor: <span class="count" id="count-labor">0</span></span>
      </div>
    </div>
  </div>

  <script>
    // Spielwerte
    let energie = 0, credits = 200, forschung = 0;
    let solarpanels = 0, handelsstationen = 0, labore = 0;
    let solarBoost = 1, handelBoost = 1, laborBoost = 1;

    // Update Funktion: Werte + Visuals
    function update() {
      document.getElementById('energie').innerText = Math.floor(energie);
      document.getElementById('credits').innerText = Math.floor(credits);
      document.getElementById('forschung').innerText = Math.floor(forschung);

      // Visual-Counts updaten
      document.getElementById('count-solar').innerText = solarpanels;
      document.getElementById('count-handel').innerText = handelsstationen;
      document.getElementById('count-labor').innerText = labore;
    }

    // Bauen von Gebäuden
    function build(typ) {
      if (typ === 'solar' && credits >= 100) { credits -= 100; solarpanels++; }
      else if (typ === 'handel' && energie >= 250) { energie -= 250; handelsstationen++; }
      else if (typ === 'labor' && credits >= 500) { credits -= 500; labore++; }
      else return; // nicht genug Ressourcen

      update();
    }

    // Upgrades
    function upgrade(typ) {
      if (forschung < 1000) return;
      forschung -= 1000;
      if (typ === 'solar') solarBoost *= 1.25;
      else if (typ === 'handel') handelBoost *= 1.25;
      else if (typ === 'labor') laborBoost *= 1.25;
      update();
    }

    // Spiel speichern
    function saveGame() {
      const save = {energie, credits, forschung, solarpanels, handelsstationen, labore, solarBoost, handelBoost, laborBoost};
      localStorage.setItem('tycoonSave', JSON.stringify(save));
      alert('Spiel gespeichert!');
    }

    // Spiel laden
    function loadGame() {
      const save = JSON.parse(localStorage.getItem('tycoonSave'));
      if (!save) return alert('Kein Spielstand gefunden.');
      ({energie, credits, forschung, solarpanels, handelsstationen, labore, solarBoost, handelBoost, laborBoost} = save);
      update();
    }

    // Spielwerte jede Sekunde erhöhen
    setInterval(() => {
      energie += solarpanels * 5 * solarBoost;
      credits += handelsstationen * 10 * handelBoost;
      forschung += labore * 5 * laborBoost;
      update();
    }, 1000);

    update();

    // === Sternenhimmel Animation ===
    const canvas = document.getElementById('starfield');
    const ctx = canvas.getContext('2d');
    let width, height;

    // Stern-Klasse
    class Star {
      constructor() {
        this.reset();
      }

      reset() {
        this.x = Math.random() * width;
        this.y = Math.random() * height;
        this.size = Math.random() * 1.5 + 0.5;
        this.speed = this.size * 0.3;
        this.alpha = Math.random() * 0.5 + 0.5;
      }

      update() {
        this.x -= this.speed;
        if (this.x < 0) {
          this.x = width;
          this.y = Math.random() * height;
          this.size = Math.random() * 1.5 + 0.5;
          this.speed = this.size * 0.3;
          this.alpha = Math.random() * 0.5 + 0.5;
        }
      }

      draw() {
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, 2 * Math.PI);
        ctx.fillStyle = `rgba(255, 255, 255, ${this.alpha})`;
        ctx.shadowColor = 'white';
        ctx.shadowBlur = 5;
        ctx.fill();
      }
    }

    // Setup Canvas Größe
    function resize() {
      width = window.innerWidth;
      height = window.innerHeight;
      canvas.width = width;
      canvas.height = height;
    }

    window.addEventListener('resize', resize);
    resize();

    // Erzeuge Sterne
    const stars = [];
    const starCount = 150;
    for (let i = 0; i < starCount; i++) {
      stars.push(new Star());
    }

    // Animation Loop
    function animate() {
      ctx.clearRect(0, 0, width, height);
      for (const star of stars) {
        star.update();
        star.draw();
      }
      requestAnimationFrame(animate);
    }

    animate();
  </script>
</body>
</html>
