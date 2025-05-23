<script setup lang="ts">
  import { ref, onMounted, onUnmounted, inject, watch, type Ref } from "vue";
  import * as THREE from "three";

  interface Location {
    name: string;
    lat: number;
    lng: number;
  }

  interface Props {
    locations: Location[];
  }

  const props = defineProps<Props>();
  const scene = inject<Ref<THREE.Scene | null>>("scene");
  const markers = ref<THREE.Group[]>([]);
  const hoveredMarker = ref<number | null>(null);

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

  const createMarkers = () => {
    if (!scene?.value) return;

    // Find the globe group
    const globeGroup = scene.value.children.find(
      (child) => child instanceof THREE.Group
    );
    if (!globeGroup) return;

    // Clear existing markers
    markers.value.forEach((marker) => {
      globeGroup.remove(marker);
      marker.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.geometry.dispose();
          if (child.material instanceof THREE.Material) {
            child.material.dispose();
          }
        }
      });
    });
    markers.value = [];

    // Create new markers
    props.locations.forEach((location, index) => {
      // Use a slightly larger radius to ensure markers are visible above the globe
      const position = latLngToVector3(location.lat, location.lng, 0.92);
      const marker = new THREE.Group();

      // Create marker sphere with smaller size
      const geometry = new THREE.SphereGeometry(0.02, 14, 14);
      const material = new THREE.MeshBasicMaterial({
        color: "#d5aa0f",
        transparent: true,
        opacity: 0.9,
      });
      const sphere = new THREE.Mesh(geometry, material);
      marker.add(sphere);

      // Add glow effect with reduced intensity and radius for smaller shadow
      const light = new THREE.PointLight("#d5aa0f", 0.1, 0.08);
      marker.add(light);

      marker.position.copy(position);
      globeGroup.add(marker);
      markers.value.push(marker);

      // Log marker creation for debugging
      console.log(`Created marker for ${location.name} at position:`, position);
    });
  };

  // Watch for scene changes
  watch(
    () => scene?.value,
    (newScene) => {
      if (newScene) {
        createMarkers();
      }
    },
    { immediate: true }
  );

  // Watch for location changes
  watch(
    () => props.locations,
    () => {
      if (scene?.value) {
        createMarkers();
      }
    },
    { deep: true }
  );

  onUnmounted(() => {
    if (!scene?.value) return;

    const globeGroup = scene.value.children.find(
      (child) => child instanceof THREE.Group
    );
    if (!globeGroup) return;

    markers.value.forEach((marker) => {
      globeGroup.remove(marker);
      marker.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.geometry.dispose();
          if (child.material instanceof THREE.Material) {
            child.material.dispose();
          }
        }
      });
    });
    markers.value = [];
  });

  const handleMarkerHover = (index: number, isHovered: boolean) => {
    if (isHovered) {
      hoveredMarker.value = index;
      document.body.style.cursor = "pointer";
      const marker = markers.value[index];
      if (marker) {
        const sphere = marker.children[0] as THREE.Mesh;
        const material = sphere.material as THREE.MeshBasicMaterial;
        material.color.set("#FFC857");
      }
    } else {
      hoveredMarker.value = null;
      document.body.style.cursor = "auto";
      const marker = markers.value[index];
      if (marker) {
        const sphere = marker.children[0] as THREE.Mesh;
        const material = sphere.material as THREE.MeshBasicMaterial;
        material.color.set("#d5aa0f");
      }
    }
  };
</script>
