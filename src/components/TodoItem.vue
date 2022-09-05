<script setup>
import { ref, watch, computed } from "vue";

const props = defineProps({
  todo: {
    type: Object,
    required: true,
    validator: function (obj) {
      const STATUS_ORDER = ["active", "paused", "new", "done"];
      const TYPE_ORDER = ["timer", "text"];
      if (
        obj.id !== undefined &&
        obj.title !== undefined &&
        obj.type !== undefined &&
        obj.status !== undefined
      ) {
        if (
          TYPE_ORDER.includes(obj.type) &&
          STATUS_ORDER.includes(obj.status)
        ) {
          if (
            obj.type === "timer" &&
            obj.maxDuration !== undefined &&
            obj.currentDuration !== undefined
          ) {
            return true;
          }
          if (
            obj.type === "text" &&
            obj.maxDuration === undefined &&
            obj.currentDuration === undefined
          ) {
            return true;
          }
        }
      }
      return false;
    },
  },
});

const emit = defineEmits(["requestTodoDeletion", "requestTodoMutation"]);

function toggleStatus() {
  emit(
    "requestTodoMutation",
    props.todo.id,
    "status",
    props.todo.status === "done" ? "new" : "done"
  );
}

// Timer
const timerId = ref(null);

watch(
  () => props.todo.status,
  (newStatus) => {
    if (newStatus === "active") {
      timerId.value = setInterval(() => {
        emit(
          "requestTodoMutation",
          props.todo.id,
          "currentDuration",
          props.todo.currentDuration + 1000
        );
      }, 1000);
    } else {
      clearInterval(timerId.value);
    }
  }
);

function startTimer() {
  // Reset current duration, if timer restarts
  if (props.todo.status === "done") {
    emit("requestTodoMutation", props.todo.id, "currentDuration", 0);
  }
  emit("requestTodoMutation", props.todo.id, "status", "active");
}

function pauseTimer() {
  emit("requestTodoMutation", props.todo.id, "status", "paused");
}

// Animation at the end of the countdown
const animateCongrats = ref(false);

watch(
  () => props.todo.currentDuration,
  (val) => {
    if (val === props.todo.maxDuration) {
      clearInterval(timerId.value);
      animateCongrats.value = true;
      setTimeout(() => {
        emit("requestTodoMutation", props.todo.id, "status", "done");
        animateCongrats.value = false;
      }, 1000);
    }
  }
);

// Milliseconds to HH:MM:SS
const timeString = computed(() => {
  if (props.todo.type === "timer") {
    let milliseconds = props.todo.maxDuration - props.todo.currentDuration;
    let seconds = Math.floor(milliseconds / 1000);
    let minutes = Math.floor(seconds / 60);
    let hours = Math.floor(minutes / 60);

    seconds = seconds % 60;
    minutes = minutes % 60;
    hours = hours % 24;

    return `${padTo2Digits(hours)}:${padTo2Digits(minutes)}:${padTo2Digits(
      seconds
    )}`;
  }
  return null;
});

function padTo2Digits(num) {
  return num.toString().padStart(2, "0");
}
</script>
<template>
  <div
    class="todo-item"
    :class="{
      done: todo.status === 'done',
      active: todo.status === 'active',
      'animate-congrats': animateCongrats,
    }"
  >
    <i
      class="lar"
      :class="{
        'la-play-circle': todo.type === 'timer' && todo.status !== 'active',
        'la-pause-circle': todo.type === 'timer' && todo.status === 'active',
        'la-circle': todo.type === 'text' && todo.status === 'new',
        'la-check-circle': todo.status === 'done',
      }"
      @click="
        todo.type === 'timer'
          ? todo.status === 'active'
            ? pauseTimer()
            : startTimer()
          : toggleStatus()
      "
    ></i>
    <span v-text="todo.title" class="todo-title"></span>
    <span
      v-if="todo.type === 'timer'"
      class="todo-left"
      v-text="timeString"
    ></span>
    <i
      class="lar la-trash-alt"
      @click="emit('requestTodoDeletion', todo.id)"
    ></i>
  </div>
</template>
