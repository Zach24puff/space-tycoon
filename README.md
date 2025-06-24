<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
  <title>Space Station Tycoon 3D</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      touch-action: none;
      -webkit-user-select: none;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
      background-color: #000;
      height: 100vh;
      width: 100vw;
    }

    #ui {
      position: fixed;
      top: 10px;
      left: 10px;
      background: rgba(0, 0, 0, 0.75);
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ffcc;
      z-index: 10;
      color: white;
      font-family: Arial, sans-serif;
      width: 90vw;
      max-width: 300px;
      font-size: 1.2em;
    }

    .circle-trigger {
      position: absolute;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: rgba(0, 255, 204, 0.6);
      box-shadow: 0 0 10px rgba(0, 255, 204, 0.9);
      pointer-events: none;
    }

    #languageSelect {
      position: fixed;
      top: 10px;
      right: 10px;
      z-index: 11;
      font-size: 1em;
      background: rgba(0, 0, 0, 0.7);
      color: white;
      border: 1px solid #00ffcc;
      border-radius: 5px;
      padding: 5px;
    }

    /* Steuerungsbuttons für Handy */
    #touchControls {
      position: fixed;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      width: 260px;
      height: 100px;
      z-index: 20;
      user-select: none;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 15px;
      pointer-events: auto;
    }

    .btn {
      background: rgba(0, 255, 204, 0.5);
      border-radius: 50%;
      width: 60px;
      height: 60px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 2em;
      color: #003333;
      user-select: none;
      touch-action: manipulation;
      box-shadow: 0 0 10px rgba(0, 255, 204, 0.8);
    }
    .btn:active {
      background: rgba(0, 255, 204, 0.8);
      box-shadow: 0 0 15px rgba(0, 255, 204, 1);
    }

    #jumpBtn {
      width: 80px;
      height: 80px;
      font-size: 1.8em;
      background: rgba(255, 165, 0, 0.6);
      color: #330000;
      box-shadow: 0 0 15px rgba(255, 165, 0, 0.9);
    }
  </style>
