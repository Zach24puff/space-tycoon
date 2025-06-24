<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
  <title>Space Station Tycoon 3D - Mobile</title>
  <style>
    html, body {
      margin: 0; padding: 0; overflow: hidden;
      background-color: #87ceeb; /* Himmel hellblau */
      touch-action: none;
      -webkit-user-select: none;
      user-select: none;
    }
    #ui {
      position: fixed;
      top: 10px; left: 10px;
      background: rgba(0,0,0,0.75);
      color: white;
      padding: 10px 15px;
      border-radius: 8px;
      font-family: Arial, sans-serif;
      font-size: 1.2em;
      z-index: 10;
      max-width: 320px;
      width: 90vw;
    }
    #controls {
      position: fixed;
      bottom: 20px; left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 15px;
      z-index: 10;
    }
    button {
      width: 60px; height: 60px;
      border-radius: 50%;
      border: none;
      background: rgba(0,255,204,0.7);
      font-size: 1.5em;
      color: #004d40;
      user-select: none;
      touch-action: manipulation;
      box-shadow: 0 0 10px #00ffcc;
    }
    button:active {
      background: rgba(0,255,204,1);
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
  <div>🪐 Plattformen: <span id="platform">1</span></div>
</div>

<div id="controls">
  <button id="leftBtn">◀</button>
  <button id="jumpBtn">⭠↑⭢</button>
  <button id="rightBtn">▶</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
<script>
  // Szene Setup
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb); // Himmelblau

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  camera.position.set(0, 20, 30);

  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);
  controls.enablePan = false;
  controls.minDistance = 10;
  controls.maxDistance = 80;
  controls.update();

  // Licht
  const dirLight = new THREE.DirectionalLight(0xffffff, 1);
  dirLight.position.set(10, 30, 10);
  scene.add(dirLight);

  const ambLight = new THREE.AmbientLight(0x606060);
  scene.add(ambLight);

  // Plattform (100x100)
  const platformGeo = new THREE.PlaneGeometry(100, 100);
  const platformMat = new THREE.MeshStandardMaterial({color: 0x888888});
  const platform = new THREE.Mesh(platformGeo, platformMat);
  platform.rotation.x = -Math.PI / 2;
  scene.add(platform);

  // Spieler Würfel
  const playerGeo = new THREE.BoxGeometry(1, 2, 1);
  const playerMat = new THREE.MeshStandardMaterial({color: 0xff5500});
  const player = new THREE.Mesh(playerGeo, playerMat);
  player.position.y = 1;
  scene.add(player);

  // Steuerung - Variablen
  const move = { left: false, right: false, forward: false, backward: false, jump: false };
  let velocityY = 0;
  let isOnGround = true;

  // Buttons
  const leftBtn = document.getElementById('leftBtn');
  const rightBtn = document.getElementById('rightBtn');
  const jumpBtn = document.getElementById('jumpBtn');

  leftBtn.addEventListener('touchstart', () => move.left = true);
  leftBtn.addEventListener('touchend', () => move.left = false);
  rightBtn.addEventListener('touchstart', () => move.right = true);
  rightBtn.addEventListener('touchend', () => move.right = false);
  jumpBtn.addEventListener('touchstart', () => {
    if(isOnGround) {
      velocityY = 0.15;
      isOnGround = false;
    }
  });

  // Auch Tastatur fürs Desktop
  window.addEventListener('keydown', e => {
    if(e.key === 'a') move.left = true;
    if(e.key === 'd') move.right = true;
    if(e.key === 'w') move.forward = true;
    if(e.key === 's') move.backward = true;
    if(e.key === ' ') {
      if(isOnGround) {
        velocityY = 0.15;
        isOnGround = false;
      }
    }
  });
  window.addEventListener('keyup', e => {
    if(e.key === 'a') move.left = false;
    if(e.key === 'd') move.right = false;
    if(e.key === 'w') move.forward = false;
    if(e.key === 's') move.backward = false;
  });

  function movePlayer() {
    const speed = 0.3;
    if(move.left) player.position.x -= speed;
    if(move.right) player.position.x += speed;
    if(move.forward) player.position.z -= speed;
    if(move.backward) player.position.z += speed;

    player.position.y += velocityY;
    velocityY -= 0.01;

    if(player.position.y <= 1) {
      player.position.y = 1;
      velocityY = 0;
      isOnGround = true;
    }

    // Grenzen Plattform
    player.position.x = Math.min(49.5, Math.max(-49.5, player.position.x));
    player.position.z = Math.min(49.5, Math.max(-49.5, player.position.z));
  }

  // UI update Variablen
  const ui = {
    credits: 200,
    labor: 0,
    handel: 0,
    solar: 0,
    platform: 1
  };

  function updateUI() {
    document.getElementById('credits').innerText = ui.credits;
    document.getElementById('labor').innerText = ui.labor;
    document.getElementById('handel').innerText = ui.handel;
    document.getElementById('solar').innerText = ui.solar;
    document.getElementById('platform').innerText = ui.platform;
  }

  setInterval(() => {
    ui.credits += ui.solar * 100 + ui.labor * 50 + ui.handel * 150 + 10; // passives Einkommen + Basis 10
    updateUI();
  }, 1000);

  // Animation Loop
  function animate() {
    requestAnimationFrame(animate);
    movePlayer();
    renderer.render(scene, camera);
  }
  animate();

  // Resize
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
</script>

</body>
</html>
