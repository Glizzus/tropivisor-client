<template>
  <v-container class="fill-height">
    <v-responsive class="d-flex align-center text-center fill-height">
      <v-row>   
      <v-col>
            <h3 class="text-secondary">Filter your destination</h3>
            <CountrySelect
              class="text-secondary text-bold"
              aria-label="Enter country to filter destinations by"
              @update="(value) => select = value"
            />
            <v-checkbox
              aria-label="Filter by destinations accessible for the visually impaired"
              v-model="impairment.visual"
              label="Visually Impaired"
            />
            <v-checkbox
              aria-label="Filter by destinations accessible for the hearing impaired"
              v-model="impairment.hearing"
              label="Hearing Impaired"
            />
            <v-checkbox
              v-model="impairment.mobility"
              label="Mobility Impaired"
            />
          </v-col>
          <v-col>
            <p v-if="matchingLocations.length == 0">No locations found that satisfy filter</p>
            <v-card
              v-else
              v-for="(location, rank) in matchingLocations"
              :key="location.name"
            >
              <section
                id="card"
                class="bg-secondary text-background"
                >
                <p>{{  rank + 1  }}</p>
                <p><strong>{{ location.name }}</strong></p>
                <v-btn
                  class="text-capitalize"
                  icon
                  @click="() => goToMaps(location.name)"
                >
                  <v-icon>mdi-map</v-icon>
                  <v-tooltip
                    activator="parent"
                    location="top"
                    text="Check it out on Google Maps"
                  />

                </v-btn>
                <v-tooltip 
                  text="Check it out on Google Maps">
                  location="top"
                >
                  <template v-slot:activation="{ props }">
                    <v-btn
                      class="text-capitalize"
                      icon
                      @click="() => goToMaps(location.name)" 
                    >
                      <v-icon>mdi-map</v-icon>
                    </v-btn>
                  </template>
                </v-tooltip>     
              </section>
            </v-card>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script lang="ts" setup>
import { computed, onUpdated, reactive, ref } from 'vue'
import CountrySelect from './CountrySelect.vue';
import axios from 'axios';
import { stringify } from 'querystring';

  const select = ref<{ name: string, code: string }>();

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
      .filter((location) => {
        if (!location.country) {
          return true;
        }
        if (!select.value) {
          return true;
        }
        return location.country == select.value.code
      })
      .filter((location) => {

        if (impairment.hearing) {
          return location.strengths.find((str) => {
            return str.disability == "auditory" && str.rating == 1
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

  function goToMaps(name: string) {
    window.open(`https://www.google.com/maps/search/${name}`)
  }

  async function getLocations() {
    const { data } = await axios.get('location');
    locations.value = data;
  }
  getLocations();

  const impairment = reactive({
    visual: false,
    hearing: false,
    mobility:  false
  });

</script>

<style>

#card {
  display: flex;
  flex-direction: row;
  padding: .25em;
  justify-content: space-between;
}
</style>