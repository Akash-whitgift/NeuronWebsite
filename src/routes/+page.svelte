<script>
  import { Canvas } from '@threlte/core';
  import Scene from '$lib/components/Scene.svelte';
  import { spring } from 'svelte/motion';
  import { onMount } from 'svelte';
  import { fade, blur } from 'svelte/transition';

  let scrollY;
  let innerHeight;
  const progress = spring(0);
  let scrollContainer;
  
  onMount(() => {
    const updateProgress = () => {
      if (!scrollContainer) return;
      const maxScroll = scrollContainer.scrollHeight - scrollContainer.clientHeight;
      progress.set(maxScroll > 0 ? scrollContainer.scrollTop / maxScroll : 0);
    };

    if (scrollContainer) {
      scrollContainer.addEventListener('scroll', updateProgress);
      window.addEventListener('resize', updateProgress);
      updateProgress();
    }

    return () => {
      if (scrollContainer) {
        scrollContainer.removeEventListener('scroll', updateProgress);
      }
      window.removeEventListener('resize', updateProgress);
    };
  });
</script>

<div class="fixed inset-0 z-0">
  <Canvas>
    <Scene progress={$progress} />
  </Canvas>
</div>

<div class="fixed inset-0 z-10 h-screen w-screen scroll-container" bind:this={scrollContainer}>
  <section class="section">
    <div class="p-4 rounded backdrop-blur-sm bg-black/30 bg-opacity-75 ">
      <h1 class="text-4xl font-bold mb-4">The Structure of a Neuron</h1>
      <p class="text-center">Scroll to explore</p>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 ml-8 mb-16">
      <h2 class="text-3xl mb-4">Cell Body (Soma)</h2>
      <p>The soma contains the nucleus and other vital organelles.</p>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 mr-8 mb-16 ml-auto">
      <h2 class="text-3xl mb-4">Dendrites</h2>
      <p>Branch-like extensions that receive signals from other neurons.</p>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 ml-8 mb-16">
      <h2 class="text-3xl mb-4">Axon</h2>
      <p>The long fiber that conducts electrical impulses.</p>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 mr-8 mb-16 ml-auto">
      <h2 class="text-3xl mb-4">Synaptic Terminals</h2>
      <p>Release neurotransmitters to communicate with other neurons.</p>
    </div>
  </section>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    background: #000;
    color: white;
    overflow: hidden; /* Prevent double scrollbars */
  }
  
  /* We need to ensure the scrollable area can receive mouse events */
  :global(canvas) {
    pointer-events: none;
  }
  
  .scroll-container {
    overflow-y: auto;
    scroll-snap-type: y mandatory;
    scroll-behavior: smooth;
    pointer-events: auto;
  }

  .section {
    height: 100vh;
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    align-items: center;
    padding: 2rem;
    color: white;
    scroll-snap-align: start;
    scroll-snap-stop: always;
  }
  
  /* Alternate section alignment for visual interest */
  .section:nth-child(odd) {
    align-items: flex-start;
  }
  
  .section:nth-child(even) {
    align-items: flex-end;
  }
  
  .section:first-child {
    justify-content: center;
    align-items: center;
  }
</style>