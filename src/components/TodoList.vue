<template>
  <section @dragover.prevent @drop.prevent="drop">
    <div class="container">
      <h1>Todo list</h1>
      <div class="add-items">
        <div class="add-control add-todo">
          <input
            v-model="newTodo"
            class="add-title new-todo-title"
            @keyup.enter="handleAdding"
            type="text"
            placeholder="Добавить задачу"
          />
          <button class="add-button add-todo-button" @click="handleAdding">
            +
          </button>
        </div>
        <div class="add-control add-group">
          <input
            v-model="newGroup"
            class="add-title new-group-title"
            @keyup.enter="addGroup"
            type="text"
            placeholder="Добавить группу задач"
          />
          <button class="add-button add-group-button" @click="addGroup">
            +
          </button>
        </div>
      </div>

      <div class="filters">
        <input
          v-model="titleFilter"
          class="title-filter"
          type="text"
          placeholder="Фильтр по задачам"
        />
        <div class="status-filters">
          <button
            class="status-filter-toggle toggle-all"
            :class="{ 'filter-active': statusFilter === 'all' }"
            @click="statusFilter = 'all'"
          >
            Все
          </button>
          <button
            class="status-filter-toggle toggle-completed"
            :class="{ 'filter-active': statusFilter === 'completed' }"
            @click="statusFilter = 'completed'"
          >
            Завершенные
          </button>
          <button
            class="status-filter-toggle toggle-active"
            :class="{ 'filter-active': statusFilter === 'active' }"
            @click="statusFilter = 'active'"
          >
            Активные
          </button>
        </div>
      </div>

      <div class="main-list">
        <todo-group
          v-for="group in todoList.groups"
          :key="'group_' + group.id"
          :group="group"
          :todos="filteredTodos"
        ></todo-group>

        <todo-item
          v-for="todo in commonTodos"
          :key="'todo_' + todo.id"
          :todo="todo"
        ></todo-item>

        <p v-if="!commonTodos.length && !todoList.groups.length">
          Нет задач по текущим фильтрам...
        </p>
      </div>
    </div>
  </section>
</template>

<script>
import { eventBus } from '../main.js';
import TodoGroup from './TodoGroup.vue';
import TodoItem from './TodoItem.vue';

const STORAGE_KEY = 'todo-list-vue';
const todoStorage = {
  fetch: function() {
    return JSON.parse(
      localStorage.getItem(STORAGE_KEY) || '{"todos":[], "groups":[]}'
    );
  },
  save: function(todoList) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todoList));
  }
};

const filters = {
  all(todos) {
    return todos;
  },
  completed(todos) {
    return todos.filter(todo => todo.completed);
  },
  active(todos) {
    return todos.filter(todo => !todo.completed);
  }
};

