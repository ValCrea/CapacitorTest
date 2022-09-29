<!--
npx cap copy
npx cap sync
npx cap open
npx cap run?
-->

<script setup lang="ts">
import { ref, onMounted } from "vue";
import type { Ref } from "vue";

import DynamicText from "@/components/DynamicText.vue";
import { Geolocation } from "@capacitor/geolocation";
import { Filesystem, Directory, Encoding } from "@capacitor/filesystem";

const loc: Ref<{ lat: number; lon: number } | null> = ref(null);
const errorLoc = ref("");
const getLocation = async () => {
  loc.value = null;
  try {
    const pos = await Geolocation.getCurrentPosition();
    loc.value = {
      lat: pos.coords.latitude,
      lon: pos.coords.longitude,
    };
    errorLoc.value = "";
  } catch (err: any) {
    if (err && err.message) errorLoc.value = err.message;
    else errorLoc.value = "Something went wrong";
  }
};
onMounted(() => getLocation());

const errorFile = ref("");
const saveLocation = async () => {
  errorFile.value = "";
  try {
    if (!loc.value) throw "No location";

    const date = new Date().toLocaleString();
    await Filesystem.appendFile({
      path: "GeoGet/locations.txt",
      data: `${date}|${loc.value.lat}|${loc.value.lon}|`,
      directory: Directory.Documents,
      encoding: Encoding.UTF8,
    });

    locationList.value = await readLocations();
  } catch (err: any) {
    errorFile.value = "Coludn't save Location";
  }
};

const locationList: Ref<{ date: string; lat: string; lon: string }[]> = ref([]);
const readLocations = async () => {
  errorFile.value = "";
  try {
    const contents = await Filesystem.readFile({
      path: "GeoGet/locations.txt",
      directory: Directory.Documents,
      encoding: Encoding.UTF8,
    });

    const data = contents.data.split("|");
    data.pop();
    if (!data) throw "Invalid data";

    const locations = [];
    for (let l = 0; l < data?.length; l += 3) {
      locations.push({
        date: data[l],
        lat: data[l + 1],
        lon: data[l + 2],
      });
    }
    return locations;
  } catch (err: any) {
    errorFile.value = "Coludn't read Locations";
    return [];
  }
};

const clearLocations = async () => {
  errorFile.value = "";
  try {
    await Filesystem.writeFile({
      path: "GeoGet/locations.txt",
      data: "",
      directory: Directory.Documents,
      encoding: Encoding.UTF8,
    });

    locationList.value = await readLocations();
  } catch (err: any) {
    errorFile.value = "Coludn't clear Locations";
  }
};
onMounted(async () => {
  locationList.value = await readLocations();
});
</script>

<template>
  <div class="app">
    <div class="app__container">
      <h1 class="app__title">
        <DynamicText
          text="GeoGet"
          type="color"
          effect="letter"
          to="#55aa55"
          :duration="1"
          :offset="0.1"
        />
      </h1>

      <b v-if="errorLoc" class="app__error">{{ errorLoc }}</b>

      <section v-else class="app__location">
        <section v-if="loc">
          <h2 class="app__subtitle">Current Location:</h2>
          <p class="app__text">Lat: {{ loc.lat }}</p>
          <p class="app__text">Lon: {{ loc.lon }}</p>
        </section>
        <section v-else>
          <h2 class="app__subtitle">
            Loading
            <DynamicText
              text="..."
              type="bounce"
              effect="letter"
              to="-0.5rem"
              :duration="0.5"
              :offset="0.1"
            />
          </h2>
        </section>
      </section>

      <section class="app__controls">
        <button @click="getLocation" class="app__button">Refresh</button>
        <button @click="saveLocation" class="app__button">Save</button>
        <button @click="clearLocations" class="app__button">Clear</button>
      </section>

      <br />
      <section class="app__saved">
        <h2>Past Locations:</h2>
        <b v-if="errorFile" class="app__error">{{ errorFile }}</b>
        <template v-else>
          <template v-if="locationList.length > 0">
            <div v-for="l in locationList" :key="l.date">
              <h3 class="app__date">-{{ l.date }}</h3>
              <p class="app__loc">Lat: {{ l.lat }}</p>
              <p class="app__loc">Lon: {{ l.lon }}</p>
            </div>
          </template>

          <b v-else class="app__info">No saved Locations</b>
        </template>
      </section>
    </div>
  </div>
</template>

<style scoped lang="scss">
$white-green: rgb(221, 255, 221);
$light-green: rgb(77, 178, 95);
$dark-green: rgb(9, 90, 24);
$black-green: rgb(0, 69, 15);
$error: rgb(101, 0, 0);

.app {
  width: 100%;
  height: 100vh;

  color: $black-green;
  background-color: $white-green;
  font-family: "Courier New", Courier, monospace;

  &__container {
    width: 90%;
    max-width: 800px;
    margin-inline: auto;

    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  &__title {
    margin-inline: auto;

    font-size: 3rem;
    font-weight: bolder;
  }

  &__subtitle {
    font-size: 1.5rem;
    font-weight: bold;
  }

  &__text {
    font-size: 1.15rem;
    text-indent: 2rem;
  }

  &__error {
    color: $error;
    font-size: 1.5rem;
    font-weight: bold;
  }

  &__location {
    font-weight: bold;
  }

  &__controls {
    margin-inline: auto;

    display: flex;
    flex-direction: row;
    gap: 2rem;
  }

  &__button {
    width: 25vw;
    max-width: 150px;
    padding: 0.5rem;

    font-size: 1.2rem;
    color: $black-green;
    background-color: $light-green;
    border: 0.15rem solid $dark-green;

    &:focus {
      filter: brightness(115%);
    }
  }

  &__saved {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  &__date {
    text-indent: 1rem;
  }

  &__loc {
    text-indent: 3rem;
    font-weight: bold;
  }

  &__info {
    text-indent: 1rem;
    color: $dark-green;
    font-size: 1.35rem;
    font-weight: bold;
  }
}
</style>
