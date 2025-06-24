<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon 3D</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
    }
    #ui {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(0,0,0,0.75);
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ffcc;
      z-index: 10;
      color: white;
      font-family: Arial, sans-serif;
    }
    .circle-button {
      position: absolute;
      width: 60px;
      height: 60px;
      border-radius: 50%;
      background: #00ffcc;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 24px;
      color: black;
      font-weight: bold;
      cursor: pointer;
      box-shadow: 0 0 10px #00ffcc;
      transition: 0.2s;
    }
    .circle-button:hover {
      background: #00aaaa;
      box-shadow: 0 0 15px #00aaaa;
    }
    #buySolar { bottom: 30px; right: 30px; }
    #buyLabor { bottom: 100px; right: 30px; }
    #buyHandel { bottom: 170px; right: 30px; }
    #buyPlatform { bottom: 240px; right: 30px; }
  </style>
</head>
<body>
  <div id="ui">
    <h1>🚀 Space Station Tycoon</h1>
    <div>💰 Credits: <span id="credits">200</span></div>
    <div>🧪 Labore: <span id="labor">0</span></div>
    <div>🏪 Handelsstationen: <span id="handel">0</span></div>
    <div>🔋 Solarpanels: <span id="solar">0</span></div>
    <div>🪐 Plattformen: <span id="platform">1</span> (Startgebiet: 100x100)</div>
    <div>⭐ Level: <span id="level">1</span></div>
  </div>

  <div id="buySolar" class="circle-button" onclick="buy('solar')">🔋</div>
  <div id="buyLabor" class="circle-button" onclick="buy('labor')">🧪</div>
  <div id="buyHandel" class="circle-button" onclick="buy('handel')">🏪</div>
  <div id="buyPlatform" class="circle-button" onclick="buy('platform')">🪐</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
  <script>
    // ... (restlicher Scriptcode bleibt unverändert)
  </script>
</body>
</html>
