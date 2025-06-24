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
  <div>🪐 Plattformen: <span id="platform">1</span> (100x100 gehört dir)</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
<script>
  // --- Spiel-Variablen ---
  let credits = 200;
  let labor = 0;
  let handel = 0;
  let solar = 0;
  let platformCount = 1; // Große Plattform gehört dir sofort

  // Preise (steigen pro Kauf)
  let priceSolar = 100;
  let priceLabor = 100;
  let priceHandel = 250;
  let priceHouseObj = 50; // Hausobjekt
  let priceSolarInc = 100; // Solarpanel Preissteigerung pro Kauf
  let priceHouseObjInc = 50; // Hausobjekt Preissteigerung pro Kauf

  // Anzeigen aktualisieren
  function updateUI() {
    document.getElementById('credits').innerText = Math.floor(credits);
    document.getElementById('labor').innerText = labor;
    document.getElementById('handel').innerText = handel;
    document.getElementById('solar').innerText = solar;
    document.getElementById('platform').innerText = platformCount;
  }

  // --- Three.js Setup ---
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x87ceeb); // Blauer Himmel

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  
  // Kamera Position etwas hinter und über der Figur, damit man von hinten sieht
  camera.position.set(0, 10, 15);

  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  // Controls nur für Zoom (keine Drehung, keine Pan)
  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enableRotate = false;
  controls.enablePan = false;
  controls.minDistance = 5;
  controls.maxDistance = 40;
  controls.update();

  // Licht
  const dirLight = new THREE.DirectionalLight(0xffffff, 1);
  dirLight.position.set(10, 30, 10);
  scene.add(dirLight);

  const ambLight = new THREE.AmbientLight(0x404040);
  scene.add(ambLight);

  // --- Boden & große Plattform ---
  const platformGeo = new THREE.PlaneGeometry(100, 100);
  const platformMat = new THREE.MeshStandardMaterial({color: 0x888888});
  const platform = new THREE.Mesh(platformGeo, platformMat);
  platform.rotation.x = -Math.PI / 2;
  scene.add(platform);

  // Gras nur am Rand (rand 10m breit)
  function addGrassPatch(x, z) {
    const grassGeo = new THREE.PlaneGeometry(10, 10);
    const grassMat = new THREE.MeshStandardMaterial({color: 0x228B22});
    const grass = new THREE.Mesh(grassGeo, grassMat);
    grass.rotation.x = -Math.PI / 2;
    grass.position.set(x, 0.01, z);
    scene.add(grass);
  }
  for(let i = -45; i <= 45; i += 10) {
    addGrassPatch(-50, i);
    addGrassPatch(50, i);
    addGrassPatch(i, -50);
    addGrassPatch(i, 50);
  }

  // --- Spieler als Würfel ---
  const playerGeo = new THREE.BoxGeometry(1,2,1);
  const playerMat = new THREE.MeshStandardMaterial({color: 0xff5500});
  const player = new THREE.Mesh(playerGeo, playerMat);
  player.position.y = 1; // Steht auf Plattform
  scene.add(player);

  // --- Häuser (einfache geometrische Objekte mit echten Farben) ---
  const houseObjs = [];

  function createHouse(x, z) {
    const baseColor = 0x8B4513;
    const windowColor = 0xFFFFCC;
    const roofColor = 0xA52A2A;

    const base = new THREE.Mesh(
      new THREE.BoxGeometry(4, 2, 4),
      new THREE.MeshStandardMaterial({color: baseColor, transparent:true, opacity:0.4})
    );
    base.position.set(x, 1, z);
    scene.add(base);
    houseObjs.push({mesh: base, bought: false});

    const window1 = new THREE.Mesh(
      new THREE.BoxGeometry(1,1,0.1),
      new THREE.MeshStandardMaterial({color: windowColor, transparent:true, opacity:0.4})
    );
    window1.position.set(x - 1, 1.5, z + 2.05);
    scene.add(window1);
    houseObjs.push({mesh: window1, bought: false});

    const window2 = window1.clone();
    window2.position.set(x + 1, 1.5, z + 2.05);
    scene.add(window2);
    houseObjs.push({mesh: window2, bought: false});

    const secondFloor = new THREE.Mesh(
      new THREE.BoxGeometry(4, 2, 4),
      new THREE.MeshStandardMaterial({color: baseColor, transparent:true, opacity:0.4})
    );
    secondFloor.position.set(x, 3.5, z);
    scene.add(secondFloor);
    houseObjs.push({mesh: secondFloor, bought: false});

    const roof = new THREE.Mesh(
      new THREE.ConeGeometry(3, 2, 4),
      new THREE.MeshStandardMaterial({color: roofColor, transparent:true, opacity:0.4})
    );
    roof.position.set(x, 5.5, z);
    roof.rotation.y = Math.PI / 4;
    scene.add(roof);
    houseObjs.push({mesh: roof, bought: false});
  }

  // Beispielhäuser
  createHouse(10, 10);
  createHouse(-10, -10);
  createHouse(20, -20);

  // --- Bäume ---
  function addTree(x, z) {
    const trunk = new THREE.Mesh(
      new THREE.CylinderGeometry(0.3, 0.3, 3),
      new THREE.MeshStandardMaterial({color: 0x8B4513})
    );
    trunk.position.set(x, 1.5, z);
    scene.add(trunk);

    const leaves = new THREE.Mesh(
      new THREE.SphereGeometry(2),
      new THREE.MeshStandardMaterial({color: 0x006400})
    );
    leaves.position.set(x, 4, z);
    scene.add(leaves);
  }
  for(let i=0; i<20; i++) {
    addTree(Math.random()*90-45, Math.random()*90-45);
  }

  // --- Kaufkreise ---
  const buyTriggers = [];

  houseObjs.forEach((obj, i) => {
    const trigger = new THREE.Mesh(
      new THREE.CircleGeometry(0.7, 32),
      new THREE.MeshBasicMaterial({color: 0x00ffcc, transparent: true, opacity: 0.5})
    );
    trigger.rotation.x = -Math.PI/2;
    trigger.position.set(obj.mesh.position.x, 0.02, obj.mesh.position.z);
    scene.add(trigger);
    buyTriggers.push({mesh: trigger, type:'houseObj', objIndex: i, bought:false});
  });

  function addBuyTrigger(x, z, type) {
    const colorMap = {
      'solar': 0x00ffff,
      'labor': 0xffff00,
      'handel': 0xff00ff,
    };
    const trigger = new THREE.Mesh(
      new THREE.CircleGeometry(0.7, 32),
      new THREE.MeshBasicMaterial({color: colorMap[type] || 0x00ffcc, transparent:true, opacity: 0.5})
    );
    trigger.rotation.x = -Math.PI/2;
    trigger.position.set(x, 0.02, z);
    scene.add(trigger);
    buyTriggers.push({mesh: trigger, type: type, bought:false});
  }

  for(let i=0; i<500; i++) {
    const x = Math.random()*90-45;
    const z = Math.random()*90-45;
    const types = ['solar','labor','handel'];
    const t = types[Math.floor(Math.random()*types.length)];
    addBuyTrigger(x, z, t);
  }

  // --- Steuerung via Maus ---
  let mouseDown = false;
  let lastMouseX = 0;
  let lastMouseY = 0;
  let moveDirection = { x: 0, z: 0 };

  document.addEventListener('mousedown', e => {
    if(e.button === 0) { // Linke Maustaste
      mouseDown = true;
      lastMouseX = e.clientX;
      lastMouseY = e.clientY;
    }
  });

  document.addEventListener('mouseup', e => {
    if(e.button === 0) {
      mouseDown = false;
      moveDirection.x = 0;
      moveDirection.z = 0;
    }
  });

  document.addEventListener('mousemove', e => {
    if (!mouseDown) return;
    const deltaX = e.clientX - lastMouseX;
    const deltaY = e.clientY - lastMouseY;
    lastMouseX = e.clientX;
    lastMouseY = e.clientY;

    const sensitivity = 0.05;

    moveDirection.x = deltaX * sensitivity;
    moveDirection.z = deltaY * sensitivity;
  });

  // --- Bewegung Spielfigur ---
  function movePlayer() {
    const speed = 0.1;

    player.position.x += moveDirection.x * speed;
    player.position.z += moveDirection.z * speed;

    if(player.position.y < 1) player.position.y = 1;

    // Grenzen der Plattform
    if(player.position.x > 49.5) player.position.x = 49.5;
    if(player.position.x < -49.5) player.position.x = -49.5;
    if(player.position.z > 49.5) player.position.z = 49.5;
    if(player.position.z < -49.5) player.position.z = -49.5;
  }

  // --- Kaufen per Überlaufen ---
  function checkBuy() {
    for(let i=0; i < buyTriggers.length; i++) {
      const bt = buyTriggers[i];
      if(bt.bought) continue;

      const dist = player.position.distanceTo(bt.mesh.position);
      if(dist < 1) {
        if(bt.type === 'houseObj') {
          if(credits >= priceHouseObj) {
            credits -= priceHouseObj;
            bt.bought = true;
            houseObjs[bt.objIndex].bought = true;
            houseObjs[bt.objIndex].mesh.material.opacity = 1;
            houseObjs[bt.objIndex].mesh.material.transparent = false;
            priceHouseObj += priceHouseObjInc;
            updateUI();
            playSound();
          }
        } else if(bt.type === 'solar') {
          if(credits >= priceSolar) {
            credits -= priceSolar;
            solar++;
            bt.bought = true;
            priceSolar += priceSolarInc;
            updateUI();
            playSound();
          }
        } else if(bt.type === 'labor') {
          if(credits >= priceLabor) {
            credits -= priceLabor;
            labor++;
            bt.bought = true;
            updateUI();
            playSound();
          }
        } else if(bt.type === 'handel') {
          if(credits >= priceHandel) {
            credits -= priceHandel;
            handel++;
            bt.bought = true;
            updateUI();
            playSound();
          }
        }
      }
    }
  }

  // --- Sound ---
  function playSound() {
    const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
    audio.volume = 0.3;
    audio.play();
  }

  // --- Einnahmen pro Sekunde ---
  setInterval(() => {
    credits += solar * 100 + labor * 50 + handel * 150;
    updateUI();
  }, 1000);

  // --- Kamera folgt Spieler ---
  function updateCamera() {
    // Position hinter Spieler
    const offset = new THREE.Vector3(0, 10, 15);
    const desiredPos = player.position.clone().add(offset);
    camera.position.lerp(desiredPos, 0.1);

    // Kamera schaut auf Spieler
    camera.lookAt(player.position);
  }

  // --- Animation ---
  function animate() {
    requestAnimationFrame(animate);
    movePlayer();
    checkBuy();
    updateCamera();
    renderer.render(scene, camera);
  }
  animate();

  // Resize-Handler
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });

  updateUI();
</script>
</body>
</html>
