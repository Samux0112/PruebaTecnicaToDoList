<template>
  <div class="container mt-5">
    <h2 class="mb-4">Inicia sesión para ingresar</h2>
    <form @submit.prevent="login">
      <div class="mb-3">
        <label for="email" class="form-label">Email:</label>
        <input v-model="email" type="email" class="form-control" id="email" required />
      </div>
      <div class="mb-3">
        <label for="password" class="form-label">Contraseña:</label>
        <input v-model="password" type="password" class="form-control" id="password" required />
      </div>
      <button type="submit" class="btn btn-primary">Iniciar sesión</button>
    </form>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import Swal from 'sweetalert2'
import { useAuthStore } from '@/stores/auth'

const email = ref('')
const password = ref('')
const router = useRouter()
const authStore = useAuthStore() // Aquí inicializas la tienda

const login = () => {
  const users = JSON.parse(localStorage.getItem('users')) || []
  const user = users.find((user) => user.email === email.value && user.password === password.value)

  if (user) {
    authStore.login() // Cambia el estado a logueado
    localStorage.setItem('loggedIn', true)
    Swal.fire({
      title: 'Éxito!',
      text: 'Has iniciado sesión correctamente.',
      icon: 'success',
      confirmButtonText: 'Aceptar'
    }).then(() => {
      router.replace('/') // Mueve el redireccionamiento dentro de la promesa
    })
  } else {
    Swal.fire({
      title: 'Error!',
      text: 'Usuario o contraseña incorrectos',
      icon: 'error',
      confirmButtonText: 'Aceptar'
    })
  }
}
</script>
