<template>
  <input class="input-field" v-model="computedModel" :type="type" :placeholder="placeholder" :required></input>
  <p v-if="error" class="error-message">{{ error }}</p>
</template>

<script lang="ts" setup>
import { computed } from 'vue';

const props = defineProps({
  modelValue: {
    type: String,
    required: true,
  },
  type: {
    type: String,
    default: 'text',
  },
  placeholder: {
    type: String,
    default: '',
  },
  error: {
    type: String,
    default: '',
  },
  required: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['update:modelValue'])

const computedModel = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value),
})

</script>

<style module lang="scss">
  .input-field {
    width: 100%;
    padding: 0.5rem;
    border-radius: 0.25rem;
    border: 1px solid #ccc;
    font-size: 1rem;
    margin-bottom: 0.5rem;
  }
  .error-message {
    color: red;
    margin-top: 0.25rem;
    font-size: 0.875rem;    
  }
</style>
