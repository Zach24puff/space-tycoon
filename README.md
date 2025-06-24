<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>3D Space Station Tycoon - Landschaft mit Straße & Häusern</title>
  <style>
    body, html {
      margin: 0; padding: 0; overflow: hidden;
      background: linear-gradient(to top, #87ceeb, #cceeff); /* Heller Himmel-Gradient */
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
      width: 220px;
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

  <!-- Three.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

  <script>
    // Szene, Kamera, Renderer
    const scene = new THREE.Scene();

    // Himmel-Farbe als Hintergrund
    scene.background = new THREE.Color(0x87ceeb); // Himmelblau

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 15);

    const renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    // Licht
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    scene.add(ambientLight);

    const sunLight = new THREE.DirectionalLight(0xffffff, 1);
    sunLight.position.set(10, 20, 10);
    sunLight.castShadow = true;
    sunLight.shadow.mapSize.width = 2048;
    sunLight.shadow.mapSize.height = 2048;
    sunLight.shadow.camera.near = 0.5;
    sunLight.shadow.camera.far = 50;
    sunLight.shadow.camera.left = -30;
    sunLight.shadow.camera.right = 30;
    sunLight.shadow.camera.top = 30;
    sunLight.shadow.camera.bottom = -30;
    scene.add(sunLight);

    // Boden (Gras)
    const grassGeometry = new THREE.PlaneGeometry(100, 100);
    const grassMaterial = new THREE.MeshStandardMaterial({color: 0x3a923a});
    const grass = new THREE.Mesh(grassGeometry, grassMaterial);
    grass.rotation.x = -Math.PI / 2;
    grass.receiveShadow = true;
    scene.add(grass);

    // Straße - lange graue Ebene in der Mitte
    const roadGeometry = new THREE.PlaneGeometry(10, 100);
    const roadMaterial = new THREE.MeshStandardMaterial({color: 0x444444});
    const road = new THREE.Mesh(roadGeometry, roadMaterial);
    road.position.y = 0.01; // leicht über Gras, damit nicht z-fighting
    road.rotation.x = -Math.PI / 2;
    road.receiveShadow = true;
    scene.add(road);

    // Straßen-Markierungen (streifen)
    const stripeGeometry = new THREE.PlaneGeometry(1, 5);
    const stripeMaterial = new THREE.MeshStandardMaterial({color: 0xffffff});
    for(let i=-40; i<40; i+=10) {
      const stripe = new THREE.Mesh(stripeGeometry, stripeMaterial);
      stripe.position.set(0, 0.02, i);
      stripe.rotation.x = -Math.PI / 2;
      stripe.receiveShadow = true;
      scene.add(stripe);
    }

    // Häuser links und rechts der Straße
    const houseGroup = new THREE.Group();

    function createHouse() {
      const house = new THREE.Group();

      // Haus-Körper
      const bodyGeo = new THREE.BoxGeometry(3, 2, 3);
      const bodyMat = new THREE.MeshStandardMaterial({color: 0xffcc66});
      const body = new THREE.Mesh(bodyGeo, bodyMat);
      body.castShadow = true;
      body.receiveShadow = true;
      body.position.y = 1;
      house.add(body);

      // Dach (Pyramide)
      const roofGeo = new THREE.ConeGeometry(2.2, 1, 4);
      const roofMat = new THREE.MeshStandardMaterial({color: 0x8b0000});
      const roof = new THREE.Mesh(roofGeo, roofMat);
      roof.position.y = 2.5;
      roof.rotation.y = Math.PI / 4;
      roof.castShadow = true;
      house.add(roof);

      return house;
    }

    // Häuser links platzieren (-10 X Achse)
    for(let i= -40; i < 50; i += 12) {
      let h = createHouse();
      h.position.set(-10, 0, i);
      houseGroup.add(h);
    }

    // Häuser rechts platzieren (+10 X Achse)
    for(let i= -40; i < 50; i += 12) {
      let h = createHouse();
      h.position.set(10, 0, i+6); // leicht versetzt
      houseGroup.add(h);
    }

    scene.add(houseGroup);

    // Spieler: Box mit Kantenlinien
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

      // Bewegung mit Steuerung:
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
      if (keys[' '] && canJump) {
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
      camera.position.z = player.position.z + 15;
      camera.position.y = player.position.y + 7;
      camera.lookAt(player.position);

      renderer.render(scene, camera);
    }

    animate();
  </script>
</body>
</html>
