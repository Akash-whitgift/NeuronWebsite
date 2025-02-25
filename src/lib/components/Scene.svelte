<script>
  import { tweened } from 'svelte/motion';
  import { cubicInOut } from 'svelte/easing';
  import { onMount, onDestroy } from 'svelte';
  import { Box3, Vector3, PerspectiveCamera, Scene, Color, GridHelper, AxesHelper, AmbientLight, DirectionalLight, Object3D, WebGLRenderer } from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
  import { Canvas } from '@threlte/core';

  export let progress = 0;

  const cameraPositions = [
    [15, 10, 20],    // Initial view - further back
    [8, 5, 12],      // Soma
    [10, 5, 10],     // Dendrites
    [-10, 5, 10],    // Axon
    [0, -8, 12]      // Synaptic terminals
  ];

  $: currentSection = Math.floor(progress * 4);
  $: sectionProgress = (progress * 4) % 1;

  const cameraPos = tweened(cameraPositions[0], {
    duration: 2000,
    easing: cubicInOut
  });

  $: cameraPos.set(cameraPositions[currentSection]);

  let model;
  let scene;
  let camera;
  let renderer;
  let controls;

  onMount(() => {
    scene = new Scene();
    camera = new PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(...cameraPositions[0]);

    renderer = new WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableZoom = true;
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.target.set(0, 0, 0);
    controls.maxPolarAngle = Math.PI * 0.85;
    controls.minDistance = 1;
    controls.maxDistance = 50;

    const gridHelper = new GridHelper(50, 50);
    gridHelper.position.set(0, -2, 0);
    scene.add(gridHelper);

    const axesHelper = new AxesHelper(25);
    scene.add(axesHelper);

    const ambientLight = new AmbientLight(1);
    scene.add(ambientLight);

    const directionalLight = new DirectionalLight(0xffffff, 1);
    directionalLight.position.set(5, 10, 5);
    directionalLight.castShadow = true;
    scene.add(directionalLight);

    if (model) {
      scene.add(model);
    }

    animate();
  });

  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  }

  onDestroy(() => {
    document.body.removeChild(renderer.domElement);
  });

  // Add reactive statement to log camera position changes
  $: if ($cameraPos) {
    console.log('Camera position updated:', $cameraPos);
  }
</script>
<Canvas>
  <!-- The Three.js scene will be rendered here -->
</Canvas>

<style>
  :global(body) {
    margin: 0;
    background: #202030;
  }
</style>