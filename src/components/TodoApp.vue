<template>
  <div class="container">
    <h1>Todo App</h1>

    <!-- Add Todo -->
    <div class="input-group">
      <input
        v-model="newTodo"
        @keyup.enter="addTodo"
        placeholder="Add a new task..."
      />
      <button @click="addTodo">Add</button>
    </div>

    <!-- Filters -->
    <div class="filters">
      <button :class="{ active: filter === 'all' }" @click="filter = 'all'">All</button>
      <button :class="{ active: filter === 'active' }" @click="filter = 'active'">Active</button>
      <button :class="{ active: filter === 'completed' }" @click="filter = 'completed'">Completed</button>
    </div>

    <!-- Empty State -->
    <p v-if="filteredTodos.length === 0" class="empty">
      No tasks yet. Add one!
    </p>

    <!-- Todo List -->
    <transition-group name="fade" tag="ul">
      <TodoItem
        v-for="todo in filteredTodos"
        :key="todo.id"
        :todo="todo"
        @update="updateTodo"
        @delete="deleteTodo"
      />
    </transition-group>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import TodoItem from './TodoItem.vue'

const newTodo = ref('')
const todos = ref([])
const filter = ref('all')

// Load from localStorage
onMounted(() => {
  const saved = localStorage.getItem('todos')
  if (saved) {
    todos.value = JSON.parse(saved)
  }
})

// Save to localStorage
watch(todos, (val) => {
  localStorage.setItem('todos', JSON.stringify(val))
}, { deep: true })

// Add todo
const addTodo = () => {
  const text = newTodo.value.trim()
  if (!text) return

  todos.value.unshift({
    id: Date.now(),
    text,
    completed: false
  })

  newTodo.value = ''
}

// Delete todo
const deleteTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}

// Update todo
const updateTodo = (updated) => {
  const index = todos.value.findIndex(t => t.id === updated.id)
  if (index !== -1) {
    todos.value[index] = updated
  }
}

// Filter logic
const filteredTodos = computed(() => {
  if (filter.value === 'active') {
    return todos.value.filter(t => !t.completed)
  }
  if (filter.value === 'completed') {
    return todos.value.filter(t => t.completed)
  }
  return todos.value
})
</script>

<style scoped>
.container {
  max-width: 400px;
  margin: 50px auto;
  font-family: Arial;
}

h1 {
  text-align: center;
}

.input-group {
  display: flex;
  gap: 10px;
}

input {
  flex: 1;
  padding: 8px;
}

button {
  padding: 8px 12px;
  cursor: pointer;
}

.filters {
  margin: 10px 0;
  display: flex;
  gap: 5px;
}

.filters button.active {
  font-weight: bold;
}

.empty {
  text-align: center;
  color: gray;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
</style>