<script setup>
import { ref, computed } from 'vue'

let localstorageTask = localStorage.getItem('tasks')

console.log(localstorageTask)

let tasks = ref(localstorageTask ? JSON.parse(localstorageTask) : [])

let editingTaskId = ref(null)
let editingTaskName = ref('')

let tasksFilter = ref('all')

const filteredTasks = computed(() => {
  if (tasksFilter.value === 'completed') return tasks.value.filter((t) => t.completed)
  if (tasksFilter.value === 'active') return tasks.value.filter((t) => !t.completed)
  return sortedTasks.value
})

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
  tasks.value.splice(index, 1)
  saveTaskStorage()
}

const taskCompleted = (task) => {
  task.completed = !task.completed
  saveTaskStorage()
}

const editTask = (task) => {
  if (editingTaskName.value.length > 0) {
    task.name = editingTaskName
    editingTaskId = null
    editingTaskName = ''
    saveTaskStorage()
  } else {
    alert('You cannot edit a task with an empty name')
  }
}

const deleteAllTasks = () => {
  if (
    confirm(
      'Warning : This action is irreversible and will lead to the deletion of all your tasks.\nAre you sure to do this ?',
    ) === true
  ) {
    localStorage.setItem('tasks', JSON.stringify([]))
    tasks.value = []
  }
}

const saveTaskStorage = () => localStorage.setItem('tasks', JSON.stringify(tasks.value))
</script>

<template>
  <h1>Todo List</h1>
  <p>Tasks : {{ tasks.length }}</p>
  <p>Todos : {{ remainingCount }}</p>
  <div>
    <button @click="tasksFilter = 'all'">All</button
    ><button @click="tasksFilter = 'completed'">Completed</button
    ><button @click="tasksFilter = 'active'">Active</button>
  </div>

  <!-- <button @click="saveTaskStorage">Save tasks in local storage</button> -->
  <div class="container">
    <form class="add-task-form" @submit.prevent="saveTask">
      <input type="text" v-model.trim="newTask" placeholder="New task" />
      <button v-bind:disabled="newTask.length === 0" class="btn btn-primary">Add task</button>
    </form>
    <button type="button" @click="deleteAllTasks">Delete All</button>
    <ul>
      <li
        class="list-element"
        @click="taskCompleted(task)"
        v-for="task in filteredTasks"
        :key="task.id"
        :class="{ strikeout: task.completed, todo: !task.completed }"
      >
        {{ task.name }}
        <input
          type="text"
          v-if="editingTaskId === task.id"
          v-model.trim="editingTaskName"
          @click.stop
          @keyup.enter="editTask(task)"
        />
        <button @click.stop="deleteTask(task.id)">Delete 🗑️</button>
        <button @click.stop="editingTaskId = task.id" v-bind:disabled="editingTaskId === task.id">
          Edit ✏️
        </button>
        <button v-if="editingTaskId === task.id" @click.stop="editTask(task)">Confirm ✔️</button>
        <button v-if="editingTaskId === task.id" @click.stop="editingTaskId = null">
          Cancel ❌
        </button>
      </li>
    </ul>
    <p v-if="!tasks.length">No tasks to do</p>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.strikeout {
  text-decoration: line-through;
  opacity: 50%;
}

.todo {
  color: orange;
}

form > button:hover {
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
