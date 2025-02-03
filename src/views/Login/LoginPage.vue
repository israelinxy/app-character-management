<template>
  <form @submit.prevent="handleLogin" :class="$style.loginForm">
    <InputElement
      v-model="formData.email.value"
      type="email"
      placeholder="Email"
      :error="formData.email.error"
      required
    />
    <div v-if="formData.email.error" :class="$style.errorMessage">
      {{ formData.email.error }}
    </div>
    <InputElement
      v-model="formData.password.value"
      type="password"
      placeholder="Password"
      :error="formData.password.error"
      required
    />
    <div v-if="formData.password.error" :class="$style.errorMessage">
      {{ formData.password.error }}
    </div>
    <button type="submit">Login</button>
  </form>
</template>

<script lang="ts" setup>
import InputElement from '@/components/InputElement.vue'
import { login } from '@/services/authenticateUser'
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { z } from 'zod'

const loginSchema = z.object({
  email: z.string().email({ message: 'Invalid email address' }),
  password: z.string().min(6, { message: 'Password must be 6 characters at least' }),
})

const formData = ref({
  email: {
    value: '',
    error: '',
  },
  password: {
    value: '',
    error: '',
  },
})

const router = useRouter()

const handleLogin = async () => {
  try {
    // Limpieza de errores previos
    Object.keys(formData.value).forEach((key) => {
      formData.value[key as keyof typeof formData.value].error = ''
    })

    // Validación con zod
    const result = loginSchema.safeParse({
      email: formData.value.email.value,
      password: formData.value.password.value,
    })

    // Manejo de errores de validacion
    if (!result.success) {
      result.error.errors.forEach((e) => {
        const field = e.path[0] as keyof typeof formData.value

        if (formData.value[field]) {
          formData.value[field].error = e.message
        }
      })
    }

    // pegar el formulario al backend
    await login(formData.value.email.value, formData.value.password.value)

    router.push('/characters')
  } catch (err) {
    console.error('Error logging in', err)
  }
}
</script>

<style module lang="scss">
.loginForm {
  display: flex;
  flex-direction: column;
  max-width: 300px;
  margin: 0 auto;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  background-color: #f9f9f9;

  :global(input) {
    margin-bottom: 15px;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 16px;

    &:focus {
      outline: none;
      border-color: #007bff;
    }
  }

  button {
    padding: 10px 15px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease;

    &:hover {
      background-color: #0056b3;
    }
  }
}

.errorMessage {
  color: #dc3545;
  font-size: 14px;
  margin-top: -10px;
  margin-bottom: 10px;
}
</style>
