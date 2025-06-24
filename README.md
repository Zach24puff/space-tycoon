<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Space Station Tycoon 3D - Korrigiert</title>
<style>
  body {
    margin: 0; overflow: hidden;
    font-family: Arial, sans-serif;
    background: #87ceeb; /* blauer Himmel */
    color: white;
  }
  #ui {
    position: absolute;
    top: 10px; left: 10px;
    background: rgba(0,0,0,0.75);
    padding: 10px;
    border-radius: 8px;
    box-shadow: 0 0 10px #00ffcc;
    z-index: 10;
    font-size: 14px;
    line-height: 1.4;
    user-select: none;
    max-width: 300px;
  }
  #ui table {
    border-collapse: collapse;
    width: 100%;
  }
  #ui th, #ui td {
    border: 1px solid #00ffcc;
    padding: 6px 10px;
    text-align: left;
  }
  #ui th {
    background-color: #00ffcc;
    color: #000;
  }
  #buyHelp {
    margin-top: 8px;
    font-size: 12px;
    color: #00ffcc;
  }
</style>
</head>
<body>
  <div id="ui">
    <table>
      <thead><tr><th>Ressource</th><th>Anzahl</th><th>Preis (nächst)</th></tr></thead>
      <tbody>
        <tr><td>💰 Credits</td><td id="credits">200</td><td>-</td></tr>
        <tr><td>🔬 Labor</td><td id="labor">0</td><td>100</td></tr>
        <tr><td>🏪 Handelsstation</td><td id="handel">0</td><td>250 Energie</td></tr>
        <tr><td>🔋 Solarpanels</td><td id="solar">0</td><td id="solarPrice">100</td></tr>
        <tr><td>🪐 Plattformen</td><td id="platform">0</td><td>0</td></tr>
        <tr><td>🚪 Türen</td><td id="doors">0</td><td id="doorPrice">50</td></tr>
        <tr><td>🪟 Fenster</td><td id="windows">0</td><td id="windowPrice">30</td></tr>
        <tr><td>🏠 Dächer</td><td id="roofs">0</td><td id="roofPrice">80</td></tr>
      </tbody>
    </table>
    <div id="buyHelp">Über die Kreise laufen, um zu kaufen</div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>

  <script>
    // Szene, Kamera, Renderer
    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#87ceeb');

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 7, 15);

    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;

    // Licht
    const dirLight = new THREE.DirectionalLight(0xffffff, 1);
    dirLight.position.set(10, 20, 10);
    scene.add(dirLight);

    const ambientLight = new THREE.AmbientLight(0x404040);
    scene.add(ambientLight);

    // Boden & Plattform
    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(500, 500),
      new THREE.MeshStandardMaterial({color: '#228B22'})
    );
    ground.rotation.x = -Math.PI/2;
    ground.position.y = 0;
    scene.add(ground);

    const platformGeom = new THREE.BoxGeometry(20, 0.3, 20);
    const platformMat = new THREE.MeshStandardMaterial({color: '#555555'});
    const platform = new THREE.Mesh(platformGeom, platformMat);
    platform.position.y = 0.15;
    scene.add(platform);

    // Haus Basis
    const houseBaseGeom = new THREE.BoxGeometry(6, 3, 6);
    const houseBaseMat = new THREE.MeshStandardMaterial({color: '#8B4513'});
    const houseBase = new THREE.Mesh(houseBaseGeom, houseBaseMat);
    houseBase.position.set(0, 1.5, 0);
    scene.add(houseBase);

    // Hausobjekte
    const houseObjects = { doors: [], windows: [], roofs: [] };

    // Spieler aus Bausteinen
    const player = new THREE.Group();
    const bodyGeom = new THREE.BoxGeometry(1, 1.5, 0.5);
    const bodyMat = new THREE.MeshStandardMaterial({color: '#ff69b4'});
    const body = new THREE.Mesh(bodyGeom, bodyMat);
    body.position.y = 1;
    player.add(body);

    const headGeom = new THREE.SphereGeometry(0.5, 16, 16);
    const headMat = new THREE.MeshStandardMaterial({color: '#ffaaaa'});
    const head = new THREE.Mesh(headGeom, headMat);
    head.position.y = 2.5;
    player.add(head);

    const armGeom = new THREE.CylinderGeometry(0.15, 0.15, 1);
    const armMat = new THREE.MeshStandardMaterial({color: '#ff69b4'});
    const leftArm = new THREE.Mesh(armGeom, armMat);
    leftArm.position.set(-0.75, 1.5, 0);
    leftArm.rotation.z = Math.PI / 8;
    player.add(leftArm);
    const rightArm = leftArm.clone();
    rightArm.position.set(0.75, 1.5, 0);
    rightArm.rotation.z = -Math.PI / 8;
    player.add(rightArm);

    const legGeom = new THREE.CylinderGeometry(0.2, 0.2, 1.2);
    const legMat = new THREE.MeshStandardMaterial({color: '#551a8b'});
    const leftLeg = new THREE.Mesh(legGeom, legMat);
    leftLeg.position.set(-0.3, 0.1, 0);
    player.add(leftLeg);
    const rightLeg = leftLeg.clone();
    rightLeg.position.set(0.3, 0.1, 0);
    player.add(rightLeg);

    player.position.y = 1;
    scene.add(player);

    // Ressourcen und Preise
    let credits = 200;
    let energy = 1000;
    let solar = 0;
    let labor = 0;
    let handel = 0;
    let platformsBuilt = 0;
    let doorsCount = 0;
    let windowsCount = 0;
    let roofsCount = 0;

    let solarPrice = 100;
    let doorPrice = 50;
    let windowPrice = 30;
    let roofPrice = 80;

    let creditsPerSecond = 0;
    const maxParts = 700;

    // Sprung & Gravitation
    let velocityY = 0;
    let isOnGround = true;

    // Steuerung
    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    // Kaufkreise
    const buyCircles = [];

    function createBuyCircle(x, z, color, type, cost, priceId=null) {
      const geom = new THREE.CircleGeometry(0.5, 32);
      const mat = new THREE.MeshBasicMaterial({color: color, opacity: 0.6, transparent: true});
      const circle = new THREE.Mesh(geom, mat);
      circle.rotation.x = -Math.PI / 2;
      circle.position.set(x, 0.15, z);
      circle.userData = { type, cost, priceId };
      scene.add(circle);
      buyCircles.push(circle);
    }

    // Kaufkreise setzen
    createBuyCircle(-6, -6, '#00ffff', 'solar', solarPrice, 'solarPrice');
    createBuyCircle(-3, -6, '#ffcc00', 'labor', 100);
    createBuyCircle(0, -6, '#cc00ff', 'handel', 250);
    createBuyCircle(3, -6, '#777777', 'platform', 0);

    createBuyCircle(-6, 6, '#654321', 'door', doorPrice, 'doorPrice');
    createBuyCircle(-3, 6, '#aaddff', 'window', windowPrice, 'windowPrice');
    createBuyCircle(0, 6, '#aa3333', 'roof', roofPrice, 'roofPrice');

    // Objekte-Arrays
    const solarModules = [];
    const laborModules = [];
    const handelModules = [];
    const platforms = [];

    // Kauf-Funktion
    function tryBuy(circle) {
      const { type, cost, priceId } = circle.userData;
      const totalParts = solar + labor + handel + platformsBuilt + doorsCount + windowsCount + roofsCount;
      if(totalParts >= maxParts){
        alert(`Maximale Anzahl an Teilen (${maxParts}) erreicht!`);
        return;
      }

      if(type === 'solar'){
        if(credits >= solarPrice){
          credits -= solarPrice;
          solar++;
          creditsPerSecond += 100;
          solarPrice += 100;
          circle.userData.cost = solarPrice;
          if(priceId) document.getElementById(priceId).innerText = solarPrice;
          addSolarModule();
          playSound();
        }
      }
      else if(type === 'labor'){
        if(credits >= cost){
          credits -= cost;
          labor++;
          creditsPerSecond += 100;
          addLaborModule();
          playSound();
        }
      }
      else if(type === 'handel'){
        if(energy >= cost){
          energy -= cost;
          handel++;
          creditsPerSecond += 100;
          addHandelModule();
          playSound();
        }
      }
      else if(type === 'platform'){
        platformsBuilt++;
        creditsPerSecond += 100;
        addPlatform(player.position.x + 5, player.position.z);
        playSound();
      }
      else if(type === 'door'){
        if(credits >= doorPrice){
          credits -= doorPrice;
          doorsCount++;
          creditsPerSecond += 100;
          doorPrice += 50;
          circle.userData.cost = doorPrice;
          if(priceId) document.getElementById(priceId).innerText = doorPrice;
          addDoor();
          playSound();
        }
      }
      else if(type === 'window'){
        if(credits >= windowPrice){
          credits -= windowPrice;
          windowsCount++;
          creditsPerSecond += 100;
          windowPrice += 50;
          circle.userData.cost = windowPrice;
          if(priceId) document.getElementById(priceId).innerText = windowPrice;
          addWindow();
          playSound();
        }
      }
      else if(type === 'roof'){
        if(credits >= roofPrice){
          credits -= roofPrice;
          roofsCount++;
          creditsPerSecond += 100;
          roofPrice += 50;
          circle.userData.cost = roofPrice;
          if(priceId) document.getElementById(priceId).innerText = roofPrice;
          addRoof();
          playSound();
        }
      }

      updateUI();
    }

    // Objekt hinzufügen
    function addSolarModule() {
      const obj = new THREE.Mesh(
        new THREE.CylinderGeometry(0.3, 0.3, 2),
        new THREE.MeshStandardMaterial({color: '#00ffff'})
      );
      obj.position.set(Math.random()*18-9, 1, Math.random()*18-9);
      scene.add(obj);
      solarModules.push(obj);
    }
    function addLaborModule() {
      const obj = new THREE.Mesh(
        new THREE.BoxGeometry(1, 2, 1),
        new THREE.MeshStandardMaterial({color: '#ffcc00'})
      );
      obj.position.set(Math.random()*18-9, 1, Math.random()*18-9);
      scene.add(obj);
      laborModules.push(obj);
    }
    function addHandelModule() {
      const obj = new THREE.Mesh(
        new THREE.BoxGeometry(2, 2, 2),
        new THREE.MeshStandardMaterial({color: '#cc00ff'})
      );
      obj.position.set(Math.random()*18-9, 1, Math.random()*18-9);
      scene.add(obj);
      handelModules.push(obj);
    }
    function addPlatform(x,z){
      const obj = new THREE.Mesh(
        new THREE.BoxGeometry(4, 0.3, 4),
        new THREE.MeshStandardMaterial({color: '#777777'})
      );
      obj.position.set(x, 0.15, z);
      scene.add(obj);
      platforms.push(obj);
    }

    function addDoor(){
      const doorGeom = new THREE.BoxGeometry(1, 2, 0.2);
      const doorMat = new THREE.MeshStandardMaterial({color: '#654321'});
      const door = new THREE.Mesh(doorGeom, doorMat);
      door.position.set(0, 1, 3.1);
      scene.add(door);
      houseObjects.doors.push(door);
    }
    function addWindow(){
      const winGeom = new THREE.PlaneGeometry(1.5, 1.5);
      const winMat = new THREE.MeshStandardMaterial({color: '#aaddff', opacity: 0.7, transparent: true});
      const window = new THREE.Mesh(winGeom, winMat);
      window.position.set(-3.1, 1.5, 0);
      window.rotation.y = Math.PI / 2;
      scene.add(window);
      houseObjects.windows.push(window);
    }
    function addRoof(){
      const roofGeom = new THREE.ConeGeometry(4.5, 2, 4);
      const roofMat = new THREE.MeshStandardMaterial({color: '#aa3333'});
      const roof = new THREE.Mesh(roofGeom, roofMat);
      roof.position.set(0, 3.5, 0);
      roof.rotation.y = Math.PI / 4;
      scene.add(roof);
      houseObjects.roofs.push(roof);
    }

    // UI Update
    function updateUI(){
      document.getElementById('credits').innerText = credits;
      document.getElementById('labor').innerText = labor;
      document.getElementById('handel').innerText = handel;
      document.getElementById('solar').innerText = solar;
      document.getElementById('platform').innerText = platformsBuilt;
      document.getElementById('doors').innerText = doorsCount;
      document.getElementById('windows').innerText = windowsCount;
      document.getElementById('roofs').innerText = roofsCount;

      document.getElementById('solarPrice').innerText = solarPrice;
      document.getElementById('doorPrice').innerText = doorPrice;
      document.getElementById('windowPrice').innerText = windowPrice;
      document.getElementById('roofPrice').innerText = roofPrice;
    }

    // Credits & Energie automatisch erhöhen
    setInterval(() => {
      credits += creditsPerSecond;
      energy += solar * 20;
      updateUI();
    }, 1000);

    // Sound beim Kauf
    function playSound(){
      const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
      audio.play();
    }

    // Spieler- und Kaufkreis-Kollision (automatisches Kaufen beim Überlaufen)
    function checkBuyCircles(){
      for(const circle of buyCircles){
        const dist = player.position.distanceTo(circle.position);
        if(dist < 1){ // Radius ca 1
          tryBuy(circle);
        }
      }
    }

    // Animation & Bewegung
    function animate(){
      const moveSpeed = 0.1;
      if(keys['w']) player.position.z -= moveSpeed;
      if(keys['s']) player.position.z += moveSpeed;
      if(keys['a']) player.position.x -= moveSpeed;
      if(keys['d']) player.position.x += moveSpeed;

      // Springen & Gravitation
      if(keys[' '] && isOnGround){
        velocityY = 0.15;
        isOnGround = false;
      }

      velocityY -= 0.01; // Gravitation
      player.position.y += velocityY;
      if(player.position.y <= 1){
        player.position.y = 1;
        velocityY = 0;
        isOnGround = true;
      }

      checkBuyCircles();

      controls.target.copy(player.position);
      controls.update();

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }

    animate();

    // Fenstergröße anpassen
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
