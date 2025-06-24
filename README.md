<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Space Station Tycoon 3D</title>
<style>
  body { margin: 0; overflow: hidden; background-color: #87ceeb; }
  #ui {
    position: absolute;
    top: 10px; left: 10px;
    background: rgba(0,0,0,0.75);
    padding: 15px;
    border-radius: 10px;
    box-shadow: 0 0 10px #00ffcc;
    z-index: 10;
    color: white;
    font-family: Arial, sans-serif;
    width: 220px;
  }
  .circle-trigger {
    position: absolute;
    width: 40px; height: 40px;
    border-radius: 50%;
    background: rgba(0,255,204,0.6);
    box-shadow: 0 0 10px rgba(0,255,204,0.9);
    pointer-events: none;
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
  <div>🪐 Plattformen: <span id="platform">1</span> (200x200 gehört dir)</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
<script>
  let credits = 200;
  let labor = 0;
  let handel = 0;
  let solar = 0;
  let platformCount = 1;

  let priceSolar = 500;
  let priceLabor = 400;
  let priceHandel = 1000;
  let priceHouseObj = 300;
  let priceSolarInc = 300;
  let priceHouseObjInc = 200;

  // Three.js Setup
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb);

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  camera.position.set(0, 40, 50);
  camera.lookAt(0, 0, 0);

  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);
  controls.enablePan = false;
  controls.minDistance = 10;
  controls.maxDistance = 100;
  controls.update();

  // Licht
  const dirLight = new THREE.DirectionalLight(0xffffff, 1);
  dirLight.position.set(10, 30, 10);
  scene.add(dirLight);

  const ambLight = new THREE.AmbientLight(0x404040);
  scene.add(ambLight);

  // Plattform 200x200 in 10x10 Bereiche aufgeteilt (je 20x20 Fliesen)
  const platformSize = 200;
  const tileSize = 10;
  const tilesCount = platformSize / tileSize; // 20

  const platformTiles = [];

  for(let x=0; x < tilesCount; x++) {
    for(let z=0; z < tilesCount; z++) {
      const geo = new THREE.PlaneGeometry(tileSize, tileSize);
      const mat = new THREE.MeshStandardMaterial({color: 0x888888});
      const tile = new THREE.Mesh(geo, mat);
      tile.rotation.x = -Math.PI/2;
      tile.position.set(
        x * tileSize - platformSize/2 + tileSize/2,
        0,
        z * tileSize - platformSize/2 + tileSize/2
      );
      scene.add(tile);
      platformTiles.push({mesh: tile, bought: false});
    }
  }

  // Spieler als Würfel
  const playerGeo = new THREE.BoxGeometry(1,2,1);
  const playerMat = new THREE.MeshStandardMaterial({color: 0xff5500});
  const player = new THREE.Mesh(playerGeo, playerMat);
  player.position.y = 1;
  scene.add(player);

  // Kauftrigger: je 1500 zufällige Plätze auf Plattform (ca. 4 pro Tile)
  const buyTriggers = [];

  function addBuyTrigger(x, z, type, tileIndex) {
    const colorMap = {
      'solar': 0x00ffff,
      'labor': 0xffff00,
      'handel': 0xff00ff,
      'houseObj': 0x00ffcc
    };
    const trigger = new THREE.Mesh(
      new THREE.CircleGeometry(0.7, 32),
      new THREE.MeshBasicMaterial({color: colorMap[type] || 0x00ffcc, transparent:true, opacity: 0.5})
    );
    trigger.rotation.x = -Math.PI/2;
    trigger.position.set(x, 0.02, z);
    scene.add(trigger);
    buyTriggers.push({mesh: trigger, type: type, bought:false, tileIndex});
  }

  // 1500 Kauftrigger zufällig auf Plattform verteilt
  for(let i=0; i<1500; i++) {
    const tileIndex = Math.floor(Math.random()*platformTiles.length);
    const tile = platformTiles[tileIndex];
    // Random Position leicht innerhalb des Tiles
    const jitter = (tileSize/2 - 1);
    const x = tile.mesh.position.x + (Math.random()*2 -1)*jitter;
    const z = tile.mesh.position.z + (Math.random()*2 -1)*jitter;
    const types = ['solar','labor','handel'];
    const t = types[Math.floor(Math.random()*types.length)];
    addBuyTrigger(x, z, t, tileIndex);
  }

  // Steuerung & Bewegung
  const keys = {};
  document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
  document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

  function movePlayer() {
    const speed = 0.4;
    if(keys['w']) player.position.z -= speed;
    if(keys['s']) player.position.z += speed;
    if(keys['a']) player.position.x -= speed;
    if(keys['d']) player.position.x += speed;

    // Begrenzung auf Plattform
    const limit = platformSize/2 - 1;
    if(player.position.x > limit) player.position.x = limit;
    if(player.position.x < -limit) player.position.x = -limit;
    if(player.position.z > limit) player.position.z = limit;
    if(player.position.z < -limit) player.position.z = -limit;
  }

  // Kaufen per Überlaufen
  function checkBuy() {
    for(let i=0; i < buyTriggers.length; i++) {
      const bt = buyTriggers[i];
      if(bt.bought) continue;

      const dist = player.position.distanceTo(bt.mesh.position);
      if(dist < 1) {
        // Preise pro Typ (kannst anpassen)
        let cost = 0;
        if(bt.type === 'solar') cost = priceSolar;
        else if(bt.type === 'labor') cost = priceLabor;
        else if(bt.type === 'handel') cost = priceHandel;
        else cost = priceHouseObj;

        if(credits >= cost) {
          credits -= cost;
          bt.bought = true;
          // Plattform-Kachel färben
          const tile = platformTiles[bt.tileIndex];
          if(tile && !tile.bought) {
            tile.bought = true;
            tile.mesh.material.color.set(0x00ff99); // z.B. grün
          }

          // Erhöhen je nach Typ
          if(bt.type === 'solar') {
            solar++;
            priceSolar += priceSolarInc;
          } else if(bt.type === 'labor') {
            labor++;
          } else if(bt.type === 'handel') {
            handel++;
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
    credits += solar * 100 + labor * 50 + handel * 150;
    updateUI();
  }, 1000);

  // UI Update
  function updateUI() {
    document.getElementById('credits').innerText = Math.floor(credits);
    document.getElementById('labor').innerText = labor;
    document.getElementById('handel').innerText = handel;
    document.getElementById('solar').innerText = solar;
    document.getElementById('platform').innerText = platformCount;
  }

  // Animation
  function animate() {
    requestAnimationFrame(animate);
    movePlayer();
    checkBuy();
    renderer.render(scene, camera);
  }
  animate();

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });

  updateUI();
</script>
</body>
</html>
