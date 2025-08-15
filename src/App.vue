<template>
  <div>
    <ThreeViewer />
   
    <!-- Scrollable content that drives ScrollTrigger -->
     <div id="scroll-content">
      <section class="spacer">
        <div ref="heroText" class="text-block">
           <h1>Scroll to see</h1>
         </div>
      </section>

      <section class="spacer">
        
      </section>
      <section class="spacer">
        <div ref="heroText" class="text-block">
          </div>
      </section>

      <section class="spacer">
        <div ref="heroText" class="text-block">
          </div>
      </section>
      
    </div>
  </div>
</template>

<script setup>
import ThreeViewer from './components/ThreeViewer.vue';
import { onMounted, ref } from 'vue';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

const heroText = ref(null);

onMounted(() => {
  // Fade + move the text upward as scroll progresses
  gsap.to(heroText.value, {
    y: -100,
    opacity: 0,
    scrollTrigger: {
      trigger: heroText.value,
      start: "top center",
      end: "bottom top",
      scrub: true,
      markers: true, // optional, remove later
    },
  });
});
</script>
<style>
#scroll-content {
  position: relative;
  z-index: 2;
  color: white;
  padding: 2rem;
}
.spacer {
  height: 100vh;
}
</style>
