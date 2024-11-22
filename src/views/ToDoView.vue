<template>
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css"
  />
  <div>
    <!-- Contenido principal -->
    <div class="container mt-5 pt-5">
      <h2 class="mb-4 text-center">Lista de Tareas</h2>

      <!-- Botón para abrir el modal -->
      <div class="d-flex justify-content-center mb-4">
        <button class="btn btn-primary" @click="openModal">Agregar una nueva tarea</button>
      </div>

      <!-- Modal para agregar tarea -->
      <div
        class="modal fade"
        id="taskModal"
        tabindex="-1"
        aria-labelledby="taskModalLabel"
        aria-hidden="true"
      >
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="taskModalLabel">
                {{ isEditing ? 'Editando una tarea' : 'Agregando una tarea nueva' }}
              </h5>
              <button
                type="button"
                class="btn-close"
                data-bs-dismiss="modal"
                aria-label="Close"
              ></button>
            </div>
            <div class="modal-body">
              <form @submit.prevent="isEditing ? updateTask() : addTask()">
                <div class="mb-3">
                  <input
                    v-model="newTask.title"
                    type="text"
                    class="form-control"
                    placeholder="Título"
                    required
                  />
                </div>
                <div class="mb-3">
                  <textarea
                    v-model="newTask.description"
                    class="form-control"
                    placeholder="Descripción (opcional)"
                  ></textarea>
                </div>
                <div class="mb-3">
                  <input v-model="newTask.dueDate" type="date" class="form-control" required />
                </div>
                <div class="mb-3">
                  <select v-model="newTask.category" class="form-select" required>
                    <option value="" disabled selected>Categoría</option>
                    <option value="personal">Personal</option>
                    <option value="trabajo">Trabajo</option>
                    <option value="estudio">Estudio</option>
                  </select>
                </div>
                <div class="d-flex justify-content-end">
                  <button type="submit" class="btn btn-primary">
                    {{ isEditing ? 'Actualizar' : 'Agregar' }}
                  </button>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>

      <!-- Filtros y Acciones Masivas -->
      <div class="mb-4 d-flex justify-content-center">
        <button @click="filterTasks('all')" class="btn btn-outline-info me-2">Todas</button>
        <button @click="filterTasks('pending')" class="btn btn-outline-warning me-2">
          Pendientes
        </button>
        <button @click="filterTasks('completed')" class="btn btn-outline-success">
          Completadas
        </button>
      </div>
      <div class="mb-4 d-flex justify-content-center">
        <input
          type="checkbox"
          v-model="allCompleted"
          @change="markAllAsCompleted"
          class="form-check-input me-2"
        />
        <label
          >Marcar todas las tareas como completadas o desenmarcar para poner como pendientes todas
          las tareas</label
        >
      </div>

      <!-- Lista de tareas -->
      <div class="row justify-content-center">
        <div v-for="(task, index) in filteredTasks" :key="index" class="col-md-6 col-lg-4 mb-4">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">
                <span :class="{ 'completed-task': task.completed }">{{ task.title }}</span>
              </h5>
              <p class="card-text">
                <span :class="{ 'completed-task': task.completed }">{{
                  task.description || 'Sin descripción'
                }}</span>
              </p>
              <p
                class="card-text text-muted"
              >
                {{ isTaskOverdue(task.dueDate) ? 'Venció el: ' : 'Vence el: ' }}{{ task.dueDate }}
              </p>
              <p class="card-text">Categoría: {{ task.category }}</p>
              <input
                type="checkbox"
                v-model="task.completed"
                @change="markAsCompleted(index)"
                class="form-check-input me-2"
              />
              <label>Marcar la tarea como completada o pendiente</label>
              <button @click="editTask(index)" class="btn btn-warning btn-sm me-2">
                <i class="bi bi-pencil"></i> Editar
              </button>
              <button @click="removeTask(index)" class="btn btn-danger btn-sm">
                <i class="bi bi-trash"></i> Eliminar
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Mensaje si no hay tareas -->
      <div v-if="tasks.length === 0" class="text-center text-muted mt-5">
        No hay tareas por mostrar.
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import Swal from 'sweetalert2'

