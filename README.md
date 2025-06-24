<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon 3D Anime Style</title>
  <style>
    body { margin: 0; overflow: hidden; font-family: Arial, sans-serif; }
    #ui {
      position: absolute;
      top: 10px; left: 10px;
      background: rgba(0,0,0,0.75);
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ffcc;
      z-index: 10;
      color: white;
      user-select: none;
    }
    .circle-button {
      position: absolute;
      width: 60px; height: 60px;
      border-radius: 50%;
      background: #00ffcc;
      display: flex; align-items: center; justify-content: center;
      font-size: 24px; color: black; font-weight: bold;
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
    <div>🔬 Labor: <span id="labor">0</span></div>
    <div>🏪 Handelsstation: <span id="handel">0</span></div>
    <div>🔋 Solarpanels: <span id="solar">0</span></div>
    <div>🪐 Plattformen: <span id="platform">0</span></div>
    <div>⭐ Level: <span id="level">1</span> (XP: <span id="xp">0</span>)</div>
  </div>

  <div id="buySolar" class="circle-button" onclick="buy('solar')">🔋</div>
  <div id="buyLabor" class="circle-button" onclick="buy('labor')">🧪</div>
  <div id="buyHandel" class="circle-button" onclick="buy('handel')">🏪</div>
  <div id="buyPlatform" class="circle-button" onclick="buy('platform')">🪐</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
  <script>
    // Ressourcen & Level
    let credits = 200;
    let energy = 1000;
    let solar = 0;
    let labor = 0;
    let handel = 0;
    let platform = 0;
    let xp = 0;
    let level = 1;

    // Level XP für nächste Stufe
    function xpForNextLevel(lv) {
      return lv * lv * 50;
    }

    // UI update
    function updateUI() {
      document.getElementById('credits').innerText = credits;
      document.getElementById('labor').innerText = labor;
      document.getElementById('handel').innerText = handel;
      document.getElementById('solar').innerText = solar;
      document.getElementById('platform').innerText = platform;
      document.getElementById('xp').innerText = xp;
      document.getElementById('level').innerText = level;
    }

    // Kaufen
    function buy(type) {
      if (type === 'solar' && credits >= 100) {
        credits -= 100;
        solar++;
        addSolar();
        addXP(20);
      } else if (type === 'labor' && credits >= 100) {
        credits -= 100;
        labor++;
        addLabor();
        addXP(20);
      } else if (type === 'handel' && energy >= 250) {
        energy -= 250;
        handel++;
        addHandel();
        addXP(30);
      } else if (type === 'platform') {
        platform++;
        addPlatform(player.position.x + 5, player.position.z);
        addXP(10);
      } else {
        alert('Nicht genug Ressourcen');
        return;
      }
      updateUI();
      playSound();
    }

    function addXP(amount) {
      xp += amount;
      while (xp >= xpForNextLevel(level)) {
        xp -= xpForNextLevel(level);
        level++;
        alert(`Level Up! Du bist jetzt Level ${level}`);
      }
      updateUI();
    }

    // Ressourcen pro Sekunde
    setInterval(() => {
      credits += solar * 10 + labor * 5 + handel * 20;
      energy += solar * 20;
      updateUI();
    }, 1000);

    // Three.js Szene Setup
    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#87ceeb');
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);

    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;

    // Licht
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);

    const ambientLight = new THREE.AmbientLight(0x404040);
    scene.add(ambientLight);

    // Boden
    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(500, 500),
      new THREE.MeshStandardMaterial({ color: '#228B22' })
    );
    ground.rotation.x = -Math.PI / 2;
    scene.add(ground);

    // Anime-Stil Spieler-Figur: Kopf, Körper, Arme, Beine
    const player = new THREE.Group();

    // Kopf
    const headGeometry = new THREE.BoxGeometry(1,1,1);
    const headMaterial = new THREE.MeshStandardMaterial({ color: '#ffccff' });
    const head = new THREE.Mesh(headGeometry, headMaterial);
    head.position.set(0,1.75,0);
    player.add(head);

    // Körper
    const bodyGeometry = new THREE.BoxGeometry(1,1.5,0.5);
    const bodyMaterial = new THREE.MeshStandardMaterial({ color: '#ff99cc' });
    const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
    body.position.set(0,1,0);
    player.add(body);

    // Arme
    const armGeometry = new THREE.BoxGeometry(0.3,1,0.3);
    const armMaterial = new THREE.MeshStandardMaterial({ color: '#ff99cc' });
    const leftArm = new THREE.Mesh(armGeometry, armMaterial);
    leftArm.position.set(-0.65,1,0);
    player.add(leftArm);

    const rightArm = new THREE.Mesh(armGeometry, armMaterial);
    rightArm.position.set(0.65,1,0);
    player.add(rightArm);

    // Beine
    const legGeometry = new THREE.BoxGeometry(0.4,1,0.4);
    const legMaterial = new THREE.MeshStandardMaterial({ color: '#cc6699' });
    const leftLeg = new THREE.Mesh(legGeometry, legMaterial);
    leftLeg.position.set(-0.3,0,0);
    player.add(leftLeg);

    const rightLeg = new THREE.Mesh(legGeometry, legMaterial);
    rightLeg.position.set(0.3,0,0);
    player.add(rightLeg);

    player.position.set(0,1,0);
    scene.add(player);

    // Variablen für Bewegung & Physik
    let velocityY = 0;
    let isOnGround = true;
    const gravity = -0.02;
    const jumpStrength = 0.4;
    const moveSpeed = 0.15;

    // Gebäude (Häuser) - begehbar
    const houses = [];

    function createHouse(x,z) {
      const house = new THREE.Group();

      // Wände
      const wallsGeom = new THREE.BoxGeometry(3,3,3);
      const wallsMat = new THREE.MeshStandardMaterial({color:'#8B4513'});
      const walls = new THREE.Mesh(wallsGeom, wallsMat);
      walls.position.y = 1.5;
      house.add(walls);

      // Dach
      const roofGeom = new THREE.ConeGeometry(2.5,1,4);
      const roofMat = new THREE.MeshStandardMaterial({color:'#A0522D'});
      const roof = new THREE.Mesh(roofGeom, roofMat);
      roof.position.y = 3.5;
      roof.rotation.y = Math.PI / 4;
      house.add(roof);

      house.position.set(x,0,z);
      scene.add(house);

      // Tür als Box für Kollision
      const door = {
        xMin: x - 0.5,
        xMax: x + 0.5,
        zMin: z + 1.5,
        zMax: z + 2.5,
        entered: false
      };
      houses.push(door);
    }

    // Erzeuge ein paar Häuser
    for(let i=0; i<5; i++) {
      createHouse(Math.random()*50-25, Math.random()*50-25);
    }

    // Steuerung
    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    // Begehbares Haus-Check
    let insideHouse = false;
    function checkHouseEntry() {
      insideHouse = false;
      for (const door of houses) {
        if (player.position.x > door.xMin && player.position.x < door.xMax &&
            player.position.z > door.zMin && player.position.z < door.zMax) {
          if(!door.entered) {
            alert("Du bist ins Haus eingetreten!");
            door.entered = true;
          }
          insideHouse = true;
          break;
        }
      }
    }

    // Kaufen Gebäude / Module
    const triggers = [];
    function addPlatform(x, z) {
      const box = new THREE.Mesh(
        new THREE.BoxGeometry(3, 0.3, 3),
        new THREE.MeshStandardMaterial({ color: '#808080' })
      );
      box.position.set(x, 0.15, z);
      scene.add(box);

      // Beispiel-Trigger (optional)
      const trigger = new THREE.Mesh(
        new THREE.SphereGeometry(0.4),
        new THREE.MeshStandardMaterial({ color: '#ff0000', transparent: true, opacity: 0.6 })
      );
      trigger.position.set(x, 0.8, z);
      trigger.userData.type = 'labor';
      scene.add(trigger);
      triggers.push(trigger);
    }

    function addSolar() {
      const obj = new THREE.Mesh(
        new THREE.CylinderGeometry(0.3, 0.3, 2),
        new THREE.MeshStandardMaterial({ color: '#00ffff' })
      );
      obj.position.set(Math.random()*20-10, 1, Math.random()*20-10);
      scene.add(obj);
    }

    function addLabor() {
      const obj = new THREE.Mesh(
        new THREE.BoxGeometry(1, 2, 1),
        new THREE.MeshStandardMaterial({ color: '#ffcc00' })
      );
      obj.position.set(Math.random()*20-10, 1, Math.random()*20-10);
      scene.add(obj);
    }

    function addHandel() {
      const obj = new THREE.Mesh(
        new THREE.BoxGeometry(2, 2, 2),
        new THREE.MeshStandardMaterial({ color: '#cc00ff' })
      );
      obj.position.set(Math.random()*20-10, 1, Math.random()*20-10);
      scene.add(obj);
    }

    // Spielsound bei Kauf
    function playSound() {
      const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
      audio.play();
    }

    // Haupt-Animation & Bewegung
    function animate() {
      // Bewegung
      if (keys['w']) player.position.z -= moveSpeed;
      if (keys['s']) player.position.z += moveSpeed;
      if (keys['a']) player.position.x -= moveSpeed;
      if (keys['d']) player.position.x += moveSpeed;

      // Springen
      if (keys[' '] && isOnGround) {
        velocityY = jumpStrength;
        isOnGround = false;
      }
      player.position.y += velocityY;
      velocityY += gravity;

      if (player.position.y <= 1) {
        player.position.y = 1;
        velocityY = 0;
        isOnGround = true;
      }

      checkHouseEntry();

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }
    animate();

    // Fenster-Resize
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    updateUI();
  </script>
</body>
</html>
