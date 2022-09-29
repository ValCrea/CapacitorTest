<script setup lang="ts">
import { ref } from "vue";
import { Geolocation } from "@capacitor/geolocation";

const loc = ref({ lat: 0, long: 0 });
const msg = ref("Click on the button to get current location");
const getLocation = async () => {
  try {
    const pos = await Geolocation.getCurrentPosition();
    loc.value = {
      lat: pos.coords.latitude,
      long: pos.coords.longitude,
    };
    msg.value = "";
  } catch (err: unknown) {
    if (err instanceof Error) msg.value = err.message;
    else msg.value = "Something went wrong";
  }
};

const count = ref(0);
</script>

<template>
  <button @click="getLocation">Get location</button>
  <div v-if="msg">{{ msg }}</div>
  <div v-else>Location: {{ loc.lat }}, {{ loc.long }}</div>
  <br />
  <button @click="count++" style="width: 90%; height: 200px; font-size: 5rem">
    {{ count }}
  </button>
</template>
