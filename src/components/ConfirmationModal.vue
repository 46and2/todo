<template>
  <div class="modal-mask">
    <div class="modal-wrapper">
      <div class="confirmation-modal">
        <p class="confirmation-text">{{ text }}</p>
        <div class="modal-controls">
          <button class="modal-button confirm-button" @click="confirm">
            OK
          </button>
          <button class="modal-button reject-button" @click="reject">
            Отмена
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ConfirmationModal',
  props: {
    text: {
      type: String,
      required: true
    }
  },
  methods: {
    confirm() {
      this.$emit('confirmed');
    },
    reject() {
      this.$emit('rejected');
    }
  }
};
</script>

<style scoped>
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.confirmation-modal {
  width: 250px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
}

.confirmation-text {
  text-align: center;
  font-size: 20px;
  font-weight: 500;
}

.modal-controls {
  display: flex;
  justify-content: space-between;
}

.modal-button {
  padding: 10px 20px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  color: #ffffff;
  outline: none;
}

.confirm-button {
  background-color: rgba(46, 139, 87, 0.85);
}

.reject-button {
  background-color: rgba(255, 99, 71, 0.85);
}
</style>
