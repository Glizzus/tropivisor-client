<template>
  <v-app-bar flat class="bg-secondary">
    <v-app-bar-title class="text-primary">
      <div id="flexparent">
        <h3>Tropivisor</h3>
        <v-btn
          icon="mdi-plus-circle"
          @click="dialog = true"
        />
        <v-dialog
          v-model="dialog">
          <v-card>
            <h3>Add new destination</h3>
            <CountrySelect
              @update="(val) => country = val.code"
            />
            <v-text-field
              v-model="address"
              label="Address"
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
          <v-btn
            label="submit"
            class="bg-blue text-white"
            @click="submitNewDestination"
            />
          </v-card>
        </v-dialog>
      </div>
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
      strengths
    });
  }
</script>

<style>
  #flexparent {
    display: flex;
    flex-direction: reverse;
    justify-content: space-between;
  }
</style>
