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

const bancoInter = "/images/bancoInter.png";

function createScene(canvas: HTMLDivElement) {
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xe2e2e2);

  const camera = new THREE.PerspectiveCamera(
    60,
    sceneSizes.width / sceneSizes.height,
    0.1,
    1000
  );

  camera.position.x = 0;
  camera.position.y = .5;
  camera.position.z = 6.5;

  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(sceneSizes.width, sceneSizes.height);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.setClearColor(0xa3a3a3);

  canvas.appendChild(renderer.domElement);

  const material = new THREE.MeshStandardMaterial({ color: 0x6e6e6e });
  const controls = new OrbitControls(camera, renderer.domElement);

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

  const firstPlaceGeometry = new THREE.BoxGeometry(5, 1, 2);
  const firstPlaceObject = new THREE.Mesh(firstPlaceGeometry, material);
  firstPlaceObject.position.set(0, -2, 0);
  firstPlaceObject.receiveShadow = true;
  scene.add(firstPlaceObject);

  const secondPlaceGeometry = new THREE.BoxGeometry(4, 0.6, 2);
  const secondPlaceObject = new THREE.Mesh(secondPlaceGeometry, material);
  secondPlaceObject.position.set(4, -2.2, 0);
  scene.add(secondPlaceObject);

  const thirdPlaceObject = new THREE.Mesh(secondPlaceGeometry, material);
  thirdPlaceObject.position.set(-4, -2.2, 0);
  scene.add(thirdPlaceObject);

  const pennyMaterials = [];
  const pennyGeometry = new THREE.CylinderGeometry(1.5, 1.5, 0.1, 32);
  const pennySideMaterial = new THREE.MeshLambertMaterial({ color: 0x4f3e37 });

  pennyMaterials.push(pennySideMaterial);

  const headsLoader = new THREE.TextureLoader();
  headsLoader.load(bancoInter, (texture: THREE.Texture) => {
    const materialTop = new THREE.MeshBasicMaterial({ map: texture });
    pennyMaterials.push(materialTop);
  });

  const tailsLoader = new THREE.TextureLoader();
  tailsLoader.load(bancoInter, (texture: THREE.Texture) => {
    texture.wrapS = THREE.RepeatWrapping;
    texture.repeat.x = - 1;
    texture.flipY = false;
    const materialBottom = new THREE.MeshBasicMaterial({ map: texture });
    pennyMaterials.push(materialBottom);
  });

  const firstPenny = new THREE.Mesh(pennyGeometry, pennyMaterials);

  firstPenny.position.set(0, 0.7, 0.3);
  firstPenny.rotateX(1.56);
  firstPenny.rotateY(1.56);
  firstPenny.castShadow = true;
  scene.add(firstPenny);

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

  function animate() {
    TWEEN.update();
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }
  animate();
}

onMounted(() => {
  if (bestMatchScene.value) createScene(bestMatchScene.value);
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
