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

  <!-- Kaufkreise -->
  <div id="buySolar" class="circle-button" style="bottom: 30px; right: 30px;" title="Solarpanel kaufen (100 Credits)">🔋</div>
  <div id="buyLabor" class="circle-button" style="bottom: 100px; right: 30px;" title="Labor kaufen (100 Credits)">🧪</div>
  <div id="buyHandel" class="circle-button" style="bottom: 170px; right: 30px;" title="Handelsstation kaufen (250 Energie)">🏪</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
  <script>
    let credits = 200;
    let energy = 1000;
    let solar = 0;
    let labor = 0;
    let handel = 0;
    let platform = 1; // Plattform gehört sofort

    // Preise (steigend)
    let priceSolar = 100;
    let priceLabor = 100;
    let priceHandel = 250;

    function updateUI() {
      document.getElementById('credits').innerText = credits;
      document.getElementById('labor').innerText = labor;
      document.getElementById('handel').innerText = handel;
      document.getElementById('solar').innerText = solar;
      document.getElementById('platform').innerText = platform;
      document.getElementById('buySolar').title = `Solarpanel kaufen (${priceSolar} Credits)`;
      document.getElementById('buyLabor').title = `Labor kaufen (${priceLabor} Credits)`;
      document.getElementById('buyHandel').title = `Handelsstation kaufen (${priceHandel} Energie)`;
    }

    // Drei Kaufkreise als Three.js Circle Meshes
    const buyCircles = [];

    // Szene, Kamera, Renderer
    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#87ceeb'); // blauer Himmel

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(20, 25, 35);
    camera.lookAt(0, 0, 0);

    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Controls
    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;

    // Licht
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);

    const ambientLight = new THREE.AmbientLight(0x404040);
    scene.add(ambientLight);

    // Große Plattform (100x100) grau
    const groundGroup = new THREE.Group();

    const platformGeometry = new THREE.PlaneGeometry(100, 100);
    const platformMaterial = new THREE.MeshStandardMaterial({ color: '#808080' });
    const platformMesh = new THREE.Mesh(platformGeometry, platformMaterial);
    platformMesh.rotation.x = -Math.PI / 2;
    platformMesh.position.y = 0;
    groundGroup.add(platformMesh);

    // Grasrand 10m breit an den Rändern
    const grassWidth = 10;
    const grassColor = '#228B22';

    // Gras am Rand als vier Streifen
    const grassFront = new THREE.Mesh(
      new THREE.PlaneGeometry(100, grassWidth),
      new THREE.MeshStandardMaterial({ color: grassColor })
    );
    grassFront.rotation.x = -Math.PI / 2;
    grassFront.position.set(0, 0.01, 55);
    groundGroup.add(grassFront);

    const grassBack = new THREE.Mesh(
      new THREE.PlaneGeometry(100, grassWidth),
      new THREE.MeshStandardMaterial({ color: grassColor })
    );
    grassBack.rotation.x = -Math.PI / 2;
    grassBack.position.set(0, 0.01, -55);
    groundGroup.add(grassBack);

    const grassLeft = new THREE.Mesh(
      new THREE.PlaneGeometry(grassWidth, 80),
      new THREE.MeshStandardMaterial({ color: grassColor })
    );
    grassLeft.rotation.x = -Math.PI / 2;
    grassLeft.position.set(-55, 0.01, 0);
    groundGroup.add(grassLeft);

    const grassRight = new THREE.Mesh(
      new THREE.PlaneGeometry(grassWidth, 80),
      new THREE.MeshStandardMaterial({ color: grassColor })
    );
    grassRight.rotation.x = -Math.PI / 2;
    grassRight.position.set(55, 0.01, 0);
    groundGroup.add(grassRight);

    scene.add(groundGroup);

    // GridHelper zum besseren Überblick (100x100)
    const gridHelper = new THREE.GridHelper(100, 20, 0x444444, 0x888888);
    scene.add(gridHelper);

    // Spieler als Würfel
    const playerGeometry = new THREE.BoxGeometry(1, 2, 1);
    const playerMaterial = new THREE.MeshStandardMaterial({ color: '#ff69b4' });
    const player = new THREE.Mesh(playerGeometry, playerMaterial);
    player.position.set(0, 1, 0);
    scene.add(player);

    // Kaufkreise zum Kaufen der Module (Solar, Labor, Handel)
    function createBuyCircle(x, z, color, type) {
      const geometry = new THREE.CircleGeometry(1, 32);
      const material = new THREE.MeshBasicMaterial({ color: color, opacity: 0.5, transparent: true });
      const circle = new THREE.Mesh(geometry, material);
      circle.rotation.x = -Math.PI/2;
      circle.position.set(x, 0.05, z);
      circle.userData.type = type;
      scene.add(circle);
      buyCircles.push(circle);
    }

    createBuyCircle(5, 5, '#00ffff', 'solar');
    createBuyCircle(8, 5, '#ffff00', 'labor');
    createBuyCircle(11, 5, '#cc00ff', 'handel');

    // Preise erhöhen sich nach jedem Kauf (Solar/Labor +100 Credits, Handel +250 Energie)
    function buy(type) {
      if (type === 'solar') {
        if (credits >= priceSolar) {
          credits -= priceSolar;
          solar++;
          priceSolar += 100;
          updateUI();
          playSound();
        } else alert('Nicht genug Credits für Solarpanel!');
      } else if (type === 'labor') {
        if (credits >= priceLabor) {
          credits -= priceLabor;
          labor++;
          priceLabor += 100;
          updateUI();
          playSound();
        } else alert('Nicht genug Credits für Labor!');
      } else if (type === 'handel') {
        if (energy >= priceHandel) {
          energy -= priceHandel;
          handel++;
          priceHandel += 250;
          updateUI();
          playSound();
        } else alert('Nicht genug Energie für Handelsstation!');
      }
    }

    // Überprüfe Abstand Spieler zu Kaufkreisen, kaufe wenn man drüber läuft
    function checkBuyZones() {
      for (const circle of buyCircles) {
        const dist = player.position.distanceTo(circle.position);
        if (dist < 1.5) {
          buy(circle.userData.type);
        }
      }
    }

    // Steuerung
    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    let velocityY = 0;
    let isOnGround = true;

    function animate() {
      // Bewegung
      if (keys['w']) player.position.z -= 0.1;
      if (keys['s']) player.position.z += 0.1;
      if (keys['a']) player.position.x -= 0.1;
      if (keys['d']) player.position.x += 0.1;

      // Springen mit Leertaste
      if (keys[' '] && isOnGround) {
        velocityY = 0.15;
        isOnGround = false;
      }

      // Gravitation
      player.position.y += velocityY;
      velocityY -= 0.01;
      if (player.position.y <= 1) {
        player.position.y = 1;
        velocityY = 0;
        isOnGround = true;
      }

      checkBuyZones();

      controls.update();
      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }

    animate();

    // Einnahmen: pro Sekunde (steigend mit Anzahl)
    setInterval(() => {
      credits += solar * 100 + labor * 100 + handel * 100; 
      energy += solar * 20; 
      updateUI();
    }, 1000);

    function playSound() {
      const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
      audio.play();
    }

    updateUI();

    // Fensteranpassung
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
