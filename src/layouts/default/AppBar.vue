<template>
  <v-app-bar flat class="bg-secondary">
    <v-app-bar-title class="text-primary">
      <div id="wrapper">
        <h3>Tropivisor</h3>
        <v-btn
          id="create-button"
          aria-label="Click to add a new destination"
          class="text-accent"
          icon="mdi-plus-circle"
          @click="dialog = true"
        />
      </div>
        <v-dialog
          v-model="dialog">
          <v-card
            id="card"
            class="bg-white"
          >
            <h3>Add new destination</h3>
            <CountrySelect
              aria-label="Enter the destination's country"
              @update="(val) => country = val.code"
            />
            <v-text-field
              aria-label="Enter the destination's address"
              v-model="address"
              label="Address"
            />
            <v-text-field
              aria-label="Enter the destination's name"
              v-model="name"
              label="Name"
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
            aria-label="Filter by destinations accessible for those with mobility impairments"
            v-model="impairment.mobility"
            label="Mobility Impaired"
          />
          <v-btn
            aria-label="Submit"
            class="bg-blue text-white"
            @click="submitNewDestination"
          >
            Submit
          </v-btn>
          </v-card>
        </v-dialog>
    </v-app-bar-title>
  </v-app-bar>
</template>

<script lang="ts" setup>
import CountrySelect from '@/components/CountrySelect.vue';
import axios from 'axios';
import { reactive, ref } from 'vue';
  const dialog = ref(false);

  const country = ref("");
  const address = ref('');
  const name = ref('');

  const impairment = reactive({
    visual: false,
    hearing: false,
    mobility:  false
  });

  async function submitNewDestination() {
    const strengths = [];
    if (impairment.hearing) {
      strengths.push({ disability: 'auditory', rating: 1 })
    }
    if (impairment.visual) {
      strengths.push({ disability: 'visual', rating: 1 })
    }
    if (impairment.mobility) {
      strengths.push({ disability: 'mobility', rating: 1 })
    }
    await axios.post('location', {
      country: country.value,
      address: address.value,
      name: name.value,
      strengths
    });
  }
</script>

<style scoped>

  h3 {
    margin-top: .5em
  }
  
  #card {
    display: flex;
    flex-direction: column;
  }

  #wrapper {
    display: flex;
    justify-content: center;
  }

  #create-button {
    margin-left: auto;
  }


</style>
