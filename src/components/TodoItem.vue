<script setup>
import { ref } from 'vue'
const props = defineProps({
    todo: Object 
})
const emit = defineEmits([
 'delete',
 'update'   
])
const isEditing = ref(false)
const editedTitle = ref(props.todo.title)
function updateTask() {
    if (editedTitle.value.trim() === '') {
        return
    }
    emit('update', {
        ...props.todo,
        title: editedTitle.value
    })
    isEditing.value = false
}
function toggleCompleted() {
    emit('update', {
        ...props.todo,
        completed: !props.todo.completed
    })
    
}
function deleteTask() {
    emit('delete', props.todo.id)
}
function startEditing() {
    editedTitle.value = props.todo.title
    isEditing.value = true
}
</script>


<template>

  <div class="todo-item">

    <!-- Normal Mode -->
    <div v-if="!isEditing">

      <input
        type="checkbox"
        :checked="todo.completed"
        @change="toggleCompleted"
      />

      <span :class="{ completed: todo.completed }">
        {{ todo.title }}
      </span>

      <button @click="startEditing">
        Edit
      </button>

      <button @click="deleteTask">
        Delete
      </button>

    </div>

    <!-- Edit Mode -->
    <div v-else>

      <input
        v-model="editedTitle"
        @keyup.enter="updateTask"
        
      />

      <button @click="updateTask">
        Save
      </button>

      <button @click="isEditing = false">
        Cancel
      </button>

    </div>

  </div>

</template>


<style scoped>
.todo-item {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
.todo-item span {
  margin: 0 10px;
}
.completed {
  text-decoration: line-through;
  color: gray;
}
button {
  margin-left: 5px;
}
</style>