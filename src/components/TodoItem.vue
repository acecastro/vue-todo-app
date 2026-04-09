<template>
  <li class="todo-item">
    <!-- Checkbox -->
    <input type="checkbox" v-model="localTodo.completed" @change="emitUpdate" />

    <!-- Display Mode -->
    <span
      v-if="!isEditing"
      :class="{ done: localTodo.completed }"
      @dblclick="startEdit"
    >
      {{ localTodo.text }}
    </span>

    <!-- Edit Mode -->
    <input
      v-else
      v-model="editText"
      @keyup.enter="saveEdit"
      @keyup.esc="cancelEdit"
      @blur="saveEdit"
      ref="editInput"
    />

    <!-- Delete -->
    <button @click="$emit('delete', localTodo.id)">✕</button>
  </li>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'

const props = defineProps({
  todo: Object
})

const emit = defineEmits(['update', 'delete'])

const localTodo = ref({ ...props.todo })
const isEditing = ref(false)
const editText = ref('')
const editInput = ref(null)

// Sync with parent updates
watch(() => props.todo, (newVal) => {
  localTodo.value = { ...newVal }
})

// Emit update
const emitUpdate = () => {
  emit('update', localTodo.value)
}

// Start editing
const startEdit = async () => {
  isEditing.value = true
  editText.value = localTodo.value.text
  await nextTick()
  editInput.value.focus()
}

// Save edit
const saveEdit = () => {
  const text = editText.value.trim()
  if (text) {
    localTodo.value.text = text
    emitUpdate()
  }
  isEditing.value = false
}

// Cancel edit
const cancelEdit = () => {
  isEditing.value = false
}
</script>

<style scoped>
.todo-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
}

.done {
  text-decoration: line-through;
  color: gray;
}

input[type="text"] {
  flex: 1;
  padding: 5px;
}

button {
  background: none;
  border: none;
  cursor: pointer;
  color: red;
}
</style>