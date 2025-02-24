<script>
  import { Canvas } from '@threlte/core';
  import Scene from '$lib/components/Scene.svelte';
  import { spring } from 'svelte/motion';
  import { onMount } from 'svelte';

  let scrollY;
  let innerHeight;
  const progress = spring(0);

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
  <section class="h-screen flex items-center justify-center text-white p-8">
    <h1 class="text-4xl font-bold">The Structure of a Neuron</h1>
  </section>

  <section class="h-screen flex items-center justify-center text-white p-8">
    <div class="max-w-2xl">
      <h2 class="text-3xl mb-4">Cell Body (Soma)</h2>
      <p>The soma is the cell body of the neuron, containing the nucleus and other vital organelles.</p>
    </div>
  </section>

  <section class="h-screen flex items-center justify-center text-white p-8">
    <div class="max-w-2xl">
      <h2 class="text-3xl mb-4">Dendrites</h2>
      <p>Dendrites are branch-like extensions that receive signals from other neurons.</p>
    </div>
  </section>

  <section class="h-screen flex items-center justify-center text-white p-8">
    <div class="max-w-2xl">
      <h2 class="text-3xl mb-4">Axon</h2>
      <p>The axon is a long fiber that conducts electrical impulses away from the soma to other neurons.</p>
    </div>
  </section>

  <section class="h-screen flex items-center justify-center text-white p-8">
    <div class="max-w-2xl">
      <h2 class="text-3xl mb-4">Synaptic Terminals</h2>
      <p>These terminals release neurotransmitters to communicate with other neurons.</p>
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
</style>