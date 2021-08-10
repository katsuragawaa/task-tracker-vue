<template>
  <AddTask v-show="showAddTask" @add-task="addTask" />
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from '../components/Tasks';
import AddTask from '../components/AddTask';

export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updatedTask = { ...taskToToggle, reminder: !taskToToggle.reminder };
      const response = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updatedTask),
      });
      const result = await response.json();
      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: result.reminder } : task
      );
    },
    async deleteTask(id) {
      if (confirm('Sure?')) {
        const response = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        });

        response.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert('Error deleting task');
      }
    },
    async addTask(task) {
      const response = await fetch('api/tasks', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(task),
      });

      const addedTask = await response.json();

      this.tasks = [...this.tasks, addedTask];
    },
    async fetchTasks() {
      const response = await fetch('api/tasks');
      return await response.json();
    },
    async fetchTask(id) {
      const response = await fetch(`api/tasks/${id}`);
      return await response.json();
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
