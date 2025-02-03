<template>
  <form @submit.prevent="handleRegister" :class="$style.registerForm">
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
    <InputElement
      v-model="formData.confirmPassword.value"
      type="password"
      placeholder="Confirm password"
      :error="formData.confirmPassword.error"
      required
    />
    <div v-if="formData.confirmPassword.error" :class="$style.errorMessage">
      {{ formData.confirmPassword.error }}
    </div>
    <button type="submit">Register</button>
  </form>
</template>

<script lang="ts" setup>
import InputElement from '@/components/InputElement.vue'
import { register } from '@/services/authenticateUser'
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { z } from 'zod'

const registerSchema = z
  .object({
    email: z.string().email({ message: 'Invalid email address' }),
    password: z.string().min(6, { message: 'Password must be 6 characters at least' }),
    confirmPassword: z
      .string()
      .min(6, { message: 'Confirm password must be 6 characters at least' }),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ['confirmPassword'],
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
  confirmPassword: {
    value: '',
    error: '',
  },
})

const router = useRouter()

const handleRegister = async () => {
  try {
    // Limpieza de errores previos
    Object.keys(formData.value).forEach((key) => {
      formData.value[key as keyof typeof formData.value].error = ''
    })

    // Validación con zod
    const result = registerSchema.safeParse({
      email: formData.value.email.value,
      password: formData.value.password.value,
      confirmPassword: formData.value.confirmPassword.value,
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
    await register(formData.value.email.value, formData.value.password.value)

    router.push('/login')
  } catch (err) {
    console.error('Error during registration', err)
  }
}
</script>

<style module lang="scss">
.registerForm {
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
      border-color: #28a745;
    }
  }

  button {
    padding: 10px 15px;
    background-color: #28a745;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease;

    &:hover {
      background-color: #218838;
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
