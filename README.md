<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
  <title>Space Station Tycoon 2D</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      touch-action: none;
      background-color: #000;
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
      font-weight: bold;
    }

    canvas {
      display: block;
    }
  </style>
</head>
<body>
  <div id="ui">
    <h1>🚀 Space Tycoon 2D</h1>
    <div>💰 Credits: <span id="credits">200</span></div>
    <div>🔋 Energie: <span id="solar">0</span></div>
    <div>🏪 Handel: <span id="handel">0</span></div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
  <script>
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x111111);

    const camera = new THREE.OrthographicCamera(-750, 750, 750, -750, 0.1, 10000);
    camera.position.set(0, 1000, 0);
    camera.lookAt(0, 0, 0);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    scene.add(new THREE.AmbientLight(0xffffff));

    const platforms = [];
    const totalBuyables = 1500;
    const perPlatform = Math.floor(totalBuyables / 3);

    for (let i = 0; i < 3; i++) {
      const platformGeo = new THREE.PlaneGeometry(1500, 1500);
      const platformMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
      const platform = new THREE.Mesh(platformGeo, platformMat);
      platform.rotation.x = -Math.PI / 2;
      platform.position.set(i * 1600, 0, 0);
      scene.add(platform);
      platforms.push(platform);
    }

    const buyables = [];
    const types = ['solar', 'handel'];

    for (let p = 0; p < platforms.length; p++) {
      for (let i = 0; i < perPlatform; i++) {
        const type = types[Math.floor(Math.random() * types.length)];
        const color = type === 'solar' ? 0xffff00 : 0x00ffcc;
        const circle = new THREE.Mesh(
          new THREE.CircleGeometry(10, 32),
          new THREE.MeshBasicMaterial({ color, transparent: true, opacity: 0.8 })
        );
        circle.rotation.x = -Math.PI / 2;
        circle.position.set(
          platforms[p].position.x + Math.random() * 1400 - 700,
          0.1,
          Math.random() * 1400 - 700
        );
        circle.userData = { type, bought: false };
        scene.add(circle);
        buyables.push(circle);
      }
    }

    const state = { credits: 200, solar: 0, handel: 0 };

    function updateUI() {
      document.getElementById('credits').innerText = Math.floor(state.credits);
      document.getElementById('solar').innerText = state.solar;
      document.getElementById('handel').innerText = state.handel;
    }

    function checkBuys() {
      buyables.forEach((item) => {
        if (!item.userData.bought) {
          const dx = item.position.x;
          const dz = item.position.z;
          if (Math.abs(dx) < 20 && Math.abs(dz) < 20) {
            const cost = item.userData.type === 'solar' ? 100 : 300;
            if (state.credits >= cost) {
              state.credits -= cost;
              state[item.userData.type]++;
              item.userData.bought = true;
              scene.remove(item);
              const audio = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_6efb4b5c60.mp3?filename=menu-click-110818.mp3");
              audio.volume = 0.3;
              audio.play();
              updateUI();
            }
          }
        }
      });
    }

    setInterval(() => {
      state.credits += state.solar * 100 + state.handel * 150;
      updateUI();
    }, 1000);

    function animate() {
      requestAnimationFrame(animate);
      checkBuys();
      renderer.render(scene, camera);
    }

    animate();
    updateUI();

    window.addEventListener('resize', () => {
      camera.left = -window.innerWidth / 2;
      camera.right = window.innerWidth / 2;
      camera.top = window.innerHeight / 2;
      camera.bottom = -window.innerHeight / 2;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