const newTask = ref({ title: '', description: '', dueDate: '', category: '', completed: false })
const tasks = ref([])
const isEditing = ref(false)
const editingIndex = ref(-1)
const filter = ref('all')
const allCompleted = ref(false)

const validateInputs = () => {
  if (!newTask.value.title.trim()) {
    Swal.fire('Error', 'El título es obligatorio.', 'error')
    return false
  }
  if (!newTask.value.dueDate) {
    Swal.fire('Error', 'La fecha de vencimiento es obligatoria.', 'error')
    return false
  }

  const currentDate = new Date()
  const dueDate = new Date(newTask.value.dueDate)

  // Establecer la hora de ambas fechas a las 00:00:00 para compararlas solo por día, mes y año
  currentDate.setHours(0, 0, 0, 0)
  dueDate.setHours(0, 0, 0, 0)

  // Comprobar si el año de la fecha de vencimiento es el año actual o futuro
  if (dueDate.getFullYear() < currentDate.getFullYear()) {
    Swal.fire('Error', 'La fecha de vencimiento debe ser en el año actual o a futuro.', 'error')
    return false
  }

  if (!newTask.value.category) {
    Swal.fire('Error', 'La categoría es obligatoria.', 'error')
    return false
  }

  return true
}

const isTaskOverdue = (dueDate) => {
  const currentDate = new Date()
  return new Date(dueDate) < currentDate
}

const addTask = () => {
  if (!validateInputs()) return
  tasks.value.push({ ...newTask.value })
  Swal.fire('Éxito', 'Tarea creada exitosamente!', 'success')
  resetForm()
  closeModal()
}

const editTask = (index) => {
  newTask.value = { ...tasks.value[index] }
  editingIndex.value = index
  isEditing.value = true
  openModal()
}

const updateTask = () => {
  if (editingIndex.value >= 0 && validateInputs()) {
    tasks.value[editingIndex.value] = { ...newTask.value }
    Swal.fire('Éxito', 'Tarea actualizada exitosamente!', 'success')
    resetForm()
    closeModal()
  }
}

const removeTask = (index) => {
  const taskToRemove = tasks.value[index].title
  Swal.fire({
    title: '¿Estás seguro?',
    text: `¿Deseas eliminar la tarea "${taskToRemove}"?`,
    icon: 'warning',
    showCancelButton: true,
    confirmButtonText: 'Sí, eliminar',
    cancelButtonText: 'Cancelar'
  }).then((result) => {
    if (result.isConfirmed) {
      tasks.value.splice(index, 1)
      Swal.fire('Eliminado', `Tarea "${taskToRemove}" eliminada!`, 'success')
    }
  })
}

const markAllAsCompleted = () => {
  tasks.value.forEach((task) => (task.completed = allCompleted.value))
  const status = allCompleted.value ? 'Completadas' : 'Pendientes'
  Swal.fire('Actualización', `Todas las tareas han sido marcadas como ${status}.`, 'info')
}

const resetForm = () => {
  newTask.value = { title: '', description: '', dueDate: '', category: '', completed: false }
  isEditing.value = false
  editingIndex.value = -1
}

const openModal = () => {
  const modal = new window.bootstrap.Modal(document.getElementById('taskModal'))
  modal.show()
}

const closeModal = () => {
  const modal = window.bootstrap.Modal.getInstance(document.getElementById('taskModal'))
  modal.hide()
}

const filterTasks = (criteria) => {
  filter.value = criteria
}

const filteredTasks = computed(() => {
  if (filter.value === 'pending') {
    return tasks.value.filter((task) => !task.completed)
  } else if (filter.value === 'completed') {
    return tasks.value.filter((task) => task.completed)
  }
  return tasks.value
})

const markAsCompleted = (index) => {
  const status = tasks.value[index].completed ? 'Completada' : 'Pendiente'
  Swal.fire('Actualización', `Tarea "${tasks.value[index].title}" marcada como ${status}.`, 'info')
}
</script>

<style scoped>
body {
  background-color: #f8f9fa;
}
.container {
  max-width: 1200px;
}
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
}
.completed-task {
  text-decoration: line-through;
  color: gray;
}
</style>
