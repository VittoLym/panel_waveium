<script setup lang="ts">
const {qr} = defineProps<{qr:string}>()
const qrUrl = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(qr)}`

</script>
<template>
<  <div class="qr-overlay">
    <div class="card">
      <div class="card-body">
        <h3>Escaneá el QR para {{ sessionId }}</h3>
        <img :src="qrUrl" alt="QR Code" class="img-fluid" />
      </div>
    </div>
  </div>
</template>
<style scoped>
h3{
    margin: 0;
}
.qr-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5); /* Fondo oscuro translúcido */
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999; /* Asegura que esté encima del resto */
}

.card {
  background-color: #242424;
  padding: 24px 32px;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  text-align: center;
  max-width: 90vw;
  width: 360px;
  height: 50vh;
  animation: popup-zoom 0.3s ease;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.img-fluid {
  width: 200px;
  height: 200px;
  margin-top: 12px;
}

@keyframes popup-zoom {
  from {
    transform: scale(0.8);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}
</style>