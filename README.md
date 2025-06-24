<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Test 3D Szene</title>
<style>
  body { margin:0; overflow:hidden; }
</style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
<script>
  const scene = new THREE.Scene();
  scene.background = new THREE.Color('#87ceeb'); // blauer Himmel

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  camera.position.set(30, 30, 30);
  camera.lookAt(0, 0, 0);

  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enablePan = false;

  // Licht
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(20, 50, 20);
  scene.add(directionalLight);

  const ambientLight = new THREE.AmbientLight(0x606060);
  scene.add(ambientLight);

  // Boden - graue Plattform 100x100
  const platformGeometry = new THREE.PlaneGeometry(100, 100);
  const platformMaterial = new THREE.MeshStandardMaterial({ color: '#808080' });
  const platformMesh = new THREE.Mesh(platformGeometry, platformMaterial);
  platformMesh.rotation.x = -Math.PI/2;
  platformMesh.position.y = 0;
  scene.add(platformMesh);

  // Gras-Rand (grün) links und rechts
  const grassGeometryV = new THREE.PlaneGeometry(10, 100);
  const grassMaterial = new THREE.MeshStandardMaterial({ color: '#228B22' });

  const grassLeft = new THREE.Mesh(grassGeometryV, grassMaterial);
  grassLeft.rotation.x = -Math.PI/2;
  grassLeft.position.set(-55, 0.01, 0);
  scene.add(grassLeft);

  const grassRight = new THREE.Mesh(grassGeometryV, grassMaterial);
  grassRight.rotation.x = -Math.PI/2;
  grassRight.position.set(55, 0.01, 0);
  scene.add(grassRight);

  // Spieler Würfel pink
  const playerGeometry = new THREE.BoxGeometry(1, 2, 1);
  const playerMaterial = new THREE.MeshStandardMaterial({ color: '#ff69b4' });
  const player = new THREE.Mesh(playerGeometry, playerMaterial);
  player.position.set(0, 1, 0);
  scene.add(player);

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });

  function animate() {
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }

  animate();
</script>
</body>
</html>
