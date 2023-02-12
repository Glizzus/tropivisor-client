<template>
  <v-container class="fill-height">
    <v-responsive class="d-flex align-center text-center fill-height">
      <v-row>
        <v-col>
          <CountrySelect
            @update="(value) => select = value"
          />
          <v-checkbox
            v-model="impairment.visual"
            label="Visually Impaired"
          />
          <v-checkbox
            v-model="impairment.hearing"
            label="Hearing Impaired"
          />
          <v-checkbox
            v-model="impairment.mobility"
            label="Mobility Impaired"
          />
        </v-col>
        <v-col>
          <v-card
            v-for="(location, rank) in matchingLocations"
            :key="location.name"
          >
            <v-row>
              <v-col class="text-left">
                {{ rank }}
              </v-col>
              <v-col>
                <v-btn
                  @click="() => updateMap(location.name)"
                >
                  {{ location.name }}
                </v-btn>
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>
      <iframe
        id="map"
        width="600"
        height="450"
        style="border:0"
        loading="lazy"
        allowfullscreen
        referrerpolicy="no-referrer-when-downgrade"
        src="https://www.google.com/maps/embed/v1/place?key=AIzaSyAKFsjsUEcecY7-AbbNgjKsLdId3zOpMpo
          &q=Space+Needle,Seattle+WA">
      </iframe>
    </v-responsive>
  </v-container>
</template>

<script lang="ts" setup>
import { computed, onUpdated, reactive, ref } from 'vue'
import CountrySelect from './CountrySelect.vue';
import axios from 'axios';

  const locations = ref<Array<{ 
    name: string,
    country: string,
    strengths: Array<{
      disability: string,
      rating: number
    }>
  }>>([]);

  const matchingLocations = computed(() => {
    return locations
      .value
      .filter((location) => location.country == select.value.code)
      .filter((location) => {
        if (impairment.hearing) {
          return location.strengths.find((str) => {
            return str.disability == "hearing" && str.rating == 1
          });
        }
        return true;
      })
      .filter((location) => {
        if (impairment.visual) {
          return location.strengths.find((str) => {
            return str.disability == "visual" && str.rating == 1
          });
        }
        return true;
      })
      .filter((location) => {
        if (impairment.mobility) {
          const found = location.strengths.find((str) => {
            return str.disability == "mobility" && str.rating == 1
          });
          return found;
        }
        return true;
      })
  });

  async function getLocations() {
    const { data } = await axios.get('location');
    locations.value = data;
  }
  getLocations();

  function updateMap(name: string) {
    const queryRegex = /(?<=&q=)(.*)(?=(&|$))/;
    const map = document.getElementById("map");
    if (!map) {
      return;
    }
    const previousSrc = map.getAttribute("src");
    if (!previousSrc) {
      return;
    }
    const newSrc = previousSrc.replace(queryRegex, name)
    map?.setAttribute("src", newSrc);
  }
  
  const select = ref({ name: "United States", code: "US" });

  const impairment = reactive({
    visual: false,
    hearing: false,
    mobility:  false
  });

</script>
