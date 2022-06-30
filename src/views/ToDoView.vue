<template>
  <div class="todo">
    <ToDoApp
      msg="Vue.js 3 + Composition API + REST API + TypeScript ToDo App"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import ToDoApp from "@/components/ToDoApp.vue"; // @ is an alias to /src

export default defineComponent({
  name: "ToDoView",
  components: {
    ToDoApp,
  },
});
</script>
