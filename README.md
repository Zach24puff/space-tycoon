<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Space Station Tycoon 3D</title>
  <style>
    body {
      margin: 0; overflow: hidden; font-family: Arial, sans-serif;
    }
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
      font-size: 28px;
      color: black;
      font-weight: bold;
      cursor: pointer;
      box-shadow: 0 0 10px #00ffcc;
      transition: 0.2s;
      user-select: none;
    }
    .circle-button:hover {
      background: #00aaaa;
      box-shadow: 0 0 15px #00aaaa;
    }
    #buySolar { bottom: 30px; right: 30px; }
    #buyLabor { bottom: 100px; right: 30px; }
    #buyHandel { bottom: 170px; right: 30px; }
  </style>
</head>
<body>
  <div id="ui">
    <h1>🚀 Space Station Tycoon</h1>
    <div>💰 Credits: <span id="credits">200</span></div>
    <div>🔬 Labor: <span id="labor">0</span></div>
    <div>🏪 Handelsstation: <span id="handel">0</span></div>
    <div>🔋 Solarpanels: <span id="solar">0</span></div>
  </div>

  <div id="buySolar" class="circle-button" title="Solarpanel kaufen">🔋</div>
  <div id="buyLabor" class="circle-button" title="Labor kaufen">🧪</div>
  <div id="buyHandel" class="circle-button" title="Handelsstation kaufen">🏪</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
  <script>
    // Ressourcen & Zähler
    let credits = 200;
    let energy = 1000;
    let solar = 0;
    let labor = 0;
    let handel = 0;

    function updateUI() {
      document.getElementById('credits').innerText = credits;
      document.getElementById('labor').innerText = labor;
      document.getElementById('handel').innerText = handel;
      document.getElementById('solar').innerText = solar;
    }

    // 3D Szene Setup
    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#87ceeb'); // blauer Himmel

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 20, 30);

    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;
    controls.minDistance = 10;
    controls.maxDistance = 80;
    controls.maxPolarAngle = Math.PI / 2.5;

    // Licht
    const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight.position.set(30, 50, 20);
    scene.add(directionalLight);
    scene.add(new THREE.AmbientLight(0x404040));

    // Große Plattform (100x100) - dir gehört sie sofort
    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(100, 100),
      new THREE.MeshStandardMaterial({ color: '#556b2f' }) // dunkles olivgrün, Gras
    );
    ground.rotation.x = -Math.PI / 2;
    ground.receiveShadow = true;
    scene.add(ground);

    // Spieler (Würfel)
    const player = new THREE.Mesh(
      new THREE.BoxGeometry(1, 2, 1),
      new THREE.MeshStandardMaterial({ color: '#ff69b4' }) // pink
    );
    player.position.set(0, 1, 0);
    scene.add(player);

    // Steuerung
    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    let velocityY = 0;
    let isOnGround = true;

    // Kaufkreise (interaktive Kreise auf dem Boden zum Kaufen)
    const buyCircles = [];

    // Modulpositionen (Kaufkreise) - diese kreise sind fest positioniert
    const modulePositions = {
      solar: new THREE.Vector3(10, 0.1, 10),
      labor: new THREE.Vector3(15, 0.1, 10),
      handel: new THREE.Vector3(20, 0.1, 10),
    };

    // Erzeuge Kaufkreise
    function createBuyCircle(type, position) {
      const circleGeo = new THREE.CircleGeometry(1, 32);
      const colors = { solar: '#00ffff', labor: '#ffcc00', handel: '#cc00ff' };
      const material = new THREE.MeshStandardMaterial({
        color: colors[type],
        transparent: true,
        opacity: 0.6,
        side: THREE.DoubleSide
      });
      const circle = new THREE.Mesh(circleGeo, material);
      circle.rotation.x = -Math.PI/2;
      circle.position.copy(position);
      circle.userData = { type };
      scene.add(circle);
      buyCircles.push(circle);
    }

    createBuyCircle('solar', modulePositions.solar);
    createBuyCircle('labor', modulePositions.labor);
    createBuyCircle('handel', modulePositions.handel);

    // Module speichern (für Darstellung)
    const modulesInScene = { solar: [], labor: [], handel: [] };

    // Module hinzufügen (sichtbar)
    function addModule(type) {
      let mesh;
      switch(type) {
        case 'solar':
          mesh = new THREE.Mesh(
            new THREE.CylinderGeometry(0.5, 0.5, 2),
            new THREE.MeshStandardMaterial({ color: '#00ffff' })
          );
          break;
        case 'labor':
          mesh = new THREE.Mesh(
            new THREE.BoxGeometry(2, 2, 2),
            new THREE.MeshStandardMaterial({ color: '#ffcc00' })
          );
          break;
        case 'handel':
          mesh = new THREE.Mesh(
            new THREE.BoxGeometry(2, 2, 2),
            new THREE.MeshStandardMaterial({ color: '#cc00ff' })
          );
          break;
      }
      if (!mesh) return;
      // Position zufällig in einem Bereich nahe Plattform
      mesh.position.set(
        (Math.random() - 0.5) * 50,
        1,
        (Math.random() - 0.5) * 50
      );
      scene.add(mesh);
      modulesInScene[type].push(mesh);
    }

    // Sound beim Kauf
    function playSound() {
      const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
      audio.play();
    }

    // Credits pro Sekunde Berechnung (wächst pro Kauf um 100)
    function creditsPerSecond() {
      return solar * 10 + labor * 5 + handel * 20;
    }

    // Player Bewegung und Kauf-Check
    function animate() {
      requestAnimationFrame(animate);

      // Bewegung
      let speed = 0.2;
      if (keys['w']) player.position.z -= speed;
      if (keys['s']) player.position.z += speed;
      if (keys['a']) player.position.x -= speed;
      if (keys['d']) player.position.x += speed;

      // Springen (Leertaste)
      if (keys[' '] && isOnGround) {
        velocityY = 0.15;
        isOnGround = false;
      }
      velocityY -= 0.01;
      player.position.y += velocityY;
      if (player.position.y <= 1) {
        player.position.y = 1;
        velocityY = 0;
        isOnGround = true;
      }

      // Kaufkreis Check
      for (const circle of buyCircles) {
        const dist = player.position.distanceTo(circle.position);
        if (dist < 1.2) { // "drüber laufen"
          tryBuy(circle.userData.type);
        }
      }

      controls.target.copy(player.position);
      controls.update();

      renderer.render(scene, camera);
    }

    // Kauf mit Preissteigerung pro Kauf (Preis steigt um 100 Credits pro Kauf)
    const basePrices = { solar: 100, labor: 100, handel: 250 };
    let currentPrices = { solar: 100, labor: 100, handel: 250 };

    function tryBuy(type) {
      if (type === 'solar' && credits >= currentPrices.solar) {
        credits -= currentPrices.solar;
        solar++;
        currentPrices.solar += 100;
        addModule('solar');
        updateUI();
        playSound();
      } else if (type === 'labor' && credits >= currentPrices.labor) {
        credits -= currentPrices.labor;
        labor++;
        currentPrices.labor += 100;
        addModule('labor');
        updateUI();
        playSound();
      } else if (type === 'handel' && energy >= currentPrices.handel) {
        energy -= currentPrices.handel;
        handel++;
        currentPrices.handel += 50; // für handel etwas langsamer Preissteigerung
        addModule('handel');
        updateUI();
        playSound();
      }
    }

    // Alle 1 Sekunde: Credits & Energie erhöhen
    setInterval(() => {
      credits += creditsPerSecond();
      energy += solar * 20; // Solarpanels produzieren Energie
      updateUI();
    }, 1000);

    // Fenster Resize
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    updateUI();
    animate();
  </script>
</body>
</html>
