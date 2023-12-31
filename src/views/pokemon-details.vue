<script setup lang="ts">
import { useRoute, onBeforeRouteUpdate } from 'vue-router'
import { usePokemonStore } from '@/stores/pokemon'
import findFirstTypeColor from '@/utilities/pokemon-type-colors'
import { storeToRefs } from 'pinia'
import { ref, onMounted } from 'vue'
import detailsCard from '@/components/details-card.vue'
import sadpikachu from '@/assets/sadpikachu.png'
import pikachu from '@/assets/pikachu.png'

const pokemonStore = usePokemonStore()
const { selectedPokemon, selectedPokemonSpecies } = storeToRefs(pokemonStore)
const loading = ref(false)
const error = ref(false)

const fetchData = async (pokemonId: number) => {
  loading.value = true
  const result = await pokemonStore.fetchPokemonDetails(pokemonId)
  const resultSpecies = await pokemonStore.fetchSpecies(pokemonId)
  const resultEvolution = await pokemonStore.fetchPokemonEvolution(
    selectedPokemonSpecies?.value?.evolution_chain?.url || ''
  )
  if (result?.error || resultSpecies?.error || resultEvolution?.error) {
    error.value = true
  }
  loading.value = false
}

onMounted(() => {
  const initialPokemonId = useRoute().params.id as unknown as number
  fetchData(initialPokemonId)
})

onBeforeRouteUpdate((to, from, next) => {
  const newPokemonId = to.params.id as unknown as number
  fetchData(newPokemonId)
  next()
})
</script>

<template>
  <div
    class="pt-4 h-[100vh]"
    :style="{
      backgroundColor: selectedPokemon?.types ? findFirstTypeColor(selectedPokemon?.types) : '#777'
    }"
    v-if="!error && !loading"
  >
    <div class="px-4 flex flex-row justify-start my-2 items-center">
      <router-link to="/pokemon">
        <font-awesome-icon
          icon="fa-solid fa-arrow-left"
          style="height: 17px; width: 17px; padding: 0; color: white"
        />
      </router-link>
    </div>
    <detailsCard />
  </div>
  <div v-if="loading" class="flex flex-col justify-center gap-2 h-[100vh] items-center">
    Fetching pokémon...
    <img :src="pikachu" alt="pikachu" class="w-[100vw]" />
  </div>
  <div v-if="error" class="flex flex-col justify-center gap-2 h-[100vh] items-center">
    Error in fetching pokémon...
    <img :src="sadpikachu" alt="pikachu" class="w-[100vw]" />
  </div>
</template>
