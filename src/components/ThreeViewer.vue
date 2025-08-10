<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);
const container = ref(null);

onMounted(() => {
  let scene, camera, renderer, model;

  // Helper to get container size
  const getContainerSize = () => {
    const el = container.value;
    return el
      ? { width: el.clientWidth, height: el.clientHeight }
      : { width: window.innerWidth, height: window.innerHeight };
  };
  
  let { width, height } = getContainerSize();

  // 1. Setup scene and camera
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
  camera.position.z = 3;
  
  // 2. Setup renderer
  renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  renderer.setSize(width, height);
  container.value.appendChild(renderer.domElement);

  // 3. Add light
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(0, 1, 2);
  scene.add(light);

  // 4. Load model with Draco support via CDN
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');

  const loader = new GLTFLoader();
  loader.setDRACOLoader(dracoLoader);

  



  loader.load(
    '/model.glb', // Make sure the model is in the /public folder
    (gltf) => {
      model = gltf.scene;
      model.scale.set(1, 1, 1); // Adjust scale as needed
      // Center the model if needed
      const box = new THREE.Box3().setFromObject(model);
      const center = box.getCenter(new THREE.Vector3());
      model.position.sub(center); // Center the model at origin
      scene.add(model);
      setupScrollAnimation();
      animate();
    },
    undefined,
    (error) => {
      console.error('GLTF Load Error:', error);
    }
  );

  // 5. Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  };
  const setupScrollAnimation = () => {
    // const scrollArea=document.createElement('div');
    // scrollArea.style.height = '200vh'; // Make the scroll area taller
    // scrollArea.style.width = '100%';  
    // document.body.appendChild(scrollArea);
    ScrollTrigger.create({
  trigger: document.body, // or "#scroll-content"
  start: "top top",
  end: "bottom bottom",
  scrub: true,
  onUpdate: (self) => {
    const progress = self.progress;
    const angle = progress * Math.PI * 2;
    camera.position.x = Math.sin(angle) * 3;
    camera.position.z = Math.cos(angle) * 3;
    camera.lookAt(0, 0, 0);
    console.log(progress)
  },
});

  };
  // 6. Handle window resize
  window.addEventListener('resize', () => {
    const { width, height } = getContainerSize();
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
  });
});
</script>

<style scoped>
.three-container {
  width: 100vw;
  height: 100vh;
  background: #000;
  overflow: hidden;
  position:fixed;
}
</style>