export default {
  name: 'TodoList',
  components: {
    TodoGroup,
    TodoItem
  },
  data() {
    return {
      newTodo: '',
      newGroup: '',
      statusFilter: 'all',
      titleFilter: '',
      todoList: todoStorage.fetch()
    };
  },
  watch: {
    todoList: {
      handler: function(todoList) {
        todoStorage.save(todoList);
      },
      deep: true
    }
  },
  computed: {
    filteredTodos() {
      let filteredByStatus = filters[this.statusFilter](this.todoList.todos);
      let filteredByTitle = this.titleFilter
        ? filteredByStatus.filter(
            todo =>
              todo.title
                .toLowerCase()
                .indexOf(this.titleFilter.toLowerCase()) !== -1
          )
        : filteredByStatus;

      return filteredByTitle;
    },
    commonTodos() {
      return this.filteredTodos.filter(todo => !todo.groupId);
    }
  },
  methods: {
    handleAdding() {
      this.addTodo(this.newTodo);
    },
    addTodo(title, groupId = null) {
      if (title.trim() === '') {
        return;
      }
      this.todoList.todos.push({
        id: this.todoList.todos.length + 1,
        title,
        groupId,
        completed: false
      });
      this.newTodo = '';
    },
    removeTodo(id) {
      this.todoList.todos = this.todoList.todos.filter(todo => todo.id !== id);
    },
    toggleTodoStatus(id, completed) {
      let index = this.todoList.todos.findIndex(todo => todo.id === id);
      if (index < 0) {
        return;
      }
      let foundTodo = this.todoList.todos[index];
      this.todoList.todos.splice(index, 1, { ...foundTodo, completed });
    },
    addGroup() {
      if (this.newGroup.trim() === '') {
        return;
      }
      this.todoList.groups.push({
        id: this.todoList.groups.length + 1,
        title: this.newGroup,
        color: this.getRandomColor()
      });
      this.newGroup = '';
    },
    removeGroup(id) {
      let thisGroupTodos = this.todoList.todos.filter(
        todo => todo.groupId === id
      );
      let idsToRemove = thisGroupTodos.map(todo => todo.id);
      this.todoList.todos = this.todoList.todos.filter(
        todo => !idsToRemove.includes(todo.id)
      );
      this.todoList.groups = this.todoList.groups.filter(
        group => group.id !== id
      );
    },
    changeGroup(todoId, newGroupId) {
      let foundIndex = this.todoList.todos.findIndex(
        todo => todo.id === todoId
      );
      if (foundIndex < 0) {
        return;
      }
      let foundTodo = this.todoList.todos[foundIndex];
      this.todoList.todos.splice(foundIndex, 1, {
        ...foundTodo,
        groupId: newGroupId
      });
    },
    drop(event) {
      let draggedTodoId = event.dataTransfer.getData('text');
      eventBus.$emit('todo-group-changed', {
        todoId: +draggedTodoId,
        groupId: null
      });
    },
    getRandomColor() {
      let color = Math.floor(Math.random() * 16777215).toString(16);
      return `#${color}`;
    }
  },
  created() {
    eventBus.$on('todo-removed', id => this.removeTodo(id));
    eventBus.$on('todo-complete-toggled', ({ id, completed }) =>
      this.toggleTodoStatus(id, completed)
    );
    eventBus.$on('grouped-todo-added', ({ title, groupId }) =>
      this.addTodo(title, groupId)
    );
    eventBus.$on('group-removed', id => this.removeGroup(id));
    eventBus.$on('todo-group-changed', ({ todoId, groupId }) =>
      this.changeGroup(todoId, groupId)
    );
  }
};
</script>

<style>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 0 10px;
}

.add-items {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 15px 0;
  border-bottom: 1px solid #86a59c;
}

.add-control {
  width: 100%;
}

.add-title {
  padding: 5px;
  font-size: inherit;
  font-family: inherit;
  border: 1px solid #86a59c;
  border-radius: 3px 0 0 3px;
}

.add-todo {
  margin-bottom: 5px;
}

.add-button {
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

.filters {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 15px 0;
  margin-bottom: 20px;
  border-bottom: 1px solid #86a59c;
}

.title-filter {
  margin-bottom: 5px;
  padding: 5px;
  font-size: inherit;
  font-family: inherit;
  border: 1px solid #86a59c;
  border-radius: 5px;
  outline: none;
}

.status-filter-toggle {
  width: 33.3%;
  padding: 5px;
  border: 1px solid #86a59c;
  cursor: pointer;
  background: transparent;
  font-size: inherit;
  font-family: inherit;
  outline: none;
}

.toggle-all {
  border-radius: 5px 0 0 5px;
  border-right: none;
}

.toggle-active {
  border-radius: 0 5px 5px 0;
  border-left: none;
}

.filter-active {
  color: #ffffff;
  background-color: seagreen;
}

@media (min-width: 540px) {
  .add-items {
    flex-direction: row;
  }
  .add-control {
    width: initial;
  }
  .add-todo {
    margin-bottom: 0;
  }
  .filters {
    flex-direction: row;
  }
  .status-filter-toggle {
    width: initial;
  }
}
</style>
