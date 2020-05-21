<template>
  <div
    class="todo-group"
    :style="groupStyle"
    @dragover.prevent
    @drop.prevent.stop="drop"
  >
    <div class="group-header">
      <h2>{{ group.title }}</h2>
      <button class="remove-group" @click="showModal = true">&times;</button>
    </div>
    <div class="group-controls">
      <input
        v-model="newGroupedTodo"
        @keyup.enter="addGroupedTodo"
        class="group-new-todo-title"
        type="text"
        placeholder="Добавить задачу"
      />
      <button class="add-grouped-todo" @click="addGroupedTodo">+</button>
    </div>
    <div class="group-todos">
      <todo-item
        v-for="todo in currentGroupTodos"
        :key="'todo_' + todo.id"
        :todo="todo"
      ></todo-item>

      <p v-if="!currentGroupTodos.length">Пусто...</p>
    </div>

    <confirmation-modal
      v-if="showModal"
      text="Удалить группу?"
      @confirmed="remove"
      @rejected="showModal = false"
    ></confirmation-modal>
  </div>
</template>

<script>
import { eventBus } from '../main.js';
import ConfirmationModal from './ConfirmationModal.vue';
import TodoItem from './TodoItem.vue';

export default {
  name: 'TodoGroup',
  components: {
    ConfirmationModal,
    TodoItem
  },
  props: {
    group: {
      type: Object,
      required: true
    },
    todos: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      showModal: false,
      showTodoForm: false,
      newGroupedTodo: ''
    };
  },
  computed: {
    currentGroupTodos() {
      return this.todos.filter(todo => todo.groupId === this.group.id);
    },
    groupStyle() {
      return {
        '--group-color': this.group.color
      };
    }
  },
  methods: {
    addGroupedTodo() {
      eventBus.$emit('grouped-todo-added', {
        title: this.newGroupedTodo,
        groupId: this.group.id
      });
      this.newGroupedTodo = '';
    },
    remove() {
      this.showModal = false;
      eventBus.$emit('group-removed', this.group.id);
    },
    drop(event) {
      let draggedTodoId = event.dataTransfer.getData('text');
      eventBus.$emit('todo-group-changed', {
        todoId: +draggedTodoId,
        groupId: this.group.id
      });
    }
  }
};
</script>

<style scoped>
.todo-group {
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid;
  border-color: var(--group-color);
  border-radius: 5px;
}

.group-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.group-controls {
  display: flex;
  justify-content: flex-start;
  margin-bottom: 15px;
}

.group-new-todo-title {
  padding: 5px;
  font-size: inherit;
  font-family: inherit;
  border: 1px solid #86a59c;
  border-radius: 3px 0 0 3px;
}

.add-grouped-todo {
  min-width: 30px;
  padding: 5px;
  font-size: inherit;
  font-family: inherit;
  cursor: pointer;
  background: transparent;
  border: 1px solid #86a59c;
  border-left: none;
  border-radius: 0 3px 3px 0;
}

.remove-group {
  font-size: 24px;
  cursor: pointer;
  border: none;
  background: transparent;
}

.group-todos {
  min-height: 55px;
}
</style>