</head>
<body>
  <select id="languageSelect">
    <option value="de">Deutsch</option>
    <option value="en">English</option>
    <option value="fr">Français</option>
    <option value="es">Español</option>
  </select>

  <div id="ui">
    <h1 id="title" style="font-size: 1.5em;">🚀 Space Station Tycoon</h1>
    <div>💰 <span id="labelCredits">Credits</span>: <span id="credits">200</span></div>
    <div>🔬 <span id="labelLabor">Labor</span>: <span id="labor">0</span></div>
    <div>🏪 <span id="labelHandel">Handelsstation</span>: <span id="handel">0</span></div>
    <div>🔋 <span id="labelSolar">Solarpanels</span>: <span id="solar">0</span></div>
    <div>🪐 <span id="labelPlatform">Plattformen</span>: <span id="platform">1</span> (100x100 gehört dir)</div>
  </div>

  <!-- Touch Steuerung Buttons -->
  <div id="touchControls">
    <div class="btn" id="leftBtn">&#8592;</div>
    <div class="btn" id="upBtn">&#8593;</div>
    <div class="btn" id="downBtn">&#8595;</div>
    <div class="btn" id="rightBtn">&#8594;</div>
    <div class="btn" id="jumpBtn">⬆︎</div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
  <script>
    // --- Spiel-Variablen ---
    const state = {
      credits: 200,
      labor: 0,
      handel: 0,
      solar: 0,
      platform: 1
    };

    // Preise (steigen pro Kauf)
    let priceSolar = 100;
    let priceLabor = 100;
    let priceHandel = 250;
    let priceHouseObj = 50; // Hausobjekt
    let priceSolarInc = 100; // Solarpanel Preissteigerung pro Kauf
    let priceHouseObjInc = 50; // Hausobjekt Preissteigerung pro Kauf

    // UI Felder
    const uiFields = ['credits', 'labor', 'handel', 'solar', 'platform'];

    // Übersetzungen
    const translations = {
      de: {
        title: '🚀 Space Station Tycoon',
        credits: 'Credits',
        labor: 'Labor',
        handel: 'Handelsstation',
        solar: 'Solarpanels',
        platform: 'Plattformen'
      },
      en: {
        title: '🚀 Space Station Tycoon',
        credits: 'Credits',
        labor: 'Lab',
        handel: 'Trade Hub',
        solar: 'Solar Panels',
        platform: 'Platforms'
      },
      fr: {
        title: '🚀 Tycoon de Station Spatiale',
        credits: 'Crédits',
        labor: 'Laboratoire',
        handel: 'Centre Commercial',
        solar: 'Panneaux Solaires',
        platform: 'Plates-formes'
      },
      es: {
        title: '🚀 Magnate de la Estación Espacial',
        credits: 'Créditos',
        labor: 'Laboratorio',
        handel: 'Centro Comercial',
        solar: 'Paneles Solares',
        platform: 'Plataformas'
      }
    };

    // --- Three.js Setup ---
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb); // Blauer Himmel

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

    const ambLight = new THREE.AmbientLight(0x404040);
    scene.add(ambLight);

    // Boden & große Plattform
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

    // Spieler als Würfel
    const playerGeo = new THREE.BoxGeometry(1,2,1);
    const playerMat = new THREE.MeshStandardMaterial({color: 0xff5500});
    const player = new THREE.Mesh(playerGeo, playerMat);
    player.position.y = 1;
    scene.add(player);

    // Häuser
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

    createHouse(10, 10);
    createHouse(-10, -10);
    createHouse(20, -20);

    // Bäume
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

    // Kaufkreise
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

    // Steuerung & Bewegung
    const keys = {
      w: false,
      a: false,
      s: false,
      d: false,
      ' ': false
    };

    // Keyboard Events
    document.addEventListener('keydown', e => {
      if(e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = true;
    });
    document.addEventListener('keyup', e => {
      if(e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = false;
    });

    // Touch Buttons Events
    function setKey(key, value) {
      if(key in keys) keys[key] = value;
    }

    document.getElementById('leftBtn').addEventListener('touchstart', () => setKey('a', true));
    document.getElementById('leftBtn').addEventListener('touchend', () => setKey('a', false));

    document.getElementById('rightBtn').addEventListener('touchstart', () => setKey('d', true));
    document.getElementById('rightBtn').addEventListener('touchend', () => setKey('d', false));

    document.getElementById('upBtn').addEventListener('touchstart', () => setKey('w', true));
    document.getElementById('upBtn').addEventListener('touchend', () => setKey('w', false));

    document.getElementById('downBtn').addEventListener('touchstart', () => setKey('s', true));
    document.getElementById('downBtn').addEventListener('touchend', () => setKey('s', false));

    document.getElementById('jumpBtn').addEventListener('touchstart', () => setKey(' ', true));
    document.getElementById('jumpBtn').addEventListener('touchend', () => setKey(' ', false));

    let velocityY = 0;
    let isOnGround = true;

    function movePlayer() {
      const speed = 0.3;
      if(keys.w) player.position.z -= speed;
      if(keys.s) player.position.z += speed;
      if(keys.a) player.position.x -= speed;
      if(keys.d) player.position.x += speed;

      if(keys[' '] && isOnGround) {
        velocityY = 0.15;
        isOnGround = false;
      }

      player.position.y += velocityY;
      velocityY -= 0.01;

      if(player.position.y <= 1) {
        player.position.y = 1;
        velocityY = 0;
        isOnGround = true;
      }

      if(player.position.x > 49.5) player.position.x = 49.5;
      if(player.position.x < -49.5) player.position.x = -49.5;
      if(player.position.z > 49.5) player.position.z = 49.5;
      if(player.position.z < -49.5) player.position.z = -49.5;
    }

    // Kaufen per Überlaufen
    function checkBuy() {
      for(let i=0; i < buyTriggers.length; i++) {
        const bt = buyTriggers[i];
        if(bt.bought) continue;

        const dist = player.position.distanceTo(bt.mesh.position);
        if(dist < 1) {
          if(bt.type === 'houseObj') {
            if(state.credits >= priceHouseObj) {
              state.credits -= priceHouseObj;
              bt.bought = true;
              houseObjs[bt.objIndex].bought = true;
              houseObjs[bt.objIndex].mesh.material.opacity = 1;
              houseObjs[bt.objIndex].mesh.material.transparent = false;
              priceHouseObj += priceHouseObjInc;
              updateUI();
              playSound();
            }
          } else if(bt.type === 'solar') {
            if(state.credits >= priceSolar) {
              state.credits -= priceSolar;
              state.solar++;
              bt.bought = true;
              priceSolar += priceSolarInc;
              updateUI();
              playSound();
            }
          } else if(bt.type === 'labor') {
            if(state.credits >= priceLabor) {
              state.credits -= priceLabor;
              state.labor++;
              bt.bought = true;
              updateUI();
              playSound();
            }
          } else if(bt.type === 'handel') {
            if(state.credits >= priceHandel) {
              state.credits -= priceHandel;
              state.handel++;
              bt.bought = true;
              updateUI();
              playSound();
            }
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
      state.credits += state.solar * 100 + state.labor * 50 + state.handel * 150;
      updateUI();
    }, 1000);

    // UI Update
    function updateUI() {
      uiFields.forEach(id => {
        const el = document.getElementById(id);
        if(el) el.innerText = state[id];
      });
    }

    // Sprache ändern
    function setLanguage(lang) {
      const t = translations[lang] || translations['de'];
      document.getElementById('title').innerText = t.title;
      document.getElementById('labelCredits').innerText = t.credits;
      document.getElementById('labelLabor').innerText = t.labor;
      document.getElementById('labelHandel').innerText = t.handel;
      document.getElementById('labelSolar').innerText = t.solar;
      document.getElementById('labelPlatform').innerText = t.platform;
    }

    document.getElementById('languageSelect').addEventListener('change', (e) => {
      setLanguage(e.target.value);
    });

    // Animation Loop
    function animate() {
      requestAnimationFrame(animate);
      movePlayer();
      checkBuy();
      renderer.render(scene, camera);
    }
    animate();

    // Fenstergröße anpassen
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    updateUI();
    setLanguage('de');
  </script>
</body>
</html>
