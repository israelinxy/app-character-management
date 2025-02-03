<template>
  <form @submit.prevent="handleSubmit" :class="$style.characterForm">
    <h2 :class="$style.formTitle">{{ isEdit ? 'Edit' : 'Create' }} Character</h2>
    <InputElement
      v-model="formData.name.value"
      type="text"
      placeholder="Name"
      :error="formData.name.error"
      required
    />
    <div v-if="formData.name.error" :class="$style.errorMessage">
      {{ formData.name.error }}
    </div>
    <InputElement
      v-model="formData.lastName.value"
      type="text"
      placeholder="Last Name"
      :error="formData.lastName.error"
      required
    />
    <div v-if="formData.lastName.error" :class="$style.errorMessage">
      {{ formData.lastName.error }}
    </div>
    <button type="submit">{{ isEdit ? 'Edit' : 'Save' }} Character</button>
  </form>
</template>

<script lang="ts" setup>
import InputElement from '@/components/InputElement.vue'
import type { Character } from '@/models/characterModel'
import { useCharacterStore } from '@/stores/useCharacterStore'
import { computed, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { z } from 'zod'

const route = useRoute()
const router = useRouter()
// Determina si es una edicion
const isEdit = computed(() => !!route.params.id)
const characterStore = useCharacterStore()

// Definir el esquema de validacion para el personaje
const characterSchema = z.object({
  name: z.string().min(1, { message: 'Name is required' }),
  lastName: z.string().min(1, { message: 'Last name is required' }),
})

// Definir el formulario
const formData = ref({
  name: { value: '', error: '' },
  lastName: { value: '', error: '' },
})

// Si es una edicion, cargar los datos del personaje
if (isEdit.value) {
  const id = Number(route.params.id)
  const character = characterStore.getCharacter(id)

  if (character) {
    formData.value = {
      name: { value: character.name, error: '' },
      lastName: { value: character.lastName, error: '' },
    }
  }
}

// Manejar el envio del formulario
const handleSubmit = async () => {
  try {
    // Limpieza de errores previos
    Object.keys(formData.value).forEach((key) => {
      formData.value[key as keyof typeof formData.value].error = ''
    })

    // Validación con zod de los datos del formulario
    const result = characterSchema.safeParse({
      name: formData.value.name.value,
      lastName: formData.value.lastName.value,
    })

    // Si la validacion falla, Manejo de errores de validacion del formulario
    if (!result.success) {
      result.error.errors.forEach((e) => {
        const field = e.path[0] as keyof typeof formData.value
        if (formData.value[field]) {
          formData.value[field].error = e.message
        }
      })
      return
    }

    // Preparar los datos para el envio
    const formDataPayload = Object.fromEntries(
      Object.entries(formData.value).map(([key, { value }]) => [key, value]),
    ) as unknown as Omit<Character, 'id'>

    // Enviar los datos al servidor (PUT para editar, POST para crear)
    if (isEdit.value) {
      const formatDataPayloadWithId = { ...formDataPayload, id: Number(route.params.id) }
      characterStore.editCharacter(formatDataPayloadWithId)
    } else {
      characterStore.addCharacter(formDataPayload)
    }

    // Redireccionar a la lista de personajes
    router.push('/characters')
  } catch (err: unknown) {
    console.error('Error saving character', err)
  }
}
</script>

<style module lang="scss">
.characterForm {
  display: flex;
  flex-direction: column;
  max-width: 400px;
  margin: 0 auto;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;

  :global(input) {
    margin-bottom: 20px;
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 5px;
    font-size: 16px;
    transition: border-color 0.3s ease;

    &:focus {
      outline: none;
      border-color: #6c5ce7;
      box-shadow: 0 0 0 2px rgba(108, 92, 231, 0.2);
    }
  }

  button {
    padding: 12px 20px;
    background-color: #6c5ce7;
    color: white;
    border: none;
    border-radius: 5px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    transition:
      background-color 0.3s ease,
      transform 0.1s ease;

    &:hover {
      background-color: #5b4bc4;
    }

    &:active {
      transform: scale(0.98);
    }
  }
}

.errorMessage {
  color: #e74c3c;
  font-size: 14px;
  margin-top: -15px;
  margin-bottom: 15px;
}

.formTitle {
  font-size: 24px;
  color: #2c3e50;
  margin-bottom: 20px;
  text-align: center;
}
</style>
