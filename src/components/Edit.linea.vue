<template>
  <div class="modal-overlay">
    <div class="modal-container">
      <div class="modal-header">
        <h2>Editar Sesión</h2>
        <button class="close-btn" @click="closeModal">✕</button>
      </div>

      <div class="modal-body">
        <label class="input-label">Nombre de la cuenta</label>
        <input type="text" v-model="editedAccountName" class="input-field" />

        <label class="input-label">Nuevo ID de sesión</label>
        <input type="text" v-model="editedSessionId" class="input-field" />
        <p class="note">Cambiar el ID puede requerir nuevo escaneo de QR</p>
      </div>

      <div class="modal-footer">
        <button class="btn cancel" @click="closeModal">Cancelar</button>
        <button class="btn save" @click="saveChanges">Guardar</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  session: {
    type: Object,
    required: true,
  }
})
const emit = defineEmits(['close', 'save'])

const editedAccountName = ref('')
const editedSessionId = ref('')

watch(
  () => props.session,
  (session) => {
    if (session) {
      editedAccountName.value = session.accountName
      editedSessionId.value = session.sessionId
    }
  },
  { immediate: true }
)

const closeModal = () => {
  emit('close')
}

const saveChanges = () => {
  emit('save', {
    accountName: editedAccountName.value,
    sessionId: editedSessionId.value,
    _id:props.session._id
  })
  closeModal()
}
</script>

<style scoped>
h2{
    color:#181818;
}
.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-container {
  background: white;
  border-radius: 12px;
  padding: 24px;
  width: 100%;
  max-width: 500px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  animation: fadeIn 0.25s ease;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: #999;
}

.modal-body {
  margin-top: 1rem;
}

.input-label {
  display: block;
  font-weight: bold;
  margin-top: 1rem;
}

.input-field {
  width: 100%;
  padding: 0.6rem;
  margin-top: 0.2rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 1rem;
}

.note {
  font-size: 0.8rem;
  color: #777;
  margin-top: 0.4rem;
}

.modal-footer {
  margin-top: 1.5rem;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.btn {
  padding: 0.6rem 1.2rem;
  font-weight: 500;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
}

.btn.cancel {
  background-color: #ddd;
  color: #333;
}

.btn.save {
  background-color: #4caf50;
  color: white;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.96);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
