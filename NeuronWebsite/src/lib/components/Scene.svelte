<script>
  import { T } from '@threlte/core';
  import { OrbitControls } from '@threlte/extras';
  import { tweened } from 'svelte/motion';
  import { cubicInOut } from 'svelte/easing';
  import { onMount } from 'svelte';
  import * as THREE from 'three';

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

  let model;
  onMount(async () => {
    const { GLTFLoader } = await import('three/examples/jsm/loaders/GLTFLoader');
    const loader = new GLTFLoader();
    
    try {
      const gltf = await new Promise((resolve, reject) => {
        loader.load(
          '/Neuron.glb',
          (gltf) => {
            model = gltf.scene;
            model.traverse((child) => {
              if (child.isMesh) {
                child.material.side = THREE.DoubleSide;
                child.castShadow = true;
                child.receiveShadow = true;
              }
            });
            
            // Position model at origin and scale up
            model.position.set(0, 0, 0);
            model.scale.set(2, 2, 2);
            model.rotation.set(0, Math.PI * 0.5, 0);
            
            // Calculate and log model boundaries
            const boundingBox = new THREE.Box3().setFromObject(model);
            console.log('Model information:', {
              position: model.position,
              rotation: model.rotation,
              scale: model.scale,
              boundaries: {
                min: boundingBox.min,
                max: boundingBox.max,
                size: boundingBox.getSize(new THREE.Vector3()),
                center: boundingBox.getCenter(new THREE.Vector3())
              }
            });

            // Log camera position
            console.log('Camera position:', $cameraPos);
            
            resolve(gltf);
          },
          (progress) => {
            console.log('Loading progress:', (progress.loaded / progress.total * 100) + '%');
          },
          reject
        );
      });
    } catch (error) {
      console.error('Error loading model:', error);
    }
  });

  // Add reactive statement to log camera position changes
  $: if ($cameraPos) {
    console.log('Camera position updated:', $cameraPos);
  }
</script>
<T.Scene>
  <T.Color attach="background" args={[0x202030]} />
  
  <T.PerspectiveCamera
    position={$cameraPos}
    fov={60}
    near={0.1}
    far={1000}
  >
    <OrbitControls 
      enableZoom={true}
      enableDamping={true}
      dampingFactor={0.05}
      target={[0, 0, 0]}
      maxPolarAngle={Math.PI * 0.85}
      minDistance={5}
      maxDistance={50}
    />
  </T.PerspectiveCamera>

  <!-- Main lighting -->
  <T.DirectionalLight 
    position={[10, 10, 10]} 
    intensity={2} 
    castShadow 
  />
  <T.DirectionalLight 
    position={[-10, 5, -10]} 
    intensity={1.5} 
  />
  <T.AmbientLight intensity={0.8} />
  <T.HemisphereLight 
    args={[0xffffbb, 0x080820, 1]} 
  />

  <!-- Debug helpers -->
  <T.GridHelper args={[50, 50]} position={[0, -2, 0]} />
  <T.AxesHelper args={[25]} />

  <!-- Model -->
  {#if model}
    <T.Group>
      <T.Object3D object={model} castShadow receiveShadow />
    </T.Group>
  {/if}
</T.Scene>

<style>
  :global(body) {
    margin: 0;
    background: #202030;
  }
</style>