<script setup>
import { ref, computed, watch, onMounted } from "vue";

// Reactive State
const newTodo = ref("");
const todos = ref([]);
const filter = ref("all");

// Load stored todos when the app starts
onMounted(() => {
  const storedTodos = localStorage.getItem("todos");
  if (storedTodos) {
    todos.value = JSON.parse(storedTodos);
  }
});

// Watch todos and update localStorage whenever they change
watch(
  todos,
  (newTodos) => {
    localStorage.setItem("todos", JSON.stringify(newTodos));
  },
  { deep: true }
);

// Computed properties
const filteredTodos = computed(() => {
  if (filter.value === "active") {
    return todos.value.filter((todo) => !todo.completed);
  } else if (filter.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  }
  return todos.value;
});

const remaining = computed(
  () => todos.value.filter((todo) => !todo.completed).length
);
const hasCompleted = computed(() => todos.value.some((todo) => todo.completed));

// Methods
const addTodo = () => {
  const text = newTodo.value.trim();
  if (text) {
    todos.value.push({ text, completed: false });
    newTodo.value = "";
  }
};

const removeTodo = (index) => {
  todos.value.splice(index, 1);
};

const clearCompleted = () => {
  todos.value = todos.value.filter((todo) => !todo.completed);
};
</script>

<template>
  <div class="todo-app container">
    <h2 class="text-center fw-bold text-dark display-4 mx-12">Todo List</h2>

    <!-- Input Box -->
    <div class="input-group shadow-sm mb-4">
      <input
        type="text"
        class="form-control input-lg"
        placeholder="What needs to be done?"
        v-model="newTodo"
        @keyup.enter="addTodo"
      />
      <button class="btn btn-primary" @click="addTodo">Add</button>
    </div>

    <!-- Todo List -->
    <ul class="list-group mb-4">
      <li
        v-for="(todo, index) in filteredTodos"
        :key="index"
        class="list-group-item d-flex todo-item"
      >
        <input
          type="checkbox"
          class="form-check-input me-3 custom-checkbox"
          v-model="todo.completed"
        />
        <span :class="{ completed: todo.completed }" class="flex-grow-1">
          {{ todo.text }}
        </span>
        <button class="btn btn-danger btn-sm" @click="removeTodo(index)">
          Delete
        </button>
      </li>
    </ul>

    <!-- Footer: Filter Options -->
    <div class="d-flex justify-content-between align-items-center">
      <span>{{ remaining }} items left</span>
      <div>
        <button
          class="btn btn-outline-dark btn-sm filter-btn me-2"
          :class="{ active: filter === 'all' }"
          @click="filter = 'all'"
        >
          All
        </button>
        <button
          class="btn btn-outline-dark btn-sm filter-btn me-2"
          :class="{ active: filter === 'active' }"
          @click="filter = 'active'"
        >
          Active
        </button>
        <button
          class="btn btn-outline-dark btn-sm filter-btn"
          :class="{ active: filter === 'completed' }"
          @click="filter = 'completed'"
        >
          Completed
        </button>
      </div>
    </div>

    <!-- Clear Completed Button -->
    <div class="text-center mt-4">
      <button
        class="btn btn-warning shadow-sm"
        @click="clearCompleted"
        :disabled="!hasCompleted"
      >
        Clear Completed
      </button>
    </div>
  </div>
</template>
