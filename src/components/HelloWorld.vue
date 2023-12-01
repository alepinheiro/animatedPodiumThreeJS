<script setup lang="ts">
import * as THREE from "three";
import { FontLoader } from "three/addons/loaders/FontLoader.js";
import { TextGeometry } from "three/addons/geometries/TextGeometry.js";
//@ts-expect-error
import * as TWEEN from "@tweenjs/tween.js";
import { OrbitControls } from "three/examples/jsm/Addons.js";
import { onBeforeUnmount, onMounted, ref } from "vue";

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
  const renderer = new THREE.WebGLRenderer({ antialias: true });
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
  camera.position.y = -0.5;
  camera.position.z = 6.5;

  return camera;
};

const startDirectionalLight = (scene: THREE.Scene) => {
  const light = new THREE.DirectionalLight(0xffffff, 2);
  light.position.set(0, 10, 0.2);
  light.castShadow = true;
  light.shadow.mapSize.width = 256;
  light.shadow.mapSize.height = 256;
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

const createText = (text: string, size: number) => {
  const textMesh = new THREE.Group();
  const fontLoader = new FontLoader();

  fontLoader.load("/fonts/OpenSansBold.json", (loadedFont) => {
    const textMaterial = new THREE.MeshPhongMaterial({
      color: 0x000,
      specular: 0x444444,
      shininess: 20,
    });

    const firstText = new TextGeometry(text, {
      font: loadedFont,
      size: size,
      height: 1,
      curveSegments: 10,
    });

    firstText.center();

    const tMesh = new THREE.Mesh(firstText, textMaterial);

    textMesh.add(tMesh);
  });

  return textMesh;
};

const addPodiumBase = (scene: THREE.Scene) => {
  //
  const material = new THREE.MeshStandardMaterial({ color: 0x6e6e6e });

  const firstPlaceGeometry = new THREE.BoxGeometry(5, 1, 2);
  const firstPlaceObject = new THREE.Mesh(firstPlaceGeometry, material);
  firstPlaceObject.position.set(0, -2, 0);
  firstPlaceObject.receiveShadow = true;
  scene.add(firstPlaceObject);

  const firstPlaceText = createText("Melhor Operação (75%)", 52);
  firstPlaceText.scale.set(0.005, 0.005, 0.005);
  firstPlaceText.rotation.y = 0.00001; // radiant
  firstPlaceText.position.set(0, -2, 1);
  scene.add(firstPlaceText);

  const secondPlaceGeometry = new THREE.BoxGeometry(4, 0.6, 2);
  const secondPlaceObject = new THREE.Mesh(secondPlaceGeometry, material);
  secondPlaceObject.position.set(-4, -2.2, 0);
  secondPlaceObject.receiveShadow = true;
  scene.add(secondPlaceObject);

  const secondPlaceText = createText("Segundo lugar (60%)", 40);
  secondPlaceText.scale.set(0.005, 0.005, 0.005);
  secondPlaceText.rotation.y = 0.00001; // radiant
  secondPlaceText.position.set(-4.25, -2.22, 1);
  scene.add(secondPlaceText);

  const thirdPlaceObject = new THREE.Mesh(secondPlaceGeometry, material);
  thirdPlaceObject.position.set(4, -2.2, 0);
  thirdPlaceObject.receiveShadow = true;
  scene.add(thirdPlaceObject);

  const thirdPlaceText = createText("Terceiro lugar (42%)", 40);
  thirdPlaceText.scale.set(0.005, 0.005, 0.005);
  thirdPlaceText.rotation.y = 0.00001; // radiant
  thirdPlaceText.position.set(4.25, -2.22, 1);
  scene.add(thirdPlaceText);
};

const createPenny = (imageUrl: string, radius: number) => {
  const pennyMaterials = [];
  const pennyGeometry = new THREE.CylinderGeometry(radius, radius, 0.1, 32);
  const pennySideMaterial = new THREE.MeshLambertMaterial({ color: 0x4f3e37 });

  pennyMaterials.push(pennySideMaterial);

  const headsLoader = new THREE.TextureLoader();
  headsLoader.load(imageUrl, (texture: THREE.Texture) => {
    texture.wrapS = THREE.RepeatWrapping;
    texture.flipY = true;
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
};

const showFirstPlace = (scene: THREE.Scene, penny: THREE.Mesh) => {
  penny.position.set(0, 0.7, 0.3);
  penny.rotateX(1.56);
  penny.rotateY(1.56);
  penny.castShadow = true;
  scene.add(penny);

  const animationStart = new TWEEN.Tween({
    y: 10,
  })
    .to({ y: 0.5 }, 1000)
    .onUpdate((coords: { x: number; y: number }) => {
      penny.position.y = coords.y;
    })
    .easing(TWEEN.Easing.Exponential.Out);

  const animationMiddle = new TWEEN.Tween({
    y: 0.5,
  })
    .to({ y: 0 }, 1000)
    .onUpdate((coords: { x: number; y: number }) => {
      penny.position.y = coords.y;
    })
    .easing(TWEEN.Easing.Exponential.In);

  const animationEnd = new TWEEN.Tween({
    y: 0,
  })
    .to({ y: 0.5 }, 1000)
    .onUpdate((coords: { x: number; y: number }) => {
      penny.position.y = coords.y;
    })
    .easing(TWEEN.Easing.Exponential.Out);

  animationStart.chain(animationMiddle);
  animationMiddle.chain(animationEnd);
  animationEnd.chain(animationMiddle);
  animationStart.start();
};

const showSecondPlace = (
  scene: THREE.Scene,
  penny: THREE.Mesh,
  position: { x: number; y: number; z: number }
) => {
  penny.position.set(position.x, position.y, position.z);
  penny.rotateX(1.56);
  penny.rotateY(1.56);
  penny.castShadow = true;
  scene.add(penny);

  const animationStart = new TWEEN.Tween({
    y: 6,
  })
    .to({ y: -0.5 }, 2000)
    .onUpdate((coords: { x: number; y: number }) => {
      penny.position.y = coords.y;
    })
    .easing(TWEEN.Easing.Exponential.Out)
    .delay(500);

  animationStart.start();
};

const startCanvas = (canvas: HTMLDivElement) => {
  const scene = startScene();
  const camera = startCamera();
  const renderer = startRenderer();
  const controls = new OrbitControls(camera, renderer.domElement);

  canvas.appendChild(renderer.domElement);

  startDirectionalLight(scene);
  addPodiumBase(scene);

  const firstPenny = createPenny("/images/bancoInter.png", 1.5);
  showFirstPlace(scene, firstPenny);
  const secondPenny = createPenny("/images/bancoItau.png", 1.2);
  showSecondPlace(scene, secondPenny, { x: 4, y: 6, z: 0.3 });

  const thirdPenny = createPenny("/images/bancoSantander.png", 1.2);
  showSecondPlace(scene, thirdPenny, { x: -4, y: 6, z: 0.3 });

  function animate() {
    secondPenny.rotateX(0.01);
    thirdPenny.rotateX(-0.01);
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
    TWEEN.update();
  }
  animate();

  onBeforeUnmount(() => {
    scene.clear();
  });
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
