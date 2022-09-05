<script setup>
import { ref, reactive, watch, computed, onBeforeMount } from "vue";
import TodoForm from "./components/TodoForm.vue";
import TodoItem from "./components/TodoItem.vue";

// Main logic
let todos = ref([]);

function addTodo(todoTitle, todoType, todoMaxDuration) {
  const todoId = todos.value.length
    ? Math.max(...todos.value.map((o) => o.id)) + 1
    : 1;
  const newTodo = {
    id: todoId,
    title: todoTitle,
    type: todoType,
    status: "new",
  };
  if (todoType === "timer") {
    newTodo.currentDuration = 0;
    newTodo.maxDuration = todoMaxDuration;
  }

  isFormOpened.value = false;
  todos.value.push(newTodo);
}

function mutateTodo(todoId, prop, newValue) {
  const index = todos.value.findIndex((todo) => {
    return todo.id === todoId;
  });
  todos.value[index][prop] = newValue;
}

function deleteTodo(todoId) {
  const index = todos.value.findIndex((todo) => {
    return todo.id === todoId;
  });
  todos.value.splice(index, 1);
}

// localStorage
onBeforeMount(() => {
  if (localStorage.getItem("todos")) {
    todos.value =
      JSON.parse(localStorage.getItem("todos")).map((obj) => {
        if (obj.status === "active") {
          return { ...obj, status: "paused" };
        }

        return obj;
      }) || [];
  } else {
    todos.value = [];
  }
});

watch(
  () => todos.value,
  (newValue) => {
    localStorage.setItem("todos", JSON.stringify(newValue));
  },

  { deep: true }
);

// Sorting
const STATUS_ORDER = ["active", "paused", "new", "done"];
const TYPE_ORDER = ["timer", "text"];

const priority = reactive([
  ["status", STATUS_ORDER],
  ["type", TYPE_ORDER],
]);

function sortByPriority(a, b, prop) {
  let propOrder = priority.find((e) => e[0] === prop)[1];

  let pa = propOrder.indexOf(a[prop]);
  let pb = propOrder.indexOf(b[prop]);

  if (pa < pb) {
    return -1;
  }
  if (pa > pb) {
    return 1;
  }
  return 0;
}

const sortedTodos = computed(() => {
  return [...todos.value].sort((a, b) => {
    let result = 0;

    for (let i = 0; i < priority.length; i++) {
      let prop = priority[i][0];
      result = sortByPriority(a, b, prop);
      if (result !== 0) {
        break;
      }
    }
    return result;
  });
});

// Hacks for style
const isFormOpened = ref(false);
const iconStyle = ref("");

watch(isFormOpened, (newValue) => {
  iconStyle.value = newValue
    ? "transform: rotate(45deg); animation: rotate 0.5s"
    : "animation: rotateBack 0.5s";
});

function beforeLeave(el) {
  const { marginLeft, marginTop, width, height } = window.getComputedStyle(el);
  el.style.left = `${el.offsetLeft - parseFloat(marginLeft, 10)}px`;
  el.style.top = `${el.offsetTop - parseFloat(marginTop, 10)}px`;
  el.style.width = width;
  el.style.height = height;
}
function clickOnIcon() {
  isFormOpened.value = !isFormOpened.value;
  document.getElementById("todo-list").scrollTop = 0;
}
</script>

<template>
  <main>
    <div id="wrapper">
      <div id="todo-add" @click="clickOnIcon">
        <i class="las la-plus" :style="iconStyle"></i>
      </div>
      <transition-group
        name="list"
        tag="div"
        id="todo-list"
        @before-leave="beforeLeave"
      >
        <todo-form
          v-if="isFormOpened"
          @requestTodoCreation="addTodo"
        ></todo-form>
        <p v-if="!todos.length">No todos yet!</p>
        <todo-item
          v-else
          v-for="todo in sortedTodos"
          :key="todo.id"
          :todo="todo"
          @requestTodoMutation="mutateTodo"
          @requestTodoDeletion="deleteTodo"
        ></todo-item>
      </transition-group>
    </div>
  </main>
  <footer>
    <!-- TODO: add links -->
    <div>Contact me</div>
    <div>Credits</div>
  </footer>
</template>
