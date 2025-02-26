<script>
  import { Canvas } from '@threlte/core';
  import Scene from '$lib/components/Scene.svelte';
  import { spring } from 'svelte/motion';
  import { onMount } from 'svelte';
  import { fade, blur } from 'svelte/transition';

  let scrollY;
  let innerHeight;
  const progress = spring(0);
  
  // Track which sections should be visible based on scroll position
  let section1Visible = false;
  let section2Visible = false;
  let section3Visible = false;
  let section4Visible = false;
  let section5Visible = false;

  // Update section visibility based on scroll position
  $: {
    const pageHeight = 500; // vh
    const sectionHeight = pageHeight / 5; // Each section is 1/5 of the total height
    
    // Calculate thresholds for when each section becomes visible
    section1Visible = scrollY < sectionHeight;
    section2Visible = scrollY >= sectionHeight * 0.8 && scrollY < sectionHeight * 1.8;
    section3Visible = scrollY >= sectionHeight * 1.8 && scrollY < sectionHeight * 2.8;
    section4Visible = scrollY >= sectionHeight * 2.8 && scrollY < sectionHeight * 3.8;
    section5Visible = scrollY >= sectionHeight * 3.8;
  }

  onMount(() => {
    const updateProgress = () => {
      const maxScroll = document.body ? document.body.scrollHeight - innerHeight : 0;
      progress.set(maxScroll > 0 ? scrollY / maxScroll : 0);
    };

    document.body.style.height = '500vh'; // Make page scrollable

    // Update progress initially and on scroll/resize
    updateProgress();
    window.addEventListener('scroll', updateProgress);
    window.addEventListener('resize', updateProgress);

    return () => {
      window.removeEventListener('scroll', updateProgress);
      window.removeEventListener('resize', updateProgress);
    };
  });
</script>

<svelte:window bind:scrollY bind:innerHeight />

<div class="fixed inset-0">
  <Canvas>
    <Scene progress={$progress} />
  </Canvas>
</div>

<div class="relative z-10">
  <section class="h-screen  text-white p-8 ">
    
      <h1 class="text-4xl font-bold p-4 rounded backdrop-blur-sm bg-black/30" 
          in:blur={{duration: 800, amount: 10}}>
        The Structure of a Neuron (Work in Progress)
      </h1>
      <p>Scroll to continue</p>
    
  </section>

  <section class="h-screen flex items-end justify-start text-white p-8">
   
      <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30"
           in:blur={{duration: 800, amount: 10}}>
        <h2 class="text-3xl mb-4">Cell Body (Soma)</h2>
        <!--<p>The soma is the cell body of the neuron, containing the nucleus and other vital organelles.</p>-->
      </div>
   
  </section>

  <section class="h-screen flex items-end justify-start text-white p-8">
   
      <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30"
           in:blur={{duration: 800, amount: 10}}>
        <h2 class="text-3xl mb-4">Dendrites</h2>
        <!--<p>Dendrites are branch-like extensions that receive signals from other neurons.</p>-->
      </div>
    
  </section>

  <section class="h-screen flex items-end justify-start text-white p-8">
   
      <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30"
           in:blur={{duration: 800, amount: 10}}>
        <h2 class="text-3xl mb-4">Axon</h2>
       <!-- <p>The axon is a long fiber that conducts electrical impulses away from the soma to other neurons.</p> -->
      </div>
   
  </section>

  <section class="h-screen flex items-end justify-end text-white p-8">
    
      <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30"
           in:blur={{duration: 800, amount: 10}}>
        <h2 class="text-3xl mb-4">Synaptic Terminals</h2>
       <!-- <p>These terminals release neurotransmitters to communicate with other neurons.</p> -->
      </div>
    
  </section>
</div>

<style>
  :global(body) {
    margin: 0;
    background: #000;
    color: white;
    overflow-x: hidden;
  }

  .relative.z-10 {
    pointer-events: none; /* Ensure text does not block interaction with the model */
  }
  section {
    height: 120vh;
  }
  .relative.z-10 section > div {
    pointer-events: auto; /* Allow interaction with text content */
  }
</style>