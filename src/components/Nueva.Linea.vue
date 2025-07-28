<script setup lang="ts">
import { ref, defineEmits } from 'vue'

const emit = defineEmits<{(e: 'submit', payload: { name: string;}): void,(e: 'close'): void}>()
const newName = ref('')

const submit = () => {
  if (newName.value ) {
    emit('submit', { name: newName.value })
    clearForm()
  }
}

const close = () => {
  emit('close')
  clearForm()
}

const clearForm = () => {
  newName.value = ''
}
</script>
<template>
  <div class="overlay">
    <div class="overlay-content">
      <h3>Agregar Nueva Línea</h3>
      <input v-model="newName" type="text" placeholder="Nombre" />
      <button @click="submit">Agregar</button>
      <button class="cancel" @click="close">Cancelar</button>
    </div>
  </div>
</template>
<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(20, 20, 20, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  
}

.overlay-content {
  background-color: #242424;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  min-width: 320px;
  max-width: 90vw;
  min-height: 30vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: end;
}
button{
  width: 20%;
}
h3{
  width: 100%;
  text-align: center;
}
.overlay-content input {
  font-size: 16px;
  border-radius: 6px;
  border: 1px solid #ccc;
  width: 100%;
  height: min-content;
  padding: 2vh 10px;
}

.overlay-content button {
  padding: 10px;
  font-size: 16px;
  border-radius: 6px;
  border: none;
  cursor: pointer;
  transition: background-color 0.3s ease;
  background-color: #25d366;
}

.overlay-content button:hover {
  background-color: #ddd;
}

.overlay-content .cancel {
  background-color: #25d366;
  color: #333;
}
</style>
