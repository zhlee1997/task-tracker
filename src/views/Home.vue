<template>
  <AddTask @add-task="addTask" v-show="isShowAddTask" />
  <Tasks
    @delete-task="deleteTask"
    @toggle-reminder="toggleReminder"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";

export default {
  components: { Tasks, AddTask },
  data() {
    return {
      tasks: [],
    };
  },
  props: {
    isShowAddTask: Boolean,
  },
  methods: {
    async fetchTasks() {
      const res = await fetch("api/tasks");
      return res.json();
    },
    async deleteTask(id) {
      if (confirm("Are you sure to delete?")) {
        const res = await fetch(`api/tasks/${id}`, {
          method: "DELETE",
        });
        if (res.ok) {
          this.tasks = this.tasks.filter((task) => task.id !== id);
        } else {
          alert("Error in delete task");
        }
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      if (taskToToggle) {
        const res = await fetch(`api/tasks/${id}`, {
          method: "PATCH",
          headers: {
            "Content-type": "application/json",
          },
          body: JSON.stringify({
            ...taskToToggle,
            reminder: !taskToToggle.reminder,
          }),
        });
        if (res.ok) {
          const data = await res.json();
          if (data) {
            this.tasks = this.tasks.map((task) =>
              task.id === id ? { ...task, reminder: data.reminder } : task
            );
          }
        }
      }
    },
    async addTask(newTask) {
      const res = await fetch("api/tasks", {
        method: "POST",
        body: JSON.stringify(newTask),
        headers: {
          "Content-type": "application/json",
        },
      });
      if (res) {
        this.tasks = [...this.tasks, newTask];
      }
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      return res.json();
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
