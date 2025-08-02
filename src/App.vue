<template>
  <div class="home">
    <div class="canvas-container" ref="canvasDom"></div>

    <div class="home-content">
      <div class="home-content-title">
        <h1>汽车展示与选配</h1>
      </div>
      <h2>选择车身颜色</h2>
      <div class="select">
        <div class="select-item" v-for="(item, index) in colors" :key="index" @click="selectColor(index)">
          <div class="select-item-color" :style="{ backgroundColor: item }"></div>
        </div>
      </div>

      <h2>选择贴膜材质</h2>
      <div class="select">
        <div class="select-item" v-for="(item, index) in materials" :key="index" @click="selectMaterial(index)">
          <div class="select-item-text">{{ item.name }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';  // 用于加载gltf模型
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';  // 用于加载压缩的gltf模型

import { onMounted, ref } from 'vue';

let canvasDom = ref(null);
let controls;

// 创建场景
const scene = new THREE.Scene();

// 创建相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 2, 6);  // 相机位置

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);

const render = () => {
  controls?.update();
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};

let wheels = [];
let carBody, frontCar, hoodCar, glassCar;

// 创建材质
const bodyMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,  // 颜色
  metalness: 1,  // 金属感
  roughness: 0.5,  // 粗糙度
  clearcoat: 1,  // 清漆层
  clearcoatRoughness: 0,  // 清漆层粗糙度
});
const frontMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,  // 颜色
  metalness: 1,  // 金属感
  roughness: 0.5,  // 粗糙度
  clearcoat: 1,  // 清漆层
  clearcoatRoughness: 0,  // 清漆层粗糙度
});
const hoodMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,  // 颜色
  metalness: 1,  // 金属感
  roughness: 0.5,  // 粗糙度
  clearcoat: 1,  // 清漆层
  clearcoatRoughness: 0,  // 清漆层粗糙度
});
const wheelsMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,  // 颜色
  metalness: 1,  // 金属感
  roughness: 0.1,  // 粗糙度
});
const glassMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,  // 颜色
  metalness: 0,  // 金属感
  roughness: 0.1,  // 粗糙度
  transmission: 1,  // 透光度
  transparent: true,  // 透明
});

let colors = ["red", "blue", "green", "gray", "orange", "purple"];

let selectColor = (index) => {
  bodyMaterial.color.set(colors[index]);
  frontMaterial.color.set(colors[index]);
  hoodMaterial.color.set(colors[index]);
  wheelsMaterial.color.set(colors[index]);
};

let materials = [
  { name: "磨砂", value: 1 },
  { name: "冰晶", value: 0 },
];
let selectMaterial = (index) => {
  bodyMaterial.clearcoatRoughness = materials[index].value;
  frontMaterial.clearcoatRoughness = materials[index].value;
  hoodMaterial.clearcoatRoughness = materials[index].value;
};

onMounted(() => {
  // 渲染器插入到页面中
  canvasDom.value.appendChild(renderer.domElement);
  // 初始化渲染器，渲染背景
  renderer.setClearColor("#000");
  scene.background = new THREE.Color("#ccc");
  scene.environment = new THREE.Color("#ccc");

  render();

  // 添加网格地面
  const gridHelper = new THREE.GridHelper(10, 10);
  gridHelper.material.transparent = true; // 使网格材质透明
  gridHelper.material.opacity = 0.2; // 设置网格透明度
  scene.add(gridHelper);

  // 添加控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.update();

  // 添加gltf汽车模型
  const gltfLoader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');
  gltfLoader.setDRACOLoader(dracoLoader);
  gltfLoader.load('./public/model/bmw01.glb', (gltf) => {
    const bmw = gltf.scene;
    console.log('bmw', bmw);
    bmw.traverse((child) => {
      if (child.isMesh) { }

      if (child.isMesh && child.name.includes('轮毂')) {
        child.material = wheelsMaterial;  // 设置轮毂材质
        wheels.push(child);
      }
      if (child.isMesh && child.name.includes('Mesh002')) {
        child.material = bodyMaterial;  // 设置车身材质
        carBody = child;
      }
      if (child.isMesh && child.name.includes('前脸')) {
        child.material = frontMaterial;  // 设置前脸材质
        frontCar = child;
      }
      if (child.isMesh && child.name.includes('引擎盖_1')) {
        child.material = hoodMaterial;  // 设置引擎盖材质
        hoodCar = child;
      }
      if (child.isMesh && child.name.includes('挡风玻璃')) {
        child.material = glassMaterial;  // 设置挡风玻璃材质
        glassCar = child;
      }
    });
    scene.add(bmw);
  });

  // 添加灯光
  const light1 = new THREE.DirectionalLight(0xffffff, 1);
  light1.position.set(0, 0, 10);
  scene.add(light1);
  const light2 = new THREE.DirectionalLight(0xffffff, 1);
  light2.position.set(0, 0, -10);
  scene.add(light2);
  const light3 = new THREE.DirectionalLight(0xffffff, 1);
  light3.position.set(10, 0, 0);
  scene.add(light3);
  const light4 = new THREE.DirectionalLight(0xffffff, 1);
  light4.position.set(-10, 0, 0);
  scene.add(light4);
  const light5 = new THREE.DirectionalLight(0xffffff, 1);
  light5.position.set(0, 10, 0);
  scene.add(light5);
  const light6 = new THREE.DirectionalLight(0xffffff, 0.3);
  light6.position.set(5, 10, 0);
  scene.add(light6);
  const light7 = new THREE.DirectionalLight(0xffffff, 0.3);
  light7.position.set(0, 10, 5);
  scene.add(light7);
  const light8 = new THREE.DirectionalLight(0xffffff, 0.3);
  light8.position.set(0, 10, -5);
  scene.add(light8);
  const light9 = new THREE.DirectionalLight(0xffffff, 0.3);
  light9.position.set(-5, 10, 0);
  scene.add(light9);
});

</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.home-content {
  position: fixed;
  top: 0;
  right: 20px;
}

.select-item-color {
  width: 50px;
  height: 50px;
  border: 1px solid #ccc;
  margin: 10px;
  display: inline-block;
  cursor: pointer;
  border-radius: 10px;
}

.select {
  display: flex;
}
</style>
