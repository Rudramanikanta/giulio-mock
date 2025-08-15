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
let Clock = new THREE.Clock();
let mixer=null;
onMounted(() => {
  let scene, camera, renderer, model, particles;

  // Get container size
  const getContainerSize = () => {
    const el = container.value;
    return el
      ? { width: el.clientWidth, height: el.clientHeight }
      : { width: window.innerWidth, height: window.innerHeight };
  };

  let { width, height } = getContainerSize();

  // Scene & Camera
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(40, width / height, 0.1, 1000);
  camera.position.z = 2.5;

  // Renderer
  renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  renderer.setSize(width, height);
  container.value.appendChild(renderer.domElement);

  // Lighting
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(0, 1, 2);
  scene.add(light);
  scene.position.y-=1.5
  // Particle Background
  const particleCount = 8000;
  const particleGeometry = new THREE.BufferGeometry();
  const positions = [];
//animation

  for (let i = 0; i < particleCount; i++) {
    positions.push(
      (Math.random() - 0.5) * 20,
      (Math.random() - 0.5) * 20,
      (Math.random() - 0.5) * 20
    );
  }

  particleGeometry.setAttribute(
    'position',
    new THREE.Float32BufferAttribute(positions, 3)
  );

  const particleMaterial = new THREE.PointsMaterial({
    color: 0xffffff,
    size: 0.02,
    transparent: true,
    opacity: 0.5,
    depthWrite: false,
  });

  particles = new THREE.Points(particleGeometry, particleMaterial);
  scene.add(particles);

  // Load 3D Model
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');

  const loader = new GLTFLoader();
  loader.setDRACOLoader(dracoLoader);
  let rightHandBone=null;
  loader.load(
    '/model.glb',
    (gltf) => {
      model = gltf.scene;
      model.scale.set(1, 1, 1);
      mixer=new THREE.AnimationMixer(model);
      const action=mixer.clipAction(gltf.animations[0]);
      action.play();
      console.log(gltf.animations)
      // Center the model using bounding box
      model.traverse((child) => {
        if (child.isMesh) {
          child.geometry.computeBoundingBox();
        }
         // Use this to identify correct bone
        if (child.name === 'handR') {
          
          rightHandBone = child;
        }
      
      });
      
      const bbox = new THREE.Box3().setFromObject(model);
      const center = bbox.getCenter(new THREE.Vector3());
      model.position.sub(center);
      action.paused = true

    // ScrollTrigger controls animation time
    gsap.to(action, {
      time: action.getClip().duration,
      ease: 'none',
      scrollTrigger: {
        trigger: '.scroll-container',
        start: 'top top',
        end: '+=3000',
        scrub: true,
        markers: false // set to true to debug
      }
    })
      scene.add(model);
      setupScrollAnimation();
      animate();
    },
    undefined,
    (error) => {
      console.error('GLTF Load Error:', error);
    }
  );

  // Animate loop
  const animate = () => {
    requestAnimationFrame(animate);
    if(mixer) mixer.update(Clock.getDelta());
    // Animate particles
    if (particles) {
      particles.rotation.y += 0.0005;
      particles.rotation.x += 0.0003;
    }

    renderer.render(scene, camera);
  };

  // ScrollTrigger camera animation
  const setupScrollAnimation = () => {
    ScrollTrigger.create({
      trigger: document.body,
      start: "top top",
      end: "bottom bottom",
      scrub: true,
      onUpdate: (self) => {
        const progress = self.progress;
        const angle = progress * Math.PI * 0.5; // Partial orbit
        camera.position.x = Math.sin(angle) * 1.5;
        camera.position.z = 2.5 + Math.cos(angle) * 0.5;
        camera.lookAt(0, 0, 0);
        if (rightHandBone) {
          console.log("animation")
        rightHandBone.rotation.x = progress * Math.PI * 0.5;
        rightHandBone.rotation.z = Math.sin(progress * Math.PI) * 0.2;
      }
      },
    });
   
  };

  // Handle window resize
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
  background: radial-gradient(circle at center, #111, #000);
  overflow: hidden;
  position: fixed;
  top: 0;
  left: 0;
}
</style>
