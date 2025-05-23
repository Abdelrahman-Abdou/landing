<template>
  <div class="w-full h-[600px]">
    <Globe
      :locations="locations"
      :connections="connections"
    />
  </div>
</template>

<script setup lang="ts">
  import Globe from "./Globe.vue";

  // Static markers for specified countries including USA
  const locations = [
    { name: "UAE (Dubai)", lat: 25.2048, lng: 55.2708 },
    { name: "Egypt (Cairo)", lat: 30.0444, lng: 31.2357 },
    { name: "Jordan (Amman)", lat: 31.9454, lng: 35.9284 },
    { name: "Saudi Arabia (Riyadh)", lat: 24.7136, lng: 46.6753 },
    { name: "Sweden (Stockholm)", lat: 59.3293, lng: 18.0686 },
    { name: "USA (Washington D.C.)", lat: 38.9072, lng: -77.0369 },
  ];

  const connections = [
    { from: "UAE (Dubai)", to: "Egypt (Cairo)" },
    { from: "Egypt (Cairo)", to: "Jordan (Amman)" },
    { from: "Jordan (Amman)", to: "Saudi Arabia (Riyadh)" },
    { from: "Saudi Arabia (Riyadh)", to: "UAE (Dubai)" },
    { from: "Sweden (Stockholm)", to: "Egypt (Cairo)" },
    { from: "UAE (Dubai)", to: "Sweden (Stockholm)" },
    { from: "Jordan (Amman)", to: "Sweden (Stockholm)" },
    { from: "USA (Washington D.C.)", to: "UAE (Dubai)" },
    { from: "USA (Washington D.C.)", to: "Sweden (Stockholm)" },
    { from: "Egypt (Cairo)", to: "USA (Washington D.C.)" },
  ];
</script>

<style scoped>
  .w-full {
    width: 100%;
  }

  .h-\[600px\] {
    height: 600px;
  }
</style>
