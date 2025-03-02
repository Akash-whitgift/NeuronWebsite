<script>
  import { tweened } from 'svelte/motion';
  import { cubicInOut } from 'svelte/easing';
  import { onMount, onDestroy } from 'svelte';
  import { Box3, Vector3, PerspectiveCamera, Scene, Color, AmbientLight, DirectionalLight, PointLight, HemisphereLight, WebGLRenderer, SphereGeometry, MeshBasicMaterial, Mesh } from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
  import { Canvas } from '@threlte/core';
  import { writable } from 'svelte/store';

  export let progress = 0;
  // Add a callback prop to send screen coordinates to parent
  export let onPointsUpdate = (points) => {};

  // 3D world coordinates for different parts of the neuron
  const neuronPoints = [
    [0, 0, 0],       // Overall neuron - center point
    [0, 0, 2],       // Soma
    [2.5, 0.5, 4],   // Dendrites
    [0, 0, -1],      // Axon
    [0, 0, -4]       // Synaptic terminals
  ];

  const cameraPositions = [
    [10, 15, 20],    // Initial view - further back
    [5, 5, 5],      // Soma
    [5, 3, 5],     // Dendrites
    [-5, 2, 2],    // Axon
    [0, 5, -10]      // Synaptic terminals
  ];

  const focusTargets = [
    [0, 0, 0],       // Initial view - origin
    [0, 0, 2],       // Soma
    [2.5, 0.5, 4],       // Dendrites
    [0, 0, -1],      // Axon
    [0, 0, -4]        // Synaptic terminals
  ];

  const fovs = [20, 50, 45, 40, 30]; // FOV values for each section

  // Calculate the initial section based on the initial progress value - FIX HERE
  const initialSection = Math.floor(progress * 5); // Changed from 4 to 5 to match +page.svelte
  $: currentSection = Math.floor(progress * 5); // Changed from 4 to 5
  $: sectionProgress = (progress * 5) % 1; // Changed from 4 to 5

  // Make sure the section index doesn't go out of bounds
  $: validSection = Math.min(currentSection, 4);

  // Initialize tweened stores with values based on the initial section
  const cameraPos = tweened(cameraPositions[initialSection], {
    duration: 2000,
    easing: cubicInOut
  });

  const fov = tweened(fovs[initialSection], {
    duration: 2000,
    easing: cubicInOut
  });

  const targetPos = tweened(focusTargets[initialSection], {
    duration: 2000,
    easing: cubicInOut
  });

  // Update these lines to use validSection instead of currentSection
  $: cameraPos.set(cameraPositions[Math.min(currentSection, 4)]);
  $: fov.set(fovs[Math.min(currentSection, 4)]);
  $: targetPos.set(focusTargets[Math.min(currentSection, 4)]);

  let model;
  let scene;
  let camera;
  let renderer;
  let controls;
  let impulse;

  const counter = writable(0);
  const opacity = writable(0);

  function updateCounter() {
    const maxScroll = document.body.scrollHeight - window.innerHeight;
    const scrollProgress = window.scrollY / maxScroll;
    const count = Math.min(100, Math.floor(scrollProgress * 100));
    counter.set(count);
    opacity.set(scrollProgress);
  }

  // Function to convert 3D world coordinates to 2D screen coordinates
  function worldToScreen(worldX, worldY, worldZ, camera, renderer) {
    if (!camera || !renderer) return { x: 50, y: 50 }; // Default to center if not available
    
    // Create a vector from the world coordinates
    const vector = new Vector3(worldX, worldY, worldZ);
    
    // Check if the point is behind the camera
    const posClone = camera.position.clone();
    const targetToPoint = vector.clone().sub(posClone);
    const cameraDirection = controls.target.clone().sub(posClone).normalize();
    const dotProduct = targetToPoint.normalize().dot(cameraDirection);
    
    // If the point is behind camera, use a default visible point
    if (dotProduct < 0) {
      return { x: 50, y: 50 };
    }
    
    // Project the 3D point to the 2D plane
    vector.project(camera);
    
    // Convert normalized device coordinates to screen percentage (0-100)
    return {
      x: ((vector.x + 1) / 2) * 100,
      y: ((1 - vector.y) / 2) * 100
    };
  }

  // Function to update all point screen positions
  function updateScreenPositions() {
    if (!camera || !renderer) return;
    
    const screenPositions = neuronPoints.map(point => {
      // Get pixel coordinates
      return worldToScreen(point[0], point[1], point[2], camera, renderer);
    });
    
    // Call the callback with updated screen positions
    onPointsUpdate(screenPositions);
  }

  onMount(() => {
    scene = new Scene();
    scene.background = new Color(0x101010); // Set background color

    camera = new PerspectiveCamera($fov, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(...cameraPositions[0]);

    renderer = new WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableZoom = true;
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.target.set(...focusTargets[0]);
    controls.maxPolarAngle = Math.PI * 0.85;
    controls.minDistance = 1;
    controls.maxDistance = 50;

    const ambientLight = new AmbientLight(0x404040, 1); // Soft white light
    scene.add(ambientLight);

    const directionalLight = new DirectionalLight(0xffffff, 1);
    directionalLight.position.set(5, 10, 5);
    directionalLight.castShadow = true;
    scene.add(directionalLight);

    const pointLight = new PointLight(0xffffff, 1, 100);
    pointLight.position.set(10, 10, 10);
    scene.add(pointLight);

    const hemisphereLight = new HemisphereLight(0xffffbb, 0x080820, 1);
    scene.add(hemisphereLight);

    const loader = new GLTFLoader();
    loader.load('/Neuron2.glb', (gltf) => {
      model = gltf.scene;
      scene.add(model);
      const box = new Box3().setFromObject(model);
      const center = box.getCenter(new Vector3());
      console.log('Model bounding box:', box);

      // Create the impulse
      const geometry = new SphereGeometry(0.1, 32, 32);
      const material = new MeshBasicMaterial({ color: 0xff0000 });
      impulse = new Mesh(geometry, material);
      impulse.position.set(0, 0, -1); // Start position of the impulse
      scene.add(impulse);

      animateImpulse();
    });

    animate();

    // Animate counter and opacity on scroll
    window.addEventListener('scroll', updateCounter);
    updateCounter();

    // Handle window resize
    window.addEventListener('resize', () => {
      if (renderer) {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
        updateScreenPositions();
      }
    });
    
    // Create visible markers at important points for debugging (optional)
    neuronPoints.forEach((point, index) => {
      const markerGeometry = new SphereGeometry(0.15, 16, 16); // Even larger size
      const markerMaterial = new MeshBasicMaterial({ 
        color: index === 0 ? 0xffff00 : // Yellow for center
               index === 1 ? 0xff00ff : // Purple for soma
               index === 2 ? 0x00ffff : // Cyan for dendrites
               index === 3 ? 0xff0000 : // Red for axon
                            0x00ff00,   // Green for terminals
        transparent: true,
        opacity: 0.8,
        emissive: index === 0 ? 0xffff00 : 
                  index === 1 ? 0xff00ff : 
                  index === 2 ? 0x00ffff :
                  index === 3 ? 0xff0000 :
                               0x00ff00,
        emissiveIntensity: 0.5
      });
      const marker = new Mesh(markerGeometry, markerMaterial);
      marker.position.set(point[0], point[1], point[2]);
      scene.add(marker);
      
      // Log the marker positions for debugging
      console.log(`Marker ${index} position:`, point);
    });

    // Do an initial position update
    setTimeout(updateScreenPositions, 100);
  });

  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
    updateScreenPositions(); // Update screen positions on each frame
  }

  function animateImpulse() {
    const duration = 2000; // Duration of the animation in milliseconds
    const start = { x: 0, y: 0, z: -1 };
    const end = { x: 0, y: 0, z: -4 };

    const tween = new TWEEN.Tween(start)
      .to(end, duration)
      .onUpdate(() => {
        impulse.position.set(start.x, start.y, start.z);
      })
      .repeat(Infinity)
      .start();
  }

  onDestroy(() => {
    document.body.removeChild(renderer.domElement);
    window.removeEventListener('scroll', updateCounter);
  });

  // Add reactive statement to log camera position changes
  $: if ($cameraPos && camera) {
    camera.position.set(...$cameraPos);
    camera.fov = $fov;
    camera.updateProjectionMatrix();
    controls.target.set(...$targetPos);
    updateScreenPositions();
    // console.log('Camera position updated:', $cameraPos);
  }
</script>

<Canvas>
  <!-- The Three.js scene will be rendered here -->
</Canvas>

<div class="fixed top-5 left-5 text-2xl text-white font-bold bg-black bg-opacity-50 p-2 rounded">
  {$counter}
</div>

<style>
  :global(body) {
    margin: 0;
    background: #101010;
  }
  :global(canvas) {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1; /* Ensure the canvas is in the background */
  }
</style>