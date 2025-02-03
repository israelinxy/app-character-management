<template>
  <ul>
    <li v-for="character in characters" :key="character.id">
      <CharacterDetail :character="character" />
    </li>
  </ul>
</template>

<script lang="ts" setup>
import type { Character } from '@/models/characterModel'
import CharacterDetail from '../CharacterDetail/CharacterDetail.vue'

defineProps<{ characters: Character[] }>()
</script>
