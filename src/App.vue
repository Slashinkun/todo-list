<script setup>
import { ref, computed } from 'vue'

let localstorageTask = localStorage.getItem('tasks')

console.log(localstorageTask)

let tasks = ref(localstorageTask ? JSON.parse(localstorageTask) : [])

const newTask = ref('')
const remainingTasks = computed(() => {
  tasks.value.filter((task) => !task.completed)
})
const remainingCount = ref(remainingTasks.length)

const sorting = (a, b) => Number(b.completed) - Number(a.completed)

const sortedTasks = computed(() => [...tasks.value].sort(sorting).reverse())

const saveTask = () => {
  tasks.value.push({
    id: tasks.value.length + 1,
    name: newTask.value,
  })
  newTask.value = ''
  saveTaskStorage()
}

const deleteTask = (id) => {
  const index = tasks.value.indexOf(id)
  tasks.value.splice(index - 1, 1)
  saveTaskStorage()
}

const taskCompleted = (task) => {
  task.completed = !task.completed
  saveTaskStorage()
}

const saveTaskStorage = () => localStorage.setItem('tasks', JSON.stringify(tasks.value))
</script>

<template>
  <h1>Todo-list</h1>
  <p>Tâches totales : {{ tasks.length }}</p>
  <p>{{ remainingCount }}</p>
  <!-- <button @click="saveTaskStorage">Save tasks in local storage</button> -->
  <div class="container">
    <form class="add-task-form" @submit.prevent="saveTask">
      <input type="text" v-model.trim="newTask" placeholder="New task" />
      <button v-bind:disabled="newTask.length === 0" class="btn btn-primary">Add task</button>
    </form>
    <ul>
      <li
        class="list-element"
        @click="taskCompleted(task)"
        v-for="task in sortedTasks"
        :key="task.id"
        :class="{ strikeout: task.completed }"
      >
        {{ task.name }}<button @click="deleteTask(task.id)">Delete</button>
      </li>
    </ul>
    <p v-if="!tasks.length">No tasks to do</p>
  </div>
</template>

<style scoped>
.strikeout {
  text-decoration: line-through;
  opacity: 50%;
}

button:hover {
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}
</style>
