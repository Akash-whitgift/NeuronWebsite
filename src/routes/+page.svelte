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
  let sceneComponent;
  
  // Local state for screen points instead of using a store
  let screenPoints = [
    { x: 0, y: 0 }, // Overall neuron 
    { x: 0, y: 0 }, // Soma
    { x: 0, y: 0 }, // Dendrites
    { x: 0, y: 0 }, // Axon
    { x: 0, y: 0 }  // Synaptic terminals
  ];
  
  // Callback function to receive updated points from Scene
  function handlePointsUpdate(points) {
    screenPoints = points;
    // Log to see what coordinates we're getting
    console.log('Screen points updated:', points);
  }
  
  // Add popup information for each section
  const popupInfo = [
    { title: "Neuron", content: "A specialized cell that transmits nerve impulses; the building block of the nervous system.", position: { x: "20%", y: "30%" } },
    { title: "Soma", content: "The cell body containing the nucleus and organelles that maintain the neuron's function.", position: { x: "25%", y: "35%" } },
    { title: "Dendrites", content: "Branch-like structures that receive signals from other neurons and transmit them to the soma.", position: { x: "75%", y: "30%" } },
    { title: "Axon", content: "A long fiber that carries electrical impulses away from the soma to other neurons.", position: { x: "25%", y: "40%" } },
    { title: "Synaptic Terminals", content: "The end points of axons where neurotransmitters are released to communicate with other cells.", position: { x: "75%", y: "60%" } }
  ];

  // Current section index
  let currentSection = 0;
  
  // Enhanced animation states
  let popupVisible = false;
  let popupAnimationComplete = false;
  let previousSection = -1;
  let animatingOut = false;
  
  // Track current and next popup positions
  let currentPopupPosition = { x: "50%", y: "50%" };
  let targetSection = 0;

  onMount(() => {
    const updateProgress = () => {
      if (!scrollContainer) return;
      const maxScroll = scrollContainer.scrollHeight - scrollContainer.clientHeight;
      const currentProgress = maxScroll > 0 ? scrollContainer.scrollTop / maxScroll : 0;
      progress.set(currentProgress);
      
      // Calculate the current section (0-4 for each section)
      const newSection = Math.floor(currentProgress * 5);
      
      // If the section changed, handle the transition
      if (newSection !== currentSection) {
        previousSection = currentSection;
        currentSection = newSection;
        handleSectionChange();
      }
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

  // Enhanced function to handle section changes with animations
  function handleSectionChange() {
    // Store the target section for after animation completes
    targetSection = currentSection;
    
    // If popup is currently visible, animate it out first
    if (popupVisible) {
      animatePopupOut().then(() => {
        // Update position after closing but before reopening
        currentPopupPosition = { 
          x: popupInfo[targetSection].position.x, 
          y: popupInfo[targetSection].position.y 
        };
        
        // After animation out completes, animate the new one in
        currentSection = targetSection;
        animatePopupIn();
      });
    } else {
      // If no popup is visible, just set position and animate in
      currentPopupPosition = { 
        x: popupInfo[currentSection].position.x, 
        y: popupInfo[currentSection].position.y 
      };
      animatePopupIn();
    }
  }
  
  // Function to animate the popup in
  function animatePopupIn() {
    popupVisible = true;
    animatingOut = false;
    
    // After a delay, complete the animation
    setTimeout(() => {
      popupAnimationComplete = true;
    }, 600); // Delay for the lines to separate before showing content
  }
  
  // Function to animate the popup out
  function animatePopupOut() {
    return new Promise(resolve => {
      // Start the exit animation
      animatingOut = true;
      popupAnimationComplete = false;
      
      // Wait for the lines to close
      setTimeout(() => {
        // Then hide the popup completely
        popupVisible = false;
        resolve();
      }, 600);
    });
  }

  // Calculate the popup center point to use as line origin
  function getPopupCoordinates(popupInfo, index) {
    // Convert percentage strings to numbers (remove the % sign and divide by 100)
    const x = parseFloat(popupInfo[index].position.x) / 100;
    const y = parseFloat(popupInfo[index].position.y) / 100;
    return { x, y };
  }
</script>

<div class="fixed inset-0 z-0">
  <Canvas>
    <Scene progress={$progress} bind:this={sceneComponent} onPointsUpdate={handlePointsUpdate} />
  </Canvas>
</div>

<!-- Fixed the container structure - proper opening and closing tags -->
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
  <section class="section" style="height: 0; visibility: hidden;"></section>
</div>

<!-- Add popup information box with enhanced transitions -->
{#if currentSection >= 0 && popupVisible}
  <div class="popup" 
       style="--popup-x: {currentPopupPosition.x}; --popup-y: {currentPopupPosition.y};">
    
    <div class="popup-content" class:show-content={popupAnimationComplete} class:hide-content={animatingOut}>
      {#if popupAnimationComplete && !animatingOut}
        <div class="popup-title" in:fade={{ duration: 300, delay: 100 }} out:fade={{ duration: 200 }}>
          {popupInfo[currentSection].title}
        </div>
        <div class="popup-text" in:fade={{ duration: 300, delay: 200 }} out:fade={{ duration: 150 }}>
          {popupInfo[currentSection].content}
        </div>
      {/if}
    </div>
    
    <!-- Top and bottom lines that separate to reveal content -->
    <div class="popup-line top" class:separated={popupAnimationComplete} class:closing={animatingOut}></div>
    <div class="popup-line bottom" class:separated={popupAnimationComplete} class:closing={animatingOut}></div>
  </div>
  
  <!-- Dynamic connector line from popup to model point with transition -->
  {#if screenPoints && screenPoints[currentSection] && popupAnimationComplete && !animatingOut}
    <svg class="connector-svg" viewBox="0 0 100 100" preserveAspectRatio="none"
         in:fade={{ duration: 300 }}>
      <line
        x1="{parseFloat(currentPopupPosition.x)}"
        y1="{parseFloat(currentPopupPosition.y)}"
        x2="{screenPoints[currentSection].x}"
        y2="{screenPoints[currentSection].y}"
        class="svg-line"
      />
      
      <circle 
        cx="{screenPoints[currentSection].x}" 
        cy="{screenPoints[currentSection].y}" 
        r="0.5" 
        class="target-point" 
      />
    </svg>
  {/if}
{/if}

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
  
  /* Popup styling */
  .popup {
    position: fixed;
    left: var(--popup-x);
    top: var(--popup-y);
    transform: translate(-50%, -50%);
    z-index: 20; /* Make sure it's above the scroll container */
    width: 300px;
    /* No transition here to allow instant repositioning */
  }
  
  .popup-content {
    background-color: rgba(0, 0, 0, 0.8);
    border-radius: 10px;
    padding: 0;
    height: 0;
    overflow: hidden;
    transition: all 0.5s ease-out;
    border: 1px solid rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(5px);
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
  }
  
  .popup-content.show-content {
    padding: 20px;
    height: auto;
  }
  
  .popup-content.hide-content {
    padding: 0;
    height: 0;
    overflow: hidden;
  }
  
  .popup-title {
    color: #ffffff;
    font-size: 1.5em;
    font-weight: bold;
    margin-bottom: 10px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.3);
    padding-bottom: 8px;
  }
  
  .popup-text {
    color: #ffffff;
    font-size: 1em;
    line-height: 1.5;
  }
  
  /* Animated lines that separate */
  .popup-line {
    position: absolute;
    left: 50%;
    width: 0%;
    height: 2px;
    background: #4d9aff;
    transform: translateX(-50%);
    transition: all 0.5s cubic-bezier(0.68, -0.55, 0.27, 1.55);
  }
  
  .popup-line.top {
    top: 0;
  }
  
  .popup-line.bottom {
    bottom: 0;
  }
  
  .popup-line.separated {
    width: 100%;
  }
  
  .popup-line.closing {
    width: 0%;
  }
  
  /* Connector line styling */
  .connector-wrapper {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    pointer-events: none;
    z-index: -1;
  }
  
  .connector-svg {
    width: 100%;
    height: 100%;
  }
  
  .svg-line {
    stroke: #4d9aff;
    stroke-width: 0.3;
    stroke-dasharray: 1 1;
    stroke-linecap: round;
    animation: dashAnimation 2s linear infinite;
  }
  
  @keyframes dashAnimation {
    to {
      stroke-dashoffset: -4;
    }
  }

  /* Enhanced connector styling */
  .connector-svg {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 15; /* Above content but below popups */
    overflow: visible;
  }
  
  .svg-line {
    stroke: #4d9aff;
    stroke-width: 0.2;
    stroke-dasharray: 1 1;
    stroke-linecap: round;
    filter: drop-shadow(0 0 2px rgba(77, 154, 255, 0.7));
    animation: dashAnimation 1s linear infinite;
  }
  
  .target-point {
    fill: #4d9aff;
    stroke: white;
    stroke-width: 0.1;
    filter: drop-shadow(0 0 2px rgba(255, 255, 255, 0.7));
    animation: pulseSize 2s ease-in-out infinite alternate;
  }
  
  @keyframes dashAnimation {
    to {
      stroke-dashoffset: -2;
    }
  }
  
  @keyframes pulseSize {
    from { r: 0.3; }
    to { r: 0.6; }
  }
</style>