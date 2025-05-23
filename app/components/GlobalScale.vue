<script setup lang="ts">
import Globe from "~/components/globe/Globe.vue";
</script>
<template>
  <section
    class="container grid grid-cols-1 lg:grid-cols-2 mt-12 mb-16 lg:gap-x-24"
  >
    <div class="col-span-1">
      <div class="text-left section-title mt-12">
        <span class="subtitle">what's stories on</span>
        <h2 class="heading-title">Global Scale</h2>
        <p class="description">
          For ambitious companies around the world, RunProf makes moving smart
          as simple, borderless, and programmable as the rest of the internet.
          we process hundreds of millions of codes each year for startups
        </p>
      </div>
    </div>
    <div class="globe col-span-1">
      <div class="globe-container">
        <Globe
          :locations="[
            { name: 'UAE (Dubai)', lat: 25.2048, lng: 55.2708 },
            { name: 'Egypt (Cairo)', lat: 30.0444, lng: 31.2357 },
            { name: 'Jordan (Amman)', lat: 31.9454, lng: 35.9284 },
            { name: 'Saudi Arabia (Riyadh)', lat: 24.7136, lng: 46.6753 },
            { name: 'Sweden (Stockholm)', lat: 59.3293, lng: 18.0686 },
            { name: 'USA (Washington D.C.)', lat: 38.9072, lng: -77.0369 },
          ]"
          :connections="[
            { from: 'UAE (Dubai)', to: 'Sweden (Stockholm)' },
            { from: 'Sweden (Stockholm)', to: 'USA (Washington D.C.)' },
            { from: 'USA (Washington D.C.)', to: 'Egypt (Cairo)' },
            { from: 'Egypt (Cairo)', to: 'Jordan (Amman)' },
            { from: 'Jordan (Amman)', to: 'Saudi Arabia (Riyadh)' },
            { from: 'Saudi Arabia (Riyadh)', to: 'UAE (Dubai)' },
            { from: 'USA (Washington D.C.)', to: 'Saudi Arabia (Riyadh)' },
            { from: 'Sweden (Stockholm)', to: 'Jordan (Amman)' },
          ]"
        />
      </div>
    </div>
  </section>
</template>

<style lang="scss">
.v-btn__content {
  .to-top {
    color: black !important;
  }
}

.cropped2 {
  width: 200px; /* width of container */
  height: 200px; /* height of container */
  object-fit: cover;
  object-position: 20% 10%; /* try 20px 10px */
  border: 5px solid black;
}

.backgroundImage {
  background-image: url("//unpkg.com/three-globe/example/img/night-sky.png") !important;
  background-position: 0 -50px;
  color: white !important;
}

.globe {
  // .col-lg, col{
  padding: 0px !important;
  // }
  canvas {
    display: block;
    touch-action: none;
    // width: 1200px !important;
    // height: 800px !important;
    height: 30rem !important ;
    margin: auto;
    @media screen and (max-width: 500px) {
      width: 20rem !important;
      // height: 20rem !important;
    }
  }
}

.padding-globe§ {
  padding-left: 10rem;
  padding-top: 2rem;
  @media screen and (max-width: 500px) {
    padding-left: 1rem;
    padding-bottom: 2rem;
  }
}

.globe-container {
  max-height: 480px;
}

@media (max-width: 500px) {
  .globe-container {
    max-height: 240px;
  }
}

@media (max-width: 560px) {
  #blog {
    padding: 15px;
  }
}
</style>
