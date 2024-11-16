<template>
  <div class="min-h-screen bg-gray-100 flex flex-col items-center">
    <header class="w-full bg-gradient-to-r from-blue-500 to-indigo-600 py-6 mb-8 shadow-md">
      <h1 class="text-3xl font-bold text-white text-center">To-Do List</h1>
    </header>

    <div class="w-full max-w-2xl px-4">
      <form @submit.prevent="addTask" class="flex gap-4 mb-6">
        <input
            v-model="newTaskTitle"
            placeholder="Task Title"
            required
            class="flex-grow p-3 text-lg border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <select
            v-model="newTaskPriority"
            required
            class="p-3 text-lg border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option disabled value="">Priority</option>
          <option value="Low">Low</option>
          <option value="Medium">Medium</option>
          <option value="High">High</option>
        </select>
        <button
            type="submit"
            class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-3 px-5 rounded-lg transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          Add
        </button>
      </form>

      <p class="text-xl font-medium mb-4">Pending Tasks: {{ pendingTasks }}</p>

      <transition-group name="list" tag="ul" class="space-y-4">
        <li
            v-for="task in sortedTasks"
            :key="task.id"
            :id="'task-' + task.id"
            :class="[
            'flex items-center p-4 bg-white rounded-lg shadow transition-all duration-300',
            {
              'border-l-4 border-red-500': task.priority === 'High',
              'border-l-4 border-yellow-500': task.priority === 'Medium',
              'border-l-4 border-green-500': task.priority === 'Low',
              'opacity-50': task.completed,
            },
          ]"
        >
          <input
              type="checkbox"
              v-model="task.completed"
              @change="taskStatusChanged(task)"
              class="w-6 h-6 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500 cursor-pointer"
          />
          <span
              contenteditable="true"
              @blur="updateTaskTitle(task, $event)"
              @input="onTaskTitleInput(task, $event)"
              class="flex-grow ml-4 text-lg focus:outline-none"
              :class="{ 'line-through text-gray-500': task.completed }"
          >
            {{ task.title }}
          </span>
          <button
              @click="deleteTask(task.id)"
              class="ml-4 bg-red-500 hover:bg-red-600 text-white font-medium py-2 px-4 rounded-lg transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-red-500"
          >
            Delete
          </button>
        </li>
      </transition-group>
    </div>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>

<script setup lang="ts">
import { ref, computed, watch, nextTick } from 'vue';
import { Task } from '~/types/types';

const tasks = ref<Task[]>([]);
const newTaskTitle = ref('');
const newTaskPriority = ref<Task['priority'] | ''>('');

const addTask = async () => {
  if (!newTaskTitle.value || !newTaskPriority.value) return;

  const newTask: Task = {
    id: Date.now(),
    title: newTaskTitle.value,
    priority: newTaskPriority.value as Task['priority'],
    completed: false,
  };

  tasks.value.push(newTask);
  console.log('Task added:', newTask);

  newTaskTitle.value = '';
  newTaskPriority.value = '';

  await nextTick();
  scrollToTask(newTask.id);
};

const deleteTask = (id: number) => {
  tasks.value = tasks.value.filter((task) => task.id !== id);
  console.log('Task deleted:', id);
};

const sortedTasks = computed(() => {
  const priorityOrder = { High: 0, Medium: 1, Low: 2 };
  return [...tasks.value].sort(
      (a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]
  );
});

const pendingTasks = computed<number>(() => tasks.value.filter((task) => !task.completed).length);

watch(
    tasks,
    (newVal, oldVal) => {
      if (newVal.length > oldVal.length) {
        console.log('A task was added.');
      } else if (newVal.length < oldVal.length) {
        console.log('A task was deleted.');
      }
    },
    { deep: true }
);

const taskStatusChanged = (task: Task) => {
  console.log('Task status changed:', task);
};

const updateTaskTitle = (task: Task, event: Event) => {
  const element = event.target as HTMLElement;
  task.title = element.innerText.trim();
};

const onTaskTitleInput = (task: Task, event: Event) => {
  const element = event.target as HTMLElement;
  task.title = element.innerText;
};

const scrollToTask = (id: number) => {
  const element = document.getElementById(`task-${id}`);
  if (element) {
    element.scrollIntoView({ behavior: 'smooth' });
  }
};
</script>
