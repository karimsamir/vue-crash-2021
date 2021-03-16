<template>
  <div class="container">
    <Header
      @toggle-add-task="toggleAddTask"
      title="Task Tracker"
      :showAddTask="showAddTask"
    />
    <div v-show="showAddTask">
      <AddTask @add-task="addTask" />
    </div>
    <Tasks
      @toggle-reminder="toggleReminder"
      @delete-task="deleteTask"
      :tasks="tasks"
    />
  </div>
</template>

<script>
import AddTask from "./components/AddTask";
import Header from "./components/Header";
import Tasks from "./components/Tasks";

export default {
  name: "App",
  components: {
    Header,
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
      showAddTask: false,
    };
  },
  methods: {
    async deleteTask(id) {
      if (confirm("Are you sure?")) {
        const res = await fetch(`api/tasks/${id}`, {
          method: "Delete",
        });

        if (res.status === 200) {
          this.tasks = this.tasks.filter((task) => task.id !== id);
        } else {
          alert("Error deleting task");
        }
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      console.log(taskToToggle);
      const updateTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

      const res = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "content-type": "application/json",
        },
        body: JSON.stringify(updateTask),
      });

      const data = await res.json();
      // console.info("toggleReminder", id);
      this.tasks = this.tasks.map((task) => {
        return task.id === id ? { ...task, reminder: data.reminder } : task;
      });
    },
    toggleAddTask() {
      this.showAddTask = !this.showAddTask;
    },
    async addTask(task) {
      const res = await fetch("api/tasks", {
        method: "Post",
        headers: {
          "content-type": "application/json",
        },
        body: JSON.stringify(task),
      });

      const data = await res.json();
      // console.info("toggleReminder", id);
      this.tasks = [...this.tasks, data];
    },
    async fetchTasks() {
      const rest = await fetch("api/tasks");

      const data = await rest.json();

      return data;
    },
    async fetchTask(id) {
      const rest = await fetch(`api/tasks/${id}`);

      const data = await rest.json();

      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400&display=swap");
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  font-family: "Poppins", sans-serif;
}
.container {
  max-width: 500px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
  border: 1px solid steelblue;
  padding: 30px;
  border-radius: 5px;
}
.btn {
  display: inline-block;
  background: #000;
  color: #fff;
  border: none;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  font-size: 15px;
  font-family: inherit;
}
.btn:focus {
  outline: none;
}
.btn:active {
  transform: scale(0.98);
}
.btn-block {
  display: block;
  width: 100%;
}
</style>
