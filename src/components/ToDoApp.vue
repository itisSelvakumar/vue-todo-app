<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <div class="header">
      <h3>
        Todo's List for <em>{{ userName }}</em>
      </h3>

      <select @change="fetchTodos($event)">
        <option :selected="true">Select user</option>
        <option v-for="user in users" :key="user.id" v-bind:value="user.id">
          {{ user.name }}
        </option>
      </select>
    </div>

    <div class="todos">
      <div
        v-for="(todoItems, index) in todosGroup"
        :key="todoItems.id"
        :index="index"
        :class="index"
      >
        <h2>{{ index == "true" ? "Completed" : "Open" }}</h2>
        <ul>
          <li v-for="todo in todoItems" :key="todo.id">
            <input
              type="checkbox"
              :value="todo.id"
              :id="'checkbox-' + todo.id"
              :name="'checkbox-' + todo.id"
              v-model="todo.completed"
              @change="updateTodos(todo.id)"
              :disabled="index == 'true'"
            />
            <label :for="'checkbox-' + todo.id">
              {{ todo.title }} ({{ todo.id }})
            </label>
            <em v-if="index == 'false'" :id="'msg-' + todo.id">
              <small>Todo is updated successfully</small>
            </em>
          </li>
        </ul>
      </div>
    </div>

    <div class="todo-add">
      <form @submit.prevent="addTodo">
        <label>New todo</label>
        <input name="newTodo" v-model="newTodo" />
        <button
          :disabled="isDisabled || !newTodo"
          title="Select user and add todo list."
        >
          Add new todo
        </button>

        <div id="success">Addess successfully...!</div>
      </form>

      <pre>
        {{ newTodoItem }}
      </pre>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { ref, onMounted } from "vue";

export default defineComponent({
  name: "ToDoApp",
  props: {
    msg: String,
  },
  setup() {
    const users = ref(null);
    const allTodos = ref(null);
    const todos = ref(null);
    const todosGroup = ref(null);
    const user = ref(null);
    const userName = ref(null);
    const newTodo = ref(null);
    var isDisabled = ref(true);
    const newTodoItem = ref(null);

    /**
     * Add new todo item and update in previous list
     */
    function addTodo() {
      fetch("https://jsonplaceholder.typicode.com/todos", {
        method: "POST",
        body: JSON.stringify({
          title: newTodo.value,
          userId: user.value,
          completed: false,
        }),
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      })
        .then((response) => response.json())
        .then((json) => (newTodoItem.value = json))
        .then((json) => {
          // push new todo into existing array
          todos.value.push(json);
          todosGroup.value = groupBy(todos.value);
          document.getElementById("success").style.display = "block";

          setTimeout(() => {
            document.getElementById("success").style.display = "none";
          }, 1000);
        });
    }

    /**
     * Group by - grouping the todo items based on the status
     */
    function groupBy(data) {
      const groups = data.reduce(
        (groups, item) => ({
          ...groups,
          [item.completed]: [...(groups[item.completed] || []), item],
        }),
        {}
      );
      return groups;
    }

    /**
     * Load the todo lists from jsonplaceholder using REST API
     */
    function fetchTodos(event) {
      if (event) {
        user.value = event.target.value;
        userName.value =
          event.target.options[event.target.options.selectedIndex].text;
      }
      isDisabled.value = user.value ? false : true;

      fetch(`https://jsonplaceholder.typicode.com/users/${user.value}/todos`)
        .then((response) => response.json())
        .then((json) => {
          todos.value = json;
          todosGroup.value = groupBy(json);
        });
    }

    /**
     * Update the todo item on click of each checkbox and update the status as completed
     */
    function updateTodos(id) {
      Object.keys(todos.value).forEach((key, index) => {
        if (todos.value[key].id == id) {
          todos.value[key].completed = true;
        }
      });

      todosGroup.value = groupBy(todos.value);

      /* fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
        method: "PUT",
        body: JSON.stringify({
          id: id,
          completed: true,
        }),
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      })
        .then((response) => response.json())
        .then((json) => console.log(json))
        .then((json) => {
          // document.getElementById("msg-" + id).style.display = "block";
          // setTimeout(() => {
          // document.getElementById("msg-" + id).style.display = "none";
          // }, 1000);
        }); */
    }

    /**
     * Fetch list of users from json server using REST API
     */
    function fetchUsers() {
      fetch("https://jsonplaceholder.typicode.com/users")
        .then((response) => response.json())
        .then((json) => (users.value = json));
    }

    onMounted(() => {
      //
    });

    // fetchTodos();

    fetchUsers();
    return {
      newTodo,
      newTodoItem,
      isDisabled,
      users,
      todos,
      todosGroup,
      allTodos,
      fetchTodos,
      user,
      userName,
      addTodo,
      updateTodos,
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: flex;
  margin: 0 10px;
  text-align: left;
  padding: 10px;
  label {
    margin-left: 5px;
  }
  em {
    display: none;
    color: green;
  }
}
a {
  color: #42b983;
}
.header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin: 0 30px;
  select {
    font-size: 20px;
  }
}
.todo-add {
  border: 2px solid blue;
  border-radius: 10px;
  padding: 10px;
  text-align: left;
  margin: 25px;
  label {
    display: block;
    font-size: 22px;
    margin-bottom: 15px;
  }
  input {
    display: block;
    margin-bottom: 15px;
    padding: 6px;
    font-size: 20px;
  }
  button {
    border: 1px solid #000;
    background-color: #333;
    color: #fff;
    border-radius: 3px;
    padding: 15px;
    margin-bottom: 15px;
    width: 250px;
    display: block;
    &:disabled {
      border: 1px solid #777;
      background-color: #999;
      cursor: not-allowed;
    }
  }
  #success {
    display: none;
    color: green;
  }
}

.todos {
  display: flex;
  align-items: flex-start;
}

.false,
.true {
  border: 2px solid orange;
  border-radius: 10px;
  flex: 50%;
  padding: 10px;
  margin: 20px;
}

.true {
  border-color: green;
}

@media (max-width: 800px) {
  .flex-item-right,
  .flex-item-left {
    flex: 100%;
  }
}
</style>
