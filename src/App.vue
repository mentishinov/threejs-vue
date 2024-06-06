<template>
  <div id="app"></div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import * as dat from 'dat.gui';

export default {
  name: 'App',
  mounted() {
    this.initThreeJS();
  },
  methods: {
    initThreeJS() {
      // Сцена
      const scene = new THREE.Scene();

      // Камера
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(0, 2, 5);

      // Рендер
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      this.$el.appendChild(renderer.domElement);

      // Контрол
      const controls = new OrbitControls(camera, renderer.domElement);

      // Свет
      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(5, 10, 7.5);
      light.castShadow = true;
      scene.add(light);

      const ambientLight = new THREE.AmbientLight(0x404040);
      scene.add(ambientLight);

      // Environment 
      const loader = new THREE.CubeTextureLoader();
      const textureCube = loader.load([
        'textures/posx.jpg', 'textures/negx.jpg',
        'textures/posy.jpg', 'textures/negy.jpg',
        'textures/posz.jpg', 'textures/negz.jpg'
      ]);
      scene.background = textureCube;

      // Геом фигуры
      const geometry1 = new THREE.BoxGeometry(1, 1, 1);
      const material1 = new THREE.MeshStandardMaterial({
        color: 0x00ff00,
        metalness: 0.5,
        roughness: 0.1,
        envMap: textureCube
      });
      const cube = new THREE.Mesh(geometry1, material1);
      cube.castShadow = true;
      cube.position.y = 0.5; // Позиция куба
      scene.add(cube);

      const geometry2 = new THREE.SphereGeometry(0.5, 32, 32);
      const material2 = new THREE.MeshStandardMaterial({
        color: 0x0000ff,
        metalness: 0.5,
        roughness: 0.1,
        envMap: textureCube
      });
      const sphere = new THREE.Mesh(geometry2, material2);
      sphere.position.set(2, 0.5, 0); // Позиция сферы
      sphere.castShadow = true;
      scene.add(sphere);

      // Текстуры двери
      const textureLoader = new THREE.TextureLoader();
      const woodTexture = textureLoader.load('textures/wood_texture.jpg');
      woodTexture.wrapS = THREE.RepeatWrapping;
      woodTexture.wrapT = THREE.RepeatWrapping;
      woodTexture.repeat.set(1, 2);

      // Дверь
      const doorGeometry = new THREE.BoxGeometry(1, 2, 0.1);
      const doorMaterial = new THREE.MeshStandardMaterial({
        map: woodTexture,
        metalness: 0.2,
        roughness: 0.7,
        envMap: textureCube
      });
      const door = new THREE.Mesh(doorGeometry, doorMaterial);
      door.position.set(-2, 1, 0); // Позиция двери
      door.castShadow = true;
      scene.add(door);

      // Установка размеров двери
      function resizeDoor(width, height) {
        door.scale.set(width, height / 2, 1);
        woodTexture.repeat.set(width, height); // Обновление повторения текстуры в зависимости от размера двери
      }

      // GUI 
      const gui = new dat.GUI();
      const doorFolder = gui.addFolder('Door Dimensions');
      const doorSettings = {
        width: 1,
        height: 2
      };
      doorFolder.add(doorSettings, 'width', 0.5, 3).onChange(value => resizeDoor(value, doorSettings.height));
      doorFolder.add(doorSettings, 'height', 0.5, 3).onChange(value => resizeDoor(doorSettings.width, value));
      doorFolder.open();

      // Цикл анимации
      function animate() {
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
      }
      animate();

      // Изменить размеры двери
      window.addEventListener('resize', () => {
        const width = window.innerWidth;
        const height = window.innerHeight;
        renderer.setSize(width, height);
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
      });
    }
  }
};
</script>

<style>
body {
  margin: 0;
  overflow: hidden;
}
</style>
