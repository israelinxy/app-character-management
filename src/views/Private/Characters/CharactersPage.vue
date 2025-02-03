<template>
  <div :class="$style.charactersPage">
    <h1>Characters List</h1>
    <CharactersList :characters="characters" :class="$style.charactersList" />
    <button :class="$style.addButton" @click="navigateToAddCharacter">Add New Character</button>
  </div>
</template>

<script lang="ts" setup>
import { useCharacterStore } from '@/stores/useCharacterStore'
import { computed } from 'vue'
import CharactersList from './CharactersList/CharactersList..vue'
import { useRouter } from 'vue-router'

console.log('Characters')
const characterStore = useCharacterStore()

const characters = computed(() => characterStore.formattedCharacters)

const router = useRouter()

const navigateToAddCharacter = () => {
  router.push('/characters/new')
}
</script>

<style module lang="scss">
.charactersPage {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;

  h1 {
    font-size: 2.5rem;
    color: #2c3e50;
    margin-bottom: 1.5rem;
    text-align: center;
    font-weight: 700;
    border-bottom: 3px solid #3498db;
    padding-bottom: 10px;
  }
}

.charactersList {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;

  :global(.character-card) {
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 15px;
    transition:
      transform 0.3s ease,
      box-shadow 0.3s ease;

    &:hover {
      transform: translateY(-5px);
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
    }

    h2 {
      font-size: 1.25rem;
      color: #34495e;
      margin-bottom: 10px;
    }

    p {
      color: #7f8c8d;
      font-size: 0.9rem;
    }
  }
}

.addButton {
  display: block;
  width: 200px;
  margin: 20px auto;
  padding: 10px 20px;
  background-color: #2ecc71;
  color: white;
  text-align: center;
  border: none;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s ease;

  &:hover {
    background-color: #27ae60;
  }
}
</style>
