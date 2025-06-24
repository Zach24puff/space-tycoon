<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Space Station Tycoon 3D - Sichtbare Szene</title>
<style>
  body {
    margin: 0; overflow: hidden;
    font-family: Arial, sans-serif;
    background: #87ceeb; /* Himmelblau */
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
      </tbody>
    </table>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>

  <script>
    // Szene Setup
    const scene = new THREE.Scene();

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);

    const renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setClearColor(0x87ceeb); // Himmelblau explizit
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enablePan = false;

    // Licht
    const dirLight = new THREE.DirectionalLight(0xffffff, 1);
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight);

    scene.add(new THREE.AmbientLight(0x404040));

    // Boden
    const groundMat = new THREE.MeshStandardMaterial({color: '#228B22'});
    const ground = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), groundMat);
    ground.rotation.x = -Math.PI / 2;
    ground.position.y = 0;
    scene.add(ground);

    // Spieler (ein einfacher Würfel)
    const playerMat = new THREE.MeshStandardMaterial({color: '#ff69b4'});
    const player = new THREE.Mesh(new THREE.BoxGeometry(1, 2, 1), playerMat);
    player.position.y = 1;
    scene.add(player);

    // Kamera schaut auf Spieler
    controls.target.set(player.position.x, player.position.y, player.position.z);
    controls.update();

    // Ressourcen & UI
    let credits = 200;
    let solar = 0;
    let labor = 0;
    let handel = 0;
    let platform = 0;
    let solarPrice = 100;

    function updateUI(){
      document.getElementById('credits').innerText = credits;
      document.getElementById('solar').innerText = solar;
      document.getElementById('labor').innerText = labor;
      document.getElementById('handel').innerText = handel;
      document.getElementById('platform').innerText = platform;
      document.getElementById('solarPrice').innerText = solarPrice;
    }

    // Kaufkreise zum Drüberlaufen
    const buyCircles = [];

    function createBuyCircle(x, z, color, type, cost, priceId){
      const geom = new THREE.CircleGeometry(0.5, 32);
      const mat = new THREE.MeshBasicMaterial({color: color, opacity: 0.6, transparent: true});
      const circle = new THREE.Mesh(geom, mat);
      circle.rotation.x = -Math.PI / 2;
      circle.position.set(x, 0.15, z);
      circle.userData = {type, cost, priceId};
      scene.add(circle);
      buyCircles.push(circle);
    }

    // Beispiel: Kaufkreis für Solarpanel
    createBuyCircle(2, 0, '#00ffff', 'solar', solarPrice, 'solarPrice');

    // Steuerung
    const keys = {};
    document.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
    document.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

    let velocityY = 0;
    let isOnGround = true;

    function tryBuy(circle){
      const {type, cost, priceId} = circle.userData;
      if(type === 'solar'){
        if(credits >= solarPrice){
          credits -= solarPrice;
          solar++;
          creditsPerSecond += 100;
          solarPrice += 100;
          if(priceId) document.getElementById(priceId).innerText = solarPrice;
          playSound();
          updateUI();
        }
      }
    }

    let creditsPerSecond = 0;

    // Automatisch Credits erhöhen
    setInterval(() => {
      credits += creditsPerSecond;
      updateUI();
    }, 1000);

    // Sound beim Kauf
    function playSound(){
      const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
      audio.play();
    }

    // Animation & Bewegung
    function animate(){
      const moveSpeed = 0.1;
      if(keys['w']) player.position.z -= moveSpeed;
      if(keys['s']) player.position.z += moveSpeed;
      if(keys['a']) player.position.x -= moveSpeed;
      if(keys['d']) player.position.x += moveSpeed;

      // Springen
      if(keys[' '] && isOnGround){
        velocityY = 0.15;
        isOnGround = false;
      }

      velocityY -= 0.01;
      player.position.y += velocityY;
      if(player.position.y <= 1){
        player.position.y = 1;
        velocityY = 0;
        isOnGround = true;
      }

      // Kaufkreis Check
      buyCircles.forEach(circle => {
        const dist = player.position.distanceTo(circle.position);
        if(dist < 1) tryBuy(circle);
      });

      controls.target.set(player.position.x, player.position.y, player.position.z);
      controls.update();

      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    }
    animate();

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    updateUI();
  </script>
</body>
</html>
