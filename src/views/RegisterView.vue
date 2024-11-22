<template>
  <div class="container mt-5">
    <h2 class="mb-4">Registro de nuevos usuarios</h2>
    <form @submit.prevent="register">
      <div class="mb-3">
        <label for="name" class="form-label">Nombre:</label>
        <input v-model="name" type="text" class="form-control" id="name" required />
      </div>
      <div class="mb-3">
        <label for="email" class="form-label">Email:</label>
        <input v-model="email" type="email" class="form-control" id="email" required />
      </div>
      <div class="mb-3">
        <label for="password" class="form-label">Password:</label>
        <input v-model="password" type="password" class="form-control" id="password" required />
      </div>
      <button type="submit" class="btn btn-primary">Registrarse</button>
    </form>
  </div>
</template>
  
<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import Swal from 'sweetalert2' // Importa SweetAlert2

const name = ref('')
const email = ref('')
const password = ref('')
const router = useRouter()

const register = () => {
  const users = JSON.parse(localStorage.getItem('users')) || []

  if (users.some((user) => user.email === email.value)) {
    Swal.fire({
      title: 'Error!',
      text: 'El correo ya está registrado',
      icon: 'error',
      confirmButtonText: 'Aceptar'
    })
  } else {
    users.push({
      name: name.value,
      email: email.value,
      password: password.value
    })
    localStorage.setItem('users', JSON.stringify(users))

    Swal.fire({
      title: 'Registro Exitoso!',
      text: 'Te has registrado correctamente.',
      icon: 'success',
      confirmButtonText: 'Aceptar'
    }).then(() => {
      router.push('/login')
    })
  }
}
</script>

  