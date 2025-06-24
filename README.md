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
    <div>🔬 Labor: <span id="labor">0</span></div>
    <div>🏪 Handelsstationen: <span id="handel">0</span></div>
    <div>🔋 Solarpanels: <span id="solar">0</span></div>
    <div>🪐 Plattformen: <span id="platform">1</span> (100x100 Startgebiet)</div>
    <div>⭐ Level: <span id="level">1</span></div>
  </div>

  <div id="buySolar" class="circle-button" onclick="buy('solar')">🔋</div>
  <div id="buyLabor" class="circle-button" onclick="buy('labor')">🧪</div>
  <div id="buyHandel" class="circle-button" onclick="buy('handel')">🏪</div>
  <div id="buyPlatform" class="circle-button" onclick="buy('platform')">🪐</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
  <script>
    let credits = 200;
    let energy = 1000;
    let solar = 0;
    let labor = 0;
    let handel = 0;
    let platform = 1;
    let xp = 0;
    let level = 1;

    function updateUI() {
      document.getElementById('credits').innerText = credits;
      document.getElementById('labor').innerText = labor;
      document.getElementById('handel').innerText = handel;
      document.getElementById('solar').innerText = solar;
      document.getElementById('platform').innerText = platform;
      document.getElementById('level').innerText = level;
    }

    function gainXP(amount) {
      xp += amount;
      if (xp >= 100) {
        xp -= 100;
        level++;
        updateUI();
      }
    }

    function buy(type) {
      if (type === 'solar' && credits >= 100) {
        credits -= 100;
        solar++;
        addSolar();
        gainXP(10);
      } else if (type === 'labor' && credits >= 100) {
        credits -= 100;
        labor++;
        addLabor();
        gainXP(10);
      } else if (type === 'handel' && energy >= 250) {
        energy -= 250;
        handel++;
        addHandel();
        gainXP(20);
      } else if (type === 'platform') {
        platform++;
        addPlatform(player.position.x + 5, player.position.z);
        gainXP(5);
      } else {
        alert('Nicht genug Ressourcen');
        return;
      }
      updateUI();
      playSound();
    }

    setInterval(() => {
      credits += solar * 10 + labor * 5 + handel * 20;
      energy += solar * 20;
      updateUI();
    }, 1000);

    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#87ceeb');
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);

    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;

    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);

    const ambientLight = new THREE.AmbientLight(0x404040);
    scene.add(ambientLight);

    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(500, 500),
      new THREE.MeshStandardMaterial({ color: '#888888' })
    );
    ground.rotation.x = -Math.PI / 2;
    scene.add(ground);

    const player = new THREE.Mesh(
      new THREE.BoxGeometry(0.8, 1.6, 0.8),
      new THREE.MeshStandardMaterial({ color: '#ff69b4' })
    );
    player.position.y = 0.8;
    scene.add(player);

    addPlatform(0, 0); // Start-Plattform

    let velocityY = 0;
    let isOnGround = true;

    function addPlatform(x, z) {
      const box = new THREE.Mesh(
        new THREE.BoxGeometry(3, 0.3, 3),
        new THREE.MeshStandardMaterial({ color: '#808080' })
      );
      box.position.set(x, 0.15, z);
      scene.add(box);
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

    function playSound() {
      const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
      audio.play();
    }

    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    function animate() {
      if (keys['w']) player.position.z -= 0.1;
      if (keys['s']) player.position.z += 0.1;
      if (keys['a']) player.position.x -= 0.1;
      if (keys['d']) player.position.x += 0.1;

      if (keys[' '] && isOnGround) {
        velocityY = 0.15;
        isOnGround = false;
      }

      player.position.y += velocityY;
      velocityY -= 0.01;

      if (player.position.y <= 0.8) {
        player.position.y = 0.8;
        velocityY = 0;
        isOnGround = true;
      }

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }
    animate();
    updateUI();
  </script>
</body>
</html>
