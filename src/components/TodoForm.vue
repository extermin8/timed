<script setup>
import { ref, watch, onMounted, nextTick } from "vue";

const emit = defineEmits(["requestTodoCreation"]);

const inputType = ref("text");
const inputTitle = ref("");
const inputTime = ref(null); //in minutes

function process() {
  if (inputTitle.value === "") {
    return;
  }
  if (inputType.value === "timer") {
    if (
      inputTime.value == 0 ||
      inputTime.value === null ||
      inputTime.value === undefined
    ) {
      return;
    }
    emit(
      "requestTodoCreation",
      inputTitle.value,
      inputType.value,
      inputTime.value * 60000 // convert to milliseconds;
    );
  } else {
    emit("requestTodoCreation", inputTitle.value, inputType.value);
  }
}

// Focus on input
const textInput = ref(null);

function focusOnTitle() {
  nextTick(() => textInput.value.focus());
}

onMounted(() => {
  focusOnTitle();
});

watch(inputType, (newType) => {
  focusOnTitle();
  if (newType === "time") {
    inputTime.value = 0;
  }
  if (newType === "text") {
    inputTime.value = null;
  }
});
</script>
<template>
  <form class="todo-item" @submit.prevent="process">
    <i
      :class="['lar', inputType === 'timer' ? 'la-clock' : 'la-circle']"
      @click="
        inputType === 'timer' ? (inputType = 'text') : (inputType = 'timer')
      "
    ></i>
    <input
      v-model.trim="inputTitle"
      placeholder="Type your todo"
      type="text"
      class="todo-title"
      ref="textInput"
    />
    <input
      v-if="inputType === 'timer'"
      class="todo-title"
      placeholder="and time in minutes"
      v-model.number="inputTime"
      type="text"
      inputmode="numeric"
    />
    <button>
      <i class="lar la-save"></i>
    </button>
  </form>
</template>
