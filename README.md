<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>3D Space Station Tycoon</title>
  <style>
    body, html {
      margin: 0; padding: 0; overflow: hidden;
      background: #000;
      font-family: Arial, sans-serif;
      color: white;
      user-select: none;
    }
    #ui {
      position: absolute;
      top: 10px; left: 10px;
      background: rgba(0,0,0,0.75);
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ffcc;
      z-index: 10;
      width: 180px;
    }
    #ui h1 {
      color: #00ffcc;
      margin: 0 0 10px 0;
      font-size: 20px;
    }
    #buyButton3d {
      position: absolute;
      bottom: 30px;
      right: 30px;
      width: 60px; height: 60px;
      border-radius: 50%;
      background: #00ffcc;
      box-shadow: 0 0 15px #00ffcc;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      font-size: 32px;
      color: black;
      user-select: none;
      z-index: 20;
      transition: background 0.3s;
    }
    #buyButton3d:hover {
      background: #00aaaa;
      box-shadow: 0 0 20px #00aaaa;
    }
  </style>
</head>
<body>
  <div id="ui">
    <h1>🚀 3D Space Station Tycoon</h1>
    <div>💰 Credits: <span id="credits">200</span></div>
    <div>Module: <span id="moduleCount">0</span></div>
  </div>

  <div id="buyButton3d" title="Kaufe Modul (+50 Credits/sec)">+</div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/PointerLockControls.js"></script>

  <script>
    // Szene, Kamera, Renderer
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x000011);

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);

    const renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Licht
    const ambientLight = new THREE.AmbientLight(0x404040);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0x00ffcc, 1);
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight);

    // Boden (Plattform)
    const floorGeometry = new THREE.PlaneGeometry(100, 100);
    const floorMaterial = new THREE.MeshStandardMaterial({color: 0x222244});
    const floor = new THREE.Mesh(floorGeometry, floorMaterial);
    floor.rotation.x = -Math.PI / 2;
    floor.position.y = 0;
    scene.add(floor);

    // Spieler als Box
    const playerGeometry = new THREE.BoxGeometry(1, 2, 1);
    const playerMaterial = new THREE.MeshStandardMaterial({color: 0x00ffcc});
    const player = new THREE.Mesh(playerGeometry, playerMaterial);
    player.position.set(0, 1, 0);
    scene.add(player);

    // Controls
    const keys = {};
    document.addEventListener('keydown', e => { keys[e.key.toLowerCase()] = true; });
    document.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

    // Spieler Variablen
    let velocity = new THREE.Vector3();
    let direction = new THREE.Vector3();
    let canJump = false;

    // Physik Parameter
    const GRAVITY = 0.5;
    const MOVE_SPEED = 0.1;
    const JUMP_SPEED = 0.2;

    // Spielzustand
    let credits = 200;
    let modules = 0;
    let creditsPerSecond = 10;

    // UI update
    function updateUI() {
      document.getElementById('credits').innerText = credits.toFixed(0);
      document.getElementById('moduleCount').innerText = modules;
    }
    updateUI();

    // Kaufen Button
    const buyButton = document.getElementById('buyButton3d');
    buyButton.addEventListener('click', () => {
      if (credits >= 100) {
        credits -= 100;
        modules++;
        creditsPerSecond = 10 + (modules - 1) * 50;
        updateUI();
      }
    });

    // Credits jede Sekunde erhöhen
    setInterval(() => {
      credits += creditsPerSecond;
      updateUI();
    }, 1000);

    // Resize Handler
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // Simple Kollisions-Check Boden
    function checkGround() {
      if (player.position.y <= 1) {
        player.position.y = 1;
        velocity.y = 0;
        canJump = true;
      } else {
        canJump = false;
      }
    }

    // Animation Loop
    function animate() {
      requestAnimationFrame(animate);

      // Bewegung
      direction.set(0, 0, 0);
      if (keys['w']) direction.z -= 1;
      if (keys['s']) direction.z += 1;
      if (keys['a']) direction.x -= 1;
      if (keys['d']) direction.x += 1;

      direction.normalize();

      // Bewegungsgeschwindigkeit
      player.position.x += direction.x * MOVE_SPEED;
      player.position.z += direction.z * MOVE_SPEED;

      // Schwerkraft
      velocity.y -= GRAVITY * 0.02;
      player.position.y += velocity.y;

      // Bodencheck
      checkGround();

      // Springen
      if (keys['w'] && canJump) {
        velocity.y = JUMP_SPEED;
        canJump = false;
      }

      // Kamera folgt Spieler
      camera.position.x = player.position.x;
      camera.position.z = player.position.z + 10;
      camera.position.y = player.position.y + 5;
      camera.lookAt(player.position);

      renderer.render(scene, camera);
    }

    animate();
  </script>
</body>
</html>
