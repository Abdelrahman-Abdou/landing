<script setup lang="ts">
  import { ref, onMounted, onUnmounted, inject, watch, type Ref } from "vue";
  import * as THREE from "three";

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
  const scene = inject<Ref<THREE.Scene | null>>("scene");
  const lines = ref<THREE.Line[]>([]);
  const animationFrameId = ref<number | null>(null);
  const lineProgress = ref<number[]>([]); // Track progress for each line

  // Convert latitude and longitude to 3D coordinates on a sphere
  function latLngToVector3(
    lat: number,
    lng: number,
    radius: number
  ): THREE.Vector3 {
    // Convert to radians
    const phi = (90 - lat) * (Math.PI / 180);
    const theta = (lng + 180) * (Math.PI / 180);

    // Calculate 3D coordinates
    // Note: In Three.js, Y is up, so we swap Y and Z
    const x = -radius * Math.sin(phi) * Math.cos(theta);
    const z = radius * Math.sin(phi) * Math.sin(theta);
    const y = radius * Math.cos(phi);

    return new THREE.Vector3(x, y, z);
  }

  const createLines = () => {
    if (!scene?.value) return;

    // Find the globe group
    const globeGroup = scene.value.children.find(
      (child) => child instanceof THREE.Group
    );
    if (!globeGroup) return;

    // Clear existing lines
    lines.value.forEach((line) => {
      globeGroup.remove(line);
      line.geometry.dispose();
      if (line.material instanceof THREE.Material) {
        line.material.dispose();
      }
    });
    lines.value = [];

    // Create new lines
    props.connections.forEach((connection, index) => {
      const fromLocation = props.locations.find(
        (loc) => loc.name === connection.from
      );
      const toLocation = props.locations.find(
        (loc) => loc.name === connection.to
      );

      if (!fromLocation || !toLocation) return;

      const start = latLngToVector3(fromLocation.lat, fromLocation.lng, 0.92);
      const end = latLngToVector3(toLocation.lat, toLocation.lng, 0.92);

      // Create curved path
      const mid = new THREE.Vector3()
        .addVectors(start, end)
        .multiplyScalar(0.5);
      const distance = start.distanceTo(end);
      mid.normalize().multiplyScalar(0.92 + distance * 0.15);

      const curve = new THREE.QuadraticBezierCurve3(start, mid, end);
      const points = curve.getPoints(50);
      const geometry = new THREE.BufferGeometry().setFromPoints(points);

      // Add color attribute for animation
      const colors = new Float32Array(points.length * 4);
      for (let i = 0; i < points.length; i++) {
        colors[i * 4] = 0.835;     // R (d5/255)
        colors[i * 4 + 1] = 0.667; // G (aa/255)
        colors[i * 4 + 2] = 0.059; // B (0f/255)
        colors[i * 4 + 3] = 0;     // A (start invisible)
      }
      geometry.setAttribute('color', new THREE.BufferAttribute(colors, 4));

      // Create solid line material
      const material = new THREE.LineBasicMaterial({
        color: "#d5aa0f",
        transparent: true,
        opacity: 0.8,
        linewidth: 2,
        vertexColors: true
      });

      const line = new THREE.Line(geometry, material);
      globeGroup.add(line);
      lines.value.push(line);
      lineProgress.value[index] = 0; // Initialize progress for this line

      // Log line creation for debugging
      console.log(`Created line from ${connection.from} to ${connection.to}`);
    });

    // Start animation
    animate();
  };

  // Watch for scene changes
  watch(
    () => scene?.value,
    (newScene) => {
      if (newScene) {
        createLines();
      }
    },
    { immediate: true }
  );

  // Watch for connections and locations changes
  watch(
    [() => props.connections, () => props.locations],
    () => {
      if (scene?.value) {
        createLines();
      }
    },
    { deep: true }
  );

  const animate = () => {
    const time = Date.now() * 0.001;

    lines.value.forEach((line, index) => {
      const material = line.material as THREE.LineBasicMaterial;
      
      // Update line progress
      lineProgress.value[index] = (time * 0.5 + index * 0.2) % 1;
      
      // Get the geometry
      const geometry = line.geometry as THREE.BufferGeometry;
      const positions = geometry.attributes.position.array as Float32Array;
      
      // Calculate visible points based on progress
      const totalPoints = positions.length / 3;
      const visiblePoints = Math.floor(totalPoints * lineProgress.value[index]);
      
      // Update opacity based on progress
      const opacity = Math.sin(time + index * 0.2) * 0.3 + 0.7;
      material.opacity = opacity;
      
      // Update line visibility
      for (let i = 0; i < totalPoints; i++) {
        const isVisible = i <= visiblePoints;
        const alpha = isVisible ? 1 : 0;
        if (geometry.attributes.color) {
          geometry.attributes.color.setXYZ(i, 1, 1, 1);
          geometry.attributes.color.setW(i, alpha);
        }
      }
      
      if (geometry.attributes.color) {
        geometry.attributes.color.needsUpdate = true;
      }
    });

    animationFrameId.value = requestAnimationFrame(animate);
  };

  onUnmounted(() => {
    if (!scene?.value) return;

    const globeGroup = scene.value.children.find(
      (child) => child instanceof THREE.Group
    );
    if (!globeGroup) return;

    if (animationFrameId.value) {
      cancelAnimationFrame(animationFrameId.value);
    }

    lines.value.forEach((line) => {
      globeGroup.remove(line);
      line.geometry.dispose();
      if (line.material instanceof THREE.Material) {
        line.material.dispose();
      }
    });
    lines.value = [];
  });
</script>
