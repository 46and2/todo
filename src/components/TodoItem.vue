<template>
  <div class="todo-wrapper">
    <div class="todo" draggable="true" @dragover.stop @dragstart="dragStart">
      <div class="todo-left">
        <input
          class="todo-status"
          type="checkbox"
          v-model="completed"
          @change="toggleCompleted"
        />
        <p class="todo-title" :class="{ completed: completed }">
          {{ todo.title }}
        </p>
      </div>
      <button class="remove-todo" @click="showModal = true">&times;</button>
    </div>

    <confirmation-modal
      v-if="showModal"
      text="Удалить задачу?"
      @confirmed="remove"
      @rejected="showModal = false"
    ></confirmation-modal>
  </div>
</template>

<script>
import { eventBus } from '../main.js';
import ConfirmationModal from './ConfirmationModal.vue';

export default {
  name: 'TodoItem',
  components: { ConfirmationModal },
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      showModal: false,
      completed: this.todo.completed
    };
  },
  methods: {
    toggleCompleted() {
      eventBus.$emit('todo-complete-toggled', {
        id: this.todo.id,
        completed: this.completed
      });
    },
    remove() {
      this.showModal = false;
      eventBus.$emit('todo-removed', this.todo.id);
    },
    dragStart(event) {
      event.dataTransfer.setData('text', this.todo.id);
      // setTimeout(() => (event.target.style.display = 'none'), 0);
    }
  }
};
</script>

<style scoped>
.todo {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #86a59c;
  cursor: grab;
}

.todo-left {
  display: flex;
  align-items: center;
}

.completed {
  color: rgba(0, 0, 0, 0.5);
  text-decoration: line-through;
}

.todo-status {
  cursor: pointer;
}

.todo-title {
  max-width: 480px;
  margin: 0;
  padding: 5px;
}

.remove-todo {
  font-size: 18px;
  cursor: pointer;
  border: none;
  background: transparent;
}
</style>
