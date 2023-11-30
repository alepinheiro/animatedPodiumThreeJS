<script setup lang="ts">
import * as THREE from "three";
import * as TWEEN from "@tweenjs/tween.js";
import { OrbitControls } from "three/examples/jsm/Addons.js";
import { onMounted, ref } from "vue";

const bestMatchScene = ref<HTMLDivElement | null>(null);
const sceneSizes = {
  width: 1000,
  height: 500,
};

const startScene = () => {
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xe2e2e2);
  return scene;
};

const startRenderer = () => {
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(sceneSizes.width, sceneSizes.height);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.setClearColor(0xa3a3a3);
  return renderer;
};

const startCamera = () => {
  const camera = new THREE.PerspectiveCamera(
    60,
    sceneSizes.width / sceneSizes.height,
    0.1,
    1000
  );

  camera.position.x = 0;
  camera.position.y = 0.5;
  camera.position.z = 6.5;

  return camera;
};

const startDirectionalLight = (scene: THREE.Scene) => {
  const light = new THREE.DirectionalLight(0xffffff, 4);
  light.position.set(0, 5, 0.2);
  light.castShadow = true;
  light.shadow.mapSize.width = 128;
  light.shadow.mapSize.height = 128;
  light.shadow.camera.near = 0.5;
  light.shadow.camera.far = 25;
  light.shadow.camera.left = -10;
  light.shadow.camera.right = 10;
  light.shadow.camera.top = 10;
  light.shadow.camera.bottom = -10;
  light.shadow.radius = 15;
  light.shadow.blurSamples = 25;
  scene.add(light);

  const ambientLight = new THREE.AmbientLight(0xfffeee, 5);
  scene.add(ambientLight);
};

const addPodiumBase = (scene: THREE.Scene) => {
  const material = new THREE.MeshStandardMaterial({ color: 0x6e6e6e });

  const firstPlaceGeometry = new THREE.BoxGeometry(5, 1, 2);
  const firstPlaceObject = new THREE.Mesh(firstPlaceGeometry, material);
  firstPlaceObject.position.set(0, -2, 0);
  firstPlaceObject.receiveShadow = true;
  scene.add(firstPlaceObject);

  const secondPlaceGeometry = new THREE.BoxGeometry(4, 0.6, 2);
  const secondPlaceObject = new THREE.Mesh(secondPlaceGeometry, material);
  secondPlaceObject.position.set(4, -2.2, 0);
  secondPlaceObject.receiveShadow = true;
  scene.add(secondPlaceObject);

  const thirdPlaceObject = new THREE.Mesh(secondPlaceGeometry, material);
  thirdPlaceObject.position.set(-4, -2.2, 0);
  thirdPlaceObject.receiveShadow = true;
  scene.add(thirdPlaceObject);
};

const createPenny = (imageUrl: string, radius: number) => {
  const pennyMaterials = [];
  const pennyGeometry = new THREE.CylinderGeometry(radius, radius, 0.1, 32);
  const pennySideMaterial = new THREE.MeshLambertMaterial({ color: 0x4f3e37 });

  pennyMaterials.push(pennySideMaterial);

  const headsLoader = new THREE.TextureLoader();
  headsLoader.load(imageUrl, (texture: THREE.Texture) => {
    const materialTop = new THREE.MeshBasicMaterial({ map: texture });
    pennyMaterials.push(materialTop);
  });

  const tailsLoader = new THREE.TextureLoader();
  tailsLoader.load(imageUrl, (texture: THREE.Texture) => {
    texture.wrapS = THREE.RepeatWrapping;
    texture.repeat.x = -1;
    texture.flipY = false;
    const materialBottom = new THREE.MeshBasicMaterial({ map: texture });
    pennyMaterials.push(materialBottom);
  });

  return new THREE.Mesh(pennyGeometry, pennyMaterials);
}

function startCanvas(canvas: HTMLDivElement) {
  // instancia a cena
  const scene = startScene();

  // instancia a camera
  const camera = startCamera();

  // instancia o renderer
  const renderer = startRenderer();

  // adiciona o canvas
  canvas.appendChild(renderer.domElement);

  // habilita os controles
  const controls = new OrbitControls(camera, renderer.domElement);

  // instancia a luz
  startDirectionalLight(scene);

  // adiciona o pódio na cena
  addPodiumBase(scene);

  // adiciona a medalha
  const firstPenny = createPenny("/images/bancoInter.png", 1.5);

  firstPenny.position.set(0, 0.7, 0.3);
  firstPenny.rotateX(1.56);
  firstPenny.rotateY(1.56);
  firstPenny.castShadow = true;
  scene.add(firstPenny);

  const secondPenny = createPenny("/images/bancoItau.png", 1.2);
  secondPenny.position.set(4, -0.5, 0.3);
  secondPenny.rotateX(1.56);
  secondPenny.rotateY(1.56);
  secondPenny.castShadow = true;
  scene.add(secondPenny);

  const thirdPenny = createPenny("/images/bancoSantander.png", 1.2);
  thirdPenny.position.set(-4, -0.5, 0.3);
  thirdPenny.rotateX(1.56);
  thirdPenny.rotateY(1.56);
  thirdPenny.castShadow = true;
  scene.add(thirdPenny);

  // anima a medalha

  const firstPennyStart = new TWEEN.Tween({
    y: 0,
  })
    .to({ y: 0.5 }, 1000)
    .onUpdate((coords) => {
      firstPenny.position.y = coords.y;
    })
    .easing(TWEEN.Easing.Exponential.Out);

  const firstPennyEnd = new TWEEN.Tween({
    y: 0.5,
  })
    .to({ y: 0 }, 1000)
    .onUpdate((coords) => {
      firstPenny.position.y = coords.y;
    })
    .easing(TWEEN.Easing.Exponential.In);

  firstPennyStart.chain(firstPennyEnd);
  firstPennyEnd.chain(firstPennyStart);
  firstPennyStart.start();

  // inicia a animação
  function animate() {
    secondPenny.rotateX(0.01);
    thirdPenny.rotateX(-0.01);
    TWEEN.update();
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }
  animate();
}

onMounted(() => {
  if (bestMatchScene.value) startCanvas(bestMatchScene.value);
});
</script>

<template>
  <div
    ref="bestMatchScene"
    :style="{
      width: `${sceneSizes.width}px`,
      height: `${sceneSizes.height}px`,
    }"
    class="border rounded-lg bg-zinc-300 overflow-hidden"
  ></div>
</template>
