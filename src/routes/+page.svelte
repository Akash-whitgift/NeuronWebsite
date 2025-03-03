<script>
  import { Canvas } from '@threlte/core';
  import Scene from '$lib/components/Scene.svelte';
  import { spring } from 'svelte/motion';
  import { onMount } from 'svelte';
  import { fade, blur, scale } from 'svelte/transition';

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
  
  // Add popup information for each section with detailed modal content
  const popupInfo = [
    { 
      title: "Neuron", 
      content: "A specialized cell that transmits nerve impulses; the building block of the nervous system.", 
      position: { x: "20%", y: "30%" },
      detailedContent: `
        <h3 class="text-xl mb-4">Neuron Structure and Function</h3>
        <p class="mb-4">Neurons are the fundamental units of the nervous system, specialized for the transmission of information through electrical and chemical signals.</p>
        <p class="mb-4">These highly specialized cells consist of a cell body (soma), dendrites, and an axon. Each part plays a crucial role in neuron function:</p>
        <ul class="mb-4 list-disc pl-5">
          <li>The <strong>soma</strong> contains the nucleus and other cellular machinery</li>
          <li><strong>Dendrites</strong> receive signals from other neurons</li>
          <li>The <strong>axon</strong> transmits electrical impulses away from the cell body</li>
          <li><strong>Synaptic terminals</strong> release neurotransmitters to communicate with other cells</li>
        </ul>
        <p>An adult human brain contains approximately 86 billion neurons, forming an incredibly complex network that facilitates everything from basic reflexes to abstract thought.</p>
      `
    },
    { 
      title: "Soma", 
      content: "The cell body containing the nucleus and organelles that maintain the neuron's function.", 
      position: { x: "25%", y: "35%" },
      detailedContent: `
        <h3 class="text-xl mb-4">The Soma (Cell Body)</h3>
        <p class="mb-4">The soma, or cell body, is the central processing unit of the neuron. It contains the nucleus, which houses the genetic material, and numerous organelles that maintain cell function.</p>
        <p class="mb-4">Key components of the soma include:</p>
        <ul class="mb-4 list-disc pl-5">
          <li>The <strong>nucleus</strong> - contains DNA and controls cellular activities</li>
          <li><strong>Mitochondria</strong> - produce energy through cellular respiration</li>
          <li><strong>Endoplasmic reticulum</strong> - involved in protein synthesis</li>
          <li><strong>Golgi apparatus</strong> - processes and packages proteins</li>
        </ul>
        <p class="mb-4">The soma integrates signals received from the dendrites and initiates action potentials that travel down the axon.</p>
        <p>Damage to the soma often leads to cell death, as it contains the essential machinery needed for neuron survival.</p>
      `
    },
    { 
      title: "Dendrites", 
      content: "Branch-like structures that receive signals from other neurons and transmit them to the soma.", 
      position: { x: "75%", y: "30%" },
      detailedContent: `
        <h3 class="text-xl mb-4">Dendrites: The Information Receivers</h3>
        <p class="mb-4">Dendrites are branched extensions of the neuron that receive signals from other neurons. Their tree-like structure provides an extensive surface area for forming connections.</p>
        <p class="mb-4">Key characteristics of dendrites include:</p>
        <ul class="mb-4 list-disc pl-5">
          <li><strong>Dendritic spines</strong> - small protrusions that form synapses with axon terminals</li>
          <li><strong>Graded potentials</strong> - local changes in membrane potential that can either excite or inhibit the neuron</li>
          <li><strong>Signal integration</strong> - dendrites sum incoming signals both spatially and temporally</li>
        </ul>
        <p class="mb-4">A single neuron can have thousands of dendrites, allowing it to receive input from many other neurons simultaneously.</p>
        <p>Dendrites actively participate in information processing through mechanisms like dendritic spikes and calcium signaling.</p>
      `
    },
    { 
      title: "Axon", 
      content: "A long fiber that carries electrical impulses away from the soma to other neurons.", 
      position: { x: "25%", y: "40%" },
      detailedContent: `
        <h3 class="text-xl mb-4">The Axon: The Information Highway</h3>
        <p class="mb-4">The axon is a long, slender projection that conducts electrical impulses away from the neuron's cell body. These signals, called action potentials, travel along the axon to the synaptic terminals.</p>
        <p class="mb-4">Important features of axons include:</p>
        <ul class="mb-4 list-disc pl-5">
          <li>The <strong>axon hillock</strong> - where action potentials are initiated</li>
          <li><strong>Myelin sheath</strong> - an insulating layer that increases signal conduction speed</li>
          <li><strong>Nodes of Ranvier</strong> - gaps in the myelin where action potentials are regenerated</li>
          <li><strong>Axonal transport</strong> - the movement of materials along the axon</li>
        </ul>
        <p class="mb-4">Some axons can be remarkably long - motor neurons controlling muscles in the feet have axons that extend from the spinal cord all the way to the foot!</p>
        <p>Damage to axons, such as in spinal cord injuries, can disrupt critical neural pathways and lead to loss of function.</p>
      `
    },
    { 
      title: "Synaptic Terminals", 
      content: "The end points of axons where neurotransmitters are released to communicate with other cells.", 
      position: { x: "75%", y: "60%" },
      detailedContent: `
        <h3 class="text-xl mb-4">Synaptic Terminals: The Communication Points</h3>
        <p class="mb-4">Synaptic terminals (also called axon terminals or terminal buttons) are specialized structures at the end of axons where neurotransmitters are released to communicate with target cells.</p>
        <p class="mb-4">The process of synaptic transmission involves several steps:</p>
        <ol class="mb-4 list-decimal pl-5">
          <li>An action potential arrives at the synaptic terminal</li>
          <li>Voltage-gated calcium channels open, allowing calcium to enter</li>
          <li>Calcium triggers the fusion of synaptic vesicles with the membrane</li>
          <li>Neurotransmitters are released into the synaptic cleft</li>
          <li>Neurotransmitters bind to receptors on the postsynaptic cell</li>
        </ol>
        <p class="mb-4">There are two main types of synapses:</p>
        <ul class="mb-4 list-disc pl-5">
          <li><strong>Chemical synapses</strong> - use neurotransmitters to transmit signals</li>
          <li><strong>Electrical synapses</strong> - allow direct ion flow between connected cells</li>
        </ul>
        <p>Synaptic terminals are highly plastic, meaning they can strengthen or weaken over time, forming the basis for learning and memory.</p>
      `
    }
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

  // Modal state variables
  let isModalOpen = false;
  let currentModalContent = { title: "", content: "", detailedContent: "" };

  onMount(() => {
    const updateProgress = () => {
      if (!scrollContainer) return;
      const maxScroll = scrollContainer.scrollHeight - scrollContainer.clientHeight;
      const currentProgress = maxScroll > 0 ? scrollContainer.scrollTop / maxScroll : 0;
      progress.set(currentProgress);
      
      // Calculate the section with a cap at 4 (the last valid section)
      const newSection = Math.min(4, Math.floor(currentProgress * 5));
      
      // Special case: If we're very close to the bottom, always show the last section
      if (scrollContainer.scrollTop + scrollContainer.clientHeight >= scrollContainer.scrollHeight - 20) {
        // If we're at the bottom, force the last section
        if (currentSection !== 4) {
          previousSection = currentSection;
          currentSection = 4;
          handleSectionChange();
        }
      } else if (newSection !== currentSection) {
        // Regular section change handling
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
    
    // Skip popup for the first section (index 0)
    if (targetSection === 0) {
      popupVisible = false;
      return;
    }
    
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

  // Function to handle popup click and show modal
  function handlePopupClick(section) {
    currentModalContent = popupInfo[section];
    isModalOpen = true;
  }

  // Function to close the modal
  function closeModal() {
    isModalOpen = false;
  }

  // Close modal when clicking outside or pressing Escape
  function handleModalBackdropClick(event) {
    // Only close if clicking directly on the backdrop, not child elements
    if (event.target === event.currentTarget) {
      closeModal();
    }
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
      <h1 class="text-5xl font-bold mb-4">The Structure of a Neuron</h1>
      <p class="text-center">Scroll to explore</p>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 ml-8 mb-16">
      <h2 class="text-5xl font-bold">Cell Body (Soma)</h2>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 mr-8 mb-16 ml-auto">
      <h2 class="text-5xl font-bold">Dendrites</h2>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 ml-8 mb-16">
      <h2 class="text-5xl font-bold">Axon</h2>
    </div>
  </section>

  <section class="section">
    <div class="max-w-2xl p-4 rounded backdrop-blur-sm bg-black/30 mr-8 mb-16 ml-auto">
      <h2 class="text-5xl font-bold">Synaptic Terminals</h2>
      <p class='p4 mr-8'>Created by Arjun, Code available here: <a href='https://github.com/Akash-whitgift/NeuronWebsite'>Github</a></p>
    </div>
  </section>
</div>

<!-- Add popup information box with enhanced transitions, click handler, and indicator -->
{#if currentSection > 0 && popupVisible}
  <div class="popup" 
       style="--popup-x: {currentPopupPosition.x}; --popup-y: {currentPopupPosition.y};"
       on:click={() => handlePopupClick(currentSection)}>
    
    <div class="popup-content" class:show-content={popupAnimationComplete} class:hide-content={animatingOut}>
      {#if popupAnimationComplete && !animatingOut}
        <!-- Click indicator with external/expand icon -->
        <div class="click-indicator">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
            <path d="M11 3a1 1 0 100 2h2.586l-6.293 6.293a1 1 0 101.414 1.414L15 6.414V9a1 1 0 102 0V4a1 1 0 00-1-1h-5z" />
            <path d="M5 5a2 2 0 00-2 2v8a2 2 0 002 2h8a2 2 0 002-2v-3a1 1 0 10-2 0v3H5V7h3a1 1 0 000-2H5z" />
          </svg>
         
        </div>

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

<!-- Modal component with increased size and larger text -->
{#if isModalOpen}
  <div 
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/40 backdrop-blur-sm"
    on:click={handleModalBackdropClick}
    transition:fade={{ duration: 200 }}
  >
    <div 
      class="modal-container relative w-[90%] md:w-[80%] h-[75%] md:w-[80] rounded-lg overflow-y-auto"
      transition:scale={{ duration: 300, start: 0.95 }}
    >
      <!-- Close button -->
      <button 
        class="absolute top-5 right-5 text-gray-400 hover:text-white"
        on:click={closeModal}
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        </svg>
      </button>

      <!-- Modal content with larger text -->
      <div class="p-8 md:p-10">
        <h2 class="text-3xl md:text-4xl font-bold mb-6 text-blue-400 border-b border-blue-500/30 pb-3">
          {currentModalContent.title}
        </h2>
        
        <div class="modal-content text-white/90 text-lg" transition:fade={{ delay: 150, duration: 300 }}>
          <!-- Use the HTML content from detailedContent -->
          {@html currentModalContent.detailedContent}
        </div>
      </div>
    </div>
  </div>
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
    cursor: pointer;
    transition: transform 0.2s ease;
  }
  
  .popup:hover {
    transform: translate(-50%, -50%) scale(1.02);
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

  /* Updated modal styles with larger text */
  .modal-container {
    background-color: rgba(0, 0, 0, 0.8);
    border: 1px solid rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(5px);
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
  }
  
  .modal-content {
    line-height: 1.7;
  }
  
  .modal-content h3 {
    color: #63b3ed;
    font-size: 1.5rem;
    margin-bottom: 1.5rem;
  }
  
  .modal-content p {
    margin-bottom: 1.25rem;
  }
  
  .modal-content strong {
    color: #a3bffa;
  }
  
  .modal-content ul, .modal-content ol {
    margin-left: 1rem;
    margin-bottom: 1.5rem;
  }
  
  .modal-content li {
    margin-bottom: 0.5rem;
  }

  /* Click indicator styles */
  .click-indicator {
    position: absolute;
    top: 20px;
    right: 20px;
    display: flex;
    align-items: center;
    color: #4d9aff;
    font-size: 0.8rem;
    opacity: 0.8;
    transition: all 0.2s ease;
    gap: 5px;
    background-color: rgba(0, 0, 0, 0.3);
    padding: 3px 6px;
    border-radius: 12px;
  }
  
  .popup:hover .click-indicator {
    opacity: 1;
    transform: scale(1.05);
  }
  
  .click-indicator svg {
    animation: pulse 2s infinite;
  }
  
  @keyframes pulse {
    0% { opacity: 0.7; transform: scale(1); }
    50% { opacity: 1; transform: scale(1.1); }
    100% { opacity: 0.7; transform: scale(1); }
  }
</style>

<svelte:window on:keydown={(e) => e.key === 'Escape' && closeModal()} />