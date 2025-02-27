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

  const cameraPositions = [
    // [10, 15, 20],    // Initial view - further back
    [5, 5, 5],      // Soma
    [5, 3, 5],     // Dendrites
    [-5, 2, 2],    // Axon
    [0, 5, -10]      // Synaptic terminals
  ];

  const focusTargets = [
    // [0, 0, 0],       // Initial view - origin
    [0, 0, 2],       // Soma
    [2.5, 0.5, 4],       // Dendrites
    [0, 0, -1],      // Axon
    [0, 0, -4]        // Synaptic terminals
  ];

  const fovs = [ 20, 50, 45, 40, 30]; // FOV values for each section

  $: currentSection = Math.floor(progress * 4);
  $: sectionProgress = (progress * 4) % 1;

  const cameraPos = tweened(cameraPositions[0], {
    duration: 2000,
    easing: cubicInOut
  });

  const fov = tweened(fovs[0], {
    duration: 2000,
    easing: cubicInOut
  });

  const targetPos = tweened(focusTargets[0], {
    duration: 2000,
    easing: cubicInOut
  });

  $: cameraPos.set(cameraPositions[currentSection]);
  $: fov.set(fovs[currentSection]);
  $: targetPos.set(focusTargets[currentSection]);

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
  });

  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
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