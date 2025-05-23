<script setup lang="ts">
  import { ref, onMounted, onUnmounted, shallowRef, provide, nextTick } from "vue";
  import * as THREE from "three";
  import { TextureLoader } from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import Markers from "./markers.vue";
  import AnimatedLines from "./animated-lines.vue";

  interface Location {
    name: string;
    lat: number;
    lng: number;
  }

  interface Connection {
    from: string;
    to: string;
  }

  interface Props {
    locations: Location[];
    connections: Connection[];
  }

  const props = defineProps<Props>();
  const containerRef = ref<HTMLElement | null>(null);
  const scene = shallowRef<THREE.Scene | null>(null);
  const camera = shallowRef<THREE.PerspectiveCamera | null>(null);
  const renderer = shallowRef<THREE.WebGLRenderer | null>(null);
  const sphere = shallowRef<THREE.Mesh | null>(null);
  const earthTexture = shallowRef<THREE.Texture | null>(null);
  const globeGroup = shallowRef<THREE.Group | null>(null);
  const controls = shallowRef<OrbitControls | null>(null);
  const isDragging = ref(false);

  // Provide scene and camera to child components
  provide("scene", scene);

  // Function to focus camera on markers
  const focusOnMarkers = () => {
    if (!props.locations.length || !controls.value) return;

    // Calculate the average position of all markers
    const positions = props.locations.map(location => {
      const phi = (90 - location.lat) * (Math.PI / 180);
      const theta = (location.lng + 180) * (Math.PI / 180);
      return new THREE.Vector3(
        -Math.sin(phi) * Math.cos(theta),
        Math.cos(phi),
        Math.sin(phi) * Math.sin(theta)
      );
    });

    const center = positions.reduce((acc, pos) => acc.add(pos), new THREE.Vector3())
      .divideScalar(positions.length);

    // Set the camera to look at the center of markers
    if (camera.value) {
      camera.value.position.copy(center.multiplyScalar(2.74));
      camera.value.lookAt(0, 0, 0);
    }

    // Update controls target
    controls.value.target.set(0, 0, 0);
    controls.value.update();
  };

  const initScene = async () => {
    if (!containerRef.value) return;

    // Create scene
    scene.value = new THREE.Scene();

    // Create camera
    camera.value = new THREE.PerspectiveCamera(
      45,
      containerRef.value.clientWidth / containerRef.value.clientHeight,
      0.1,
      1000
    );
    camera.value.position.z = 2.5;

    // Create renderer
    renderer.value = new THREE.WebGLRenderer({
      antialias: true,
      alpha: true,
      powerPreference: "high-performance",
    });
    renderer.value.setSize(
      containerRef.value.clientWidth,
      containerRef.value.clientHeight
    );
    containerRef.value.appendChild(renderer.value.domElement);

    // Create a group for the globe and its elements
    globeGroup.value = new THREE.Group();
    scene.value.add(globeGroup.value);

    // Load texture
    const textureLoader = new TextureLoader();
    earthTexture.value = await textureLoader.loadAsync("/glob3d.jpg");

    // Create sphere
    const geometry = new THREE.SphereGeometry(0.85, 32, 32);
    const material = new THREE.MeshPhongMaterial({
      map: earthTexture.value,
      bumpMap: earthTexture.value,
      bumpScale: 0.05,
      specular: new THREE.Color("grey"),
      shininess: 5,
    });
    sphere.value = new THREE.Mesh(geometry, material);
    globeGroup.value.add(sphere.value);

    // Add lights
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.value.add(ambientLight);

    const pointLight = new THREE.PointLight(0xffffff, 1);
    pointLight.position.set(5, 3, 5);
    scene.value.add(pointLight);

    // Add OrbitControls after scene is fully set up
    if (camera.value && renderer.value) {
      controls.value = new OrbitControls(
        camera.value,
        renderer.value.domElement
      );
      controls.value.enableDamping = true;
      controls.value.dampingFactor = 0.05;
      controls.value.rotateSpeed = 0.5;
      controls.value.minDistance = 2;
      controls.value.maxDistance = 5;
      controls.value.enablePan = false;

      // Add event listeners for drag state
      controls.value.addEventListener("start", () => {
        isDragging.value = true;
      });
      controls.value.addEventListener("end", () => {
        isDragging.value = false;
      });
    }

    // Force initial render
    if (scene.value && camera.value && renderer.value) {
      renderer.value.render(scene.value, camera.value);
    }

    // Focus on markers after a short delay to ensure everything is loaded
    setTimeout(focusOnMarkers, 100);

    // Start animation
    animate();
  };

  let animationFrameId: number;

  const animate = () => {
    if (!scene.value || !camera.value || !renderer.value || !globeGroup.value)
      return;

    animationFrameId = requestAnimationFrame(animate);

    // Update controls
    if (controls.value) {
      controls.value.update();
    }

    // Rotate the globe when not being dragged
    if (!isDragging.value) {
      globeGroup.value.rotation.y += 0.0015;
    }

    // Render scene
    renderer.value.render(scene.value, camera.value);
  };

  const handleResize = () => {
    if (!containerRef.value || !camera.value || !renderer.value) return;

    camera.value.aspect =
      containerRef.value.clientWidth / containerRef.value.clientHeight;
    camera.value.updateProjectionMatrix();
    renderer.value.setSize(
      containerRef.value.clientWidth,
      containerRef.value.clientHeight
    );
  };

  onMounted(async () => {
    // Wait for next tick to ensure container is mounted
    await nextTick();
    
    // Initialize scene immediately
    await initScene();
    
    // Force a resize to ensure proper dimensions
    handleResize();
    
    // Add resize listener
    window.addEventListener("resize", handleResize);
  });

  onUnmounted(() => {
    window.removeEventListener("resize", handleResize);
    cancelAnimationFrame(animationFrameId);

    if (controls.value) {
      controls.value.dispose();
    }
    if (renderer.value) {
      renderer.value.dispose();
    }
    if (earthTexture.value) {
      earthTexture.value.dispose();
    }
    if (sphere.value) {
      sphere.value.geometry.dispose();
      (sphere.value.material as THREE.Material).dispose();
    }
  });
</script>

<template>
  <div
    class="globe-container"
    ref="containerRef"
  >
    <Markers :locations="locations" />
    <AnimatedLines
      :locations="locations"
      :connections="connections"
    />
  </div>
</template>

<style scoped>
  .globe-container {
    width: calc(100% + 30px);
    height: 100%;
    position: relative;
    background: transparent;
    cursor: grab;
    margin-left: -15px; /* Center the extra width */
  }
  .globe-container:active {
    cursor: grabbing;
  }
</style>
