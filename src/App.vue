<script setup>
import { useAuthStore } from '@/stores/auth' // Importamos el store de autenticación
import { useRouter } from 'vue-router'
import Swal from 'sweetalert2'

const authStore = useAuthStore()
const router = useRouter()

const logout = () => {
  Swal.fire({
    title: '¿Estás seguro?',
    text: '¿Deseas cerrar sesión?',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonText: 'Sí, cerrar sesión',
    cancelButtonText: 'Cancelar'
  }).then((result) => {
    if (result.isConfirmed) {
      localStorage.removeItem('loggedIn') // Elimina la información de sesión
      authStore.logout() // Cambia el estado a no logueado
      router.push('/login') // Redirige al usuario a la página de login
    }
  })
}
</script>

<template>
  <header>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark fixed-top">
      <div class="container-fluid">
        <img alt="To-Do Logo" class="logo me-3" src="@/assets/logo.png" width="50" height="50" />
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarNav"
          aria-controls="navbarNav"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav ms-auto">
            <li class="nav-item">
              <RouterLink to="/" class="nav-link" active-class="active"><b>Inicio</b></RouterLink>
            </li>
            <li class="nav-item">
              <RouterLink to="/about" class="nav-link" active-class="active"
                ><b>Sobre la app</b></RouterLink
              >
            </li>
            <li class="nav-item" v-if="!authStore.isLoggedIn">
              <RouterLink to="/register" class="nav-link" active-class="active"
                ><b>Registrarse</b></RouterLink
              >
            </li>
            <li class="nav-item" v-if="authStore.isLoggedIn">
              <button @click="logout" class="btn btn-danger ms-3">Cerrar Sesión</button>
            </li>
          </ul>
        </div>
      </div>
    </nav>
  </header>

  <RouterView />
</template>

<style scoped>
/* Estilo para el logo */
.logo {
  margin-right: 1rem;
}

/* Ajustes para las clases activas */
.nav-link {
  color: white !important; /* Asegura que el texto sea siempre visible */
}

.nav-link.active {
  background-color: #0056b3; /* Color de fondo activo */
  color: white !important; /* Texto blanco */
}

/* Ajustes para el navbar fijo */
body {
  padding-top: 56px; /* Espacio para evitar que el contenido se superponga al navbar */
}
</style>
