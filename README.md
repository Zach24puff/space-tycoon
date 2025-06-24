<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Space Station Tycoon.io</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #0b0f2a;
      color: white;
      padding: 20px;
    }
    h1 { color: #00ffcc; }
    .btn {
      margin: 5px;
      padding: 10px;
      background: #222;
      border: 1px solid #00ffcc;
      color: white;
      cursor: pointer;
    }
    .btn:hover {
      background: #00ffcc;
      color: #000;
    }
    .module {
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <h1>🚀 Space Station Tycoon.io</h1>
  <p>⚡ Energie: <span id="energie">0</span> | 💰 Credits: <span id="credits">0</span> | 🔬 Forschung: <span id="forschung">0</span></p>

  <div class="module">
    <button class="btn" onclick="build('solar')">🔋 Solarpanel (100💰)</button>
    <button class="btn" onclick="build('handel')">🏪 Handelsstation (250⚡)</button>
    <button class="btn" onclick="build('labor')">🧪 Labor (500💰)</button>
  </div>

  <div class="module">
    <button class="btn" onclick="upgrade('solar')">🔋 Upgrade Solar (1000🔬)</button>
    <button class="btn" onclick="upgrade('handel')">🏪 Upgrade Handel (1000🔬)</button>
    <button class="btn" onclick="upgrade('labor')">🧪 Upgrade Labor (1000🔬)</button>
  </div>

  <div class="module">
    <button class="btn" onclick="saveGame()">💾 Speichern</button>
    <button class="btn" onclick="loadGame()">📂 Laden</button>
  </div>

  <script>
    let energie = 0, credits = 200, forschung = 0;
    let solarpanels = 0, handelsstationen = 0, labore = 0;
    let solarBoost = 1, handelBoost = 1, laborBoost = 1;

    function update() {
      document.getElementById('energie').innerText = Math.floor(energie);
      document.getElementById('credits').innerText = Math.floor(credits);
      document.getElementById('forschung').innerText = Math.floor(forschung);
    }

    function build(typ) {
      if (typ === 'solar' && credits >= 100) { credits -= 100; solarpanels++; }
      if (typ === 'handel' && energie >= 250) { energie -= 250; handelsstationen++; }
      if (typ === 'labor' && credits >= 500) { credits -= 500; labore++; }
      update();
    }

    function upgrade(typ) {
      if (forschung < 1000) return;
      forschung -= 1000;
      if (typ === 'solar') solarBoost *= 1.25;
      if (typ === 'handel') handelBoost *= 1.25;
      if (typ === 'labor') laborBoost *= 1.25;
      update();
    }

    function saveGame() {
      const save = {energie, credits, forschung, solarpanels, handelsstationen, labore, solarBoost, handelBoost, laborBoost};
      localStorage.setItem('tycoonSave', JSON.stringify(save));
      alert('Spiel gespeichert!');
    }

    function loadGame() {
      const save = JSON.parse(localStorage.getItem('tycoonSave'));
      if (!save) return alert('Kein Spielstand gefunden.');
      ({energie, credits, forschung, solarpanels, handelsstationen, labore, solarBoost, handelBoost, laborBoost} = save);
      update();
    }

    setInterval(() => {
      energie += solarpanels * 5 * solarBoost;
      credits += handelsstationen * 10 * handelBoost;
      forschung += labore * 5 * laborBoost;
      update();
    }, 1000);

    update();
  </script>
</body>
</html>
