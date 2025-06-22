<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<title>スマホ対応ミニマイクラ</title>
<style>
  body { margin:0; overflow:hidden; touch-action: manipulation; }
  canvas { display:block; }
</style>
</head>
<body>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>

<script>
// 基本セットアップ
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(5, 7, 10);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// タッチ対応OrbitControls
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.target.set(5, 0, 5);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.update();

// グリッド（地面）
const gridHelper = new THREE.GridHelper(20, 20);
scene.add(gridHelper);

// ライト
const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(10, 20, 10);
scene.add(directionalLight);
const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
scene.add(ambientLight);

// ブロックサイズ
const BLOCK_SIZE = 1;
const blocks = {};

// テクスチャロード
const textureLoader = new THREE.TextureLoader();
const grassTexture = textureLoader.load('https://threejsfundamentals.org/threejs/resources/images/flower-1.jpg'); // とりあえず花テクスチャの代用
grassTexture.magFilter = THREE.NearestFilter;
grassTexture.minFilter = THREE.NearestMipMapNearestFilter;

// ブロックマテリアル
const blockMaterial = new THREE.MeshLambertMaterial({map: grassTexture});

// ブロック追加関数
function addBlock(x, y, z) {
  const geometry = new THREE.BoxGeometry(BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
  const mesh = new THREE.Mesh(geometry, blockMaterial);
  mesh.position.set(x + BLOCK_SIZE/2, y + BLOCK_SIZE/2, z + BLOCK_SIZE/2);
  scene.add(mesh);
  blocks[`${x}_${y}_${z}`] = mesh;
}

// 初期地面配置
for(let x=0; x<10; x++) {
  for(let z=0; z<10; z++) {
    addBlock(x, 0, z);
  }
}

// 効果音用AudioContext
const AudioContext = window.AudioContext || window.webkitAudioContext;
const audioCtx = new AudioContext();

function playSound(freq = 440, duration = 0.1) {
  if(audioCtx.state === 'suspended') audioCtx.resume();
  const oscillator = audioCtx.createOscillator();
  const gainNode = audioCtx.createGain();

  oscillator.type = 'square';
  oscillator.frequency.setValueAtTime(freq, audioCtx.currentTime);
  gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);

  oscillator.connect(gainNode);
  gainNode.connect(audioCtx.destination);

  oscillator.start();
  oscillator.stop(audioCtx.currentTime + duration);
}

// レイキャストとクリック処理
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

function onClick(event) {
  event.preventDefault();

  // タッチ対応
  let clientX, clientY;
  if(event.changedTouches) {
    clientX = event.changedTouches[0].clientX;
    clientY = event.changedTouches[0].clientY;
  } else {
    clientX = event.clientX;
    clientY = event.clientY;
  }

  mouse.x = ( clientX / window.innerWidth ) * 2 - 1;
  mouse.y = - ( clientY / window.innerHeight ) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(Object.values(blocks));

  if (intersects.length > 0) {
    const intersect = intersects[0];
    const pos = intersect.object.position.clone().subScalar(BLOCK_SIZE/2);
    const key = `${pos.x}_${pos.y}_${pos.z}`;

    if(event.shiftKey) {
      // Shift押しクリックはPC用：壊す
      scene.remove(blocks[key]);
      delete blocks[key];
      playSound(220, 0.2);
    } else {
      // 普通タップ・クリックは置く
      const normal = intersect.face.normal;
      const newPos = pos.clone().add(normal);
      const newKey = `${newPos.x}_${newPos.y}_${newPos.z}`;
      if(!blocks[newKey]) {
        addBlock(newPos.x, newPos.y, newPos.z);
        playSound(440, 0.1);
      }
    }
  }
}

window.addEventListener('click', onClick);
window.addEventListener('touchend', onClick);

// アニメーションループ
function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

animate();

// リサイズ対応
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

</body>
</html>
