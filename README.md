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
    font-weight: bold;
    font-size: 1.1em;
  }
</style>
</head>
<body>
<div id="ui">
  <h1>🚀 Space Station Tycoon</h1>
  <div>💰 Credits: <span id="credits">200</span></div>
  <div>🏗️ Objekte gekauft: <span id="objects">0</span></div>
  <div>🔋 Energie (Solarpanels): <span id="solar">0</span></div>
  <div>🏪 Handelsstationen: <span id="handel">0</span></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
<script>
  let credits = 200;
  let solar = 0;
  let handel = 0;
  let objectsBought = 0;

  let priceSolar = 500;
  let priceHandel = 1000;
  let priceHouseObj = 300;
  let priceSolarInc = 300;
  let priceHandelInc = 500;
  let priceHouseObjInc = 200;

  // Three.js Setup
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb);

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 2000);
  camera.position.set(0, 120, 150);
  camera.lookAt(0, 0, 0);

  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);
  controls.enablePan = false;
  controls.minDistance = 50;
  controls.maxDistance = 500;
  controls.update();

  // Licht
  const dirLight = new THREE.DirectionalLight(0xffffff, 1);
  dirLight.position.set(100, 200, 100);
  scene.add(dirLight);

  const ambLight = new THREE.AmbientLight(0x404040);
  scene.add(ambLight);

  // Plattform 1000x1000 in 10x10 Meter Fliesen => 100x100 Fliesen
  const platformSize = 1000;
  const tileSize = 10;
  const tilesCount = platformSize / tileSize;

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

  // Gekaufte Objekte sichtbar machen
  const purchasedObjects = [];

  function createPurchasedObject(x, z, type) {
    let color;
    if(type === 'solar') color = 0xffff00;
    else if(type === 'handel') color = 0xff00ff;
    else color = 0x00ffff; // Hausobjekt oder sonstiges

    const geo = new THREE.BoxGeometry(2, 4, 2);
    const mat = new THREE.MeshStandardMaterial({color});
    const obj = new THREE.Mesh(geo, mat);
    obj.position.set(x, 2, z);
    scene.add(obj);
    purchasedObjects.push(obj);
  }

  // Kauftrigger: 1500 zufällige Kaufpunkte
  const buyTriggers = [];

  function addBuyTrigger(x, z, type, tileIndex) {
    const colorMap = {
      'solar': 0xffff00,
      'handel': 0xff00ff,
      'houseObj': 0x00ffff
    };
    const trigger = new THREE.Mesh(
      new THREE.CircleGeometry(0.7, 32),
      new THREE.MeshBasicMaterial({color: colorMap[type] || 0x00ffff, transparent:true, opacity: 0.5})
    );
    trigger.rotation.x = -Math.PI/2;
    trigger.position.set(x, 0.02, z);
    scene.add(trigger);
    buyTriggers.push({mesh: trigger, type: type, bought:false, tileIndex, x, z});
  }

  // Verteilt 1500 Kaufpunkte zufällig
  for(let i=0; i<1500; i++) {
    const tileIndex = Math.floor(Math.random()*platformTiles.length);
    const tile = platformTiles[tileIndex];
    const jitter = (tileSize/2 - 1);
    const x = tile.mesh.position.x + (Math.random()*2 -1)*jitter;
    const z = tile.mesh.position.z + (Math.random()*2 -1)*jitter;
    const types = ['solar','handel','houseObj'];
    const t = types[Math.floor(Math.random()*types.length)];
    addBuyTrigger(x, z, t, tileIndex);
  }

  // Steuerung & Bewegung
  const keys = {};
  document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
  document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

  function movePlayer() {
    const speed = 0.6;
    if(keys['w']) player.position.z -= speed;
    if(keys['s']) player.position.z += speed;
    if(keys['a']) player.position.x -= speed;
    if(keys['d']) player.position.x += speed;

    const limit = platformSize/2 - 1;
    if(player.position.x > limit) player.position.x = limit;
    if(player.position.x < -limit) player.position.x = -limit;
    if(player.position.z > limit) player.position.z = limit;
    if(player.position.z < -limit) player.position.z = -limit;
  }

  // Kauflogik + Objekte sichtbar machen + Kaufkreise entfernen
  function checkBuy() {
    for(let i=0; i < buyTriggers.length; i++) {
      const bt = buyTriggers[i];
      if(bt.bought) continue;

      const dist = player.position.distanceTo(bt.mesh.position);
      if(dist < 1) {
        let cost = 0;
        if(bt.type === 'solar') cost = priceSolar;
        else if(bt.type === 'handel') cost = priceHandel;
        else cost = priceHouseObj;

        if(credits >= cost) {
          credits -= cost;
          bt.bought = true;
          scene.remove(bt.mesh); // Kaufkreis wegnehmen
          createPurchasedObject(bt.x, bt.z, bt.type);

          const tile = platformTiles[bt.tileIndex];
          if(tile && !tile.bought) {
            tile.bought = true;
            tile.mesh.material.color.set(0x00ff99);
          }

          if(bt.type === 'solar') {
            solar++;
            priceSolar += priceSolarInc;
          } else if(bt.type === 'handel') {
            handel++;
            priceHandel += priceHandelInc;
          } else {
            objectsBought++;
            priceHouseObj += priceHouseObjInc;
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
    credits += solar * 100 + handel * 150;
    updateUI();
  }, 1000);

  // UI Update
  function updateUI() {
    document.getElementById('credits').innerText = Math.floor(credits);
    document.getElementById('solar').innerText = solar;
    document.getElementById('handel').innerText = handel;
    document.getElementById('objects').innerText = objectsBought;
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
