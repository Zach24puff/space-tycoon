<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>3D Space Station Tycoon - mit Sound & Effekten</title>
  <style>
    /* (Style unverändert, siehe vorheriges Beispiel) */
    body, html {
      margin: 0; padding: 0; overflow: hidden;
      background: #000011;
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
      user-select: none;
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

  <!-- Three.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

  <script>
    // Szene, Kamera, Renderer
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x000011);

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);

    const renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    // Licht
    const ambientLight = new THREE.AmbientLight(0x404040, 1.2);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0x00ffcc, 1);
    dirLight.position.set(5, 10, 7);
    dirLight.castShadow = true;
    dirLight.shadow.mapSize.width = 1024;
    dirLight.shadow.mapSize.height = 1024;
    dirLight.shadow.camera.near = 0.5;
    dirLight.shadow.camera.far = 50;
    dirLight.shadow.camera.left = -20;
    dirLight.shadow.camera.right = 20;
    dirLight.shadow.camera.top = 20;
    dirLight.shadow.camera.bottom = -20;
    scene.add(dirLight);

    // Boden (Plattform)
    const floorGeometry = new THREE.PlaneGeometry(100, 100);
    const floorMaterial = new THREE.MeshStandardMaterial({color: 0x222244});
    const floor = new THREE.Mesh(floorGeometry, floorMaterial);
    floor.rotation.x = -Math.PI / 2;
    floor.position.y = 0;
    floor.receiveShadow = true;
    scene.add(floor);

    // Spieler: Box + Edges (leuchtende Kanten)
    const playerGeometry = new THREE.BoxGeometry(1, 2, 1);
    const playerMaterial = new THREE.MeshStandardMaterial({color: 0x00ffcc});
    const player = new THREE.Mesh(playerGeometry, playerMaterial);
    player.position.set(0, 1, 0);
    player.castShadow = true;
    scene.add(player);

    const edges = new THREE.EdgesGeometry(playerGeometry);
    const line = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({color: 0x00ffff}));
    player.add(line);

    // Steuerung
    const keys = {};
    window.addEventListener('keydown', e => { keys[e.key.toLowerCase()] = true; });
    window.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

    // Physik Variablen
    let velocityY = 0;
    const gravity = 0.5;
    const moveSpeed = 0.12;
    const jumpSpeed = 0.22;
    let canJump = false;

    // Spielvariablen
    let credits = 200;
    let modules = 0;
    let creditsPerSecond = 10;

    // UI aktualisieren
    function updateUI() {
      document.getElementById('credits').innerText = credits.toFixed(0);
      document.getElementById('moduleCount').innerText = modules;
    }
    updateUI();

    // Sound-Setup
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    const audioCtx = new AudioContext();

    function playBeep(frequency, duration = 0.1) {
      if(audioCtx.state === 'suspended') audioCtx.resume();
      const oscillator = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();

      oscillator.connect(gainNode);
      gainNode.connect(audioCtx.destination);

      oscillator.type = 'square';
      oscillator.frequency.value = frequency;
      gainNode.gain.setValueAtTime(0.2, audioCtx.currentTime);

      oscillator.start();
      oscillator.stop(audioCtx.currentTime + duration);
    }

    // Partikel-System für Kauf-Animation
    const particles = [];
    const particleGeometry = new THREE.SphereGeometry(0.05, 8, 8);
    const particleMaterial = new THREE.MeshBasicMaterial({color: 0x00ffcc});

    function spawnParticles(position) {
      for(let i=0; i<15; i++) {
        const p = new THREE.Mesh(particleGeometry, particleMaterial);
        p.position.copy(position);
        p.velocity = new THREE.Vector3(
          (Math.random()-0.5)*0.5,
          Math.random()*0.7,
          (Math.random()-0.5)*0.5
        );
        particles.push(p);
        scene.add(p);
      }
    }

    // Kauf-Button
    const buyButton = document.getElementById('buyButton3d');
    buyButton.addEventListener('click', () => {
      if (credits >= 100) {
        credits -= 100;
        modules++;
        creditsPerSecond = 10 + (modules - 1) * 50;
        updateUI();
        playBeep(600, 0.15);
        spawnParticles(player.position.clone());
      } else {
        playBeep(200, 0.1); // Fehler-Ton bei zu wenig Credits
      }
    });

    // Credits pro Sekunde erhöhen
    setInterval(() => {
      credits += creditsPerSecond;
      updateUI();
    }, 1000);

    // Fenstergröße anpassen
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // Boden Check
    function checkGround() {
      if (player.position.y <= 1) {
        player.position.y = 1;
        velocityY = 0;
        canJump = true;
      } else {
        canJump = false;
      }
    }

    // Animation Loop
    function animate() {
      requestAnimationFrame(animate);

      // Bewegung mit geänderter Steuerung:
      // W = vorwärts (negative Z)
      // S = rückwärts (positive Z)
      // A = links (negative X)
      // D = rechts (positive X)
      let directionX = 0;
      let directionZ = 0;
      if (keys['a']) directionX -= 1;
      if (keys['d']) directionX += 1;
      if (keys['w']) directionZ -= 1;
      if (keys['s']) directionZ += 1;

      // Bewegung normalisieren
      let length = Math.hypot(directionX, directionZ);
      if (length > 0) {
        directionX /= length;
        directionZ /= length;
      }

      // Spieler bewegen
      player.position.x += directionX * moveSpeed;
      player.position.z += directionZ * moveSpeed;

      // Schwerkraft anwenden
      velocityY -= gravity * 0.02;
      player.position.y += velocityY;

      // Sprung nur auf Leertaste
      if (keys[' ' ] && canJump) {
        velocityY = jumpSpeed;
        canJump = false;
        playBeep(800, 0.12);
      }

      // Bodencheck
      checkGround();

      // Partikel-Update
      for(let i = particles.length - 1; i >= 0; i--) {
        let p = particles[i];
        p.position.add(p.velocity);
        p.velocity.y -= 0.02;
        if(p.position.y < 0) {
          scene.remove(p);
          particles.splice(i, 1);
        }
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
