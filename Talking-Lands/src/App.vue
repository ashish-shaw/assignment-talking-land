<template>
  <div class="flex space-x-4">
    <Map />
    <Polygon/>
  </div>
    
</template>

<script>
import Map from "@/components/Map.vue";
import Polygon from "./components/Polygon.vue";

export default {
  components: {
    Map,
    Polygon
  },
};
</script>


