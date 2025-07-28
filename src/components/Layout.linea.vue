<script setup lang="ts">
    const {id, socket} = defineProps<{ id: object, socket: WebSocket }>()
    import { ref } from 'vue'
    const mensaje = ref<string>('');

    function agregarLinea() {
    console.log('Agregar nueva línea')
    }

    function verChats() {
    console.log('Ver chats')
    }

    function verContactos() {
    console.log('Ver contactos')
    }

    function subirHistoria() {
    console.log('Subir historia')
    }
    
    // Enviar mensaje por WebSocket
    const enviarMensaje = () => {
        if (!id || !mensaje.value.trim()) return;

        console.log(`📤 Enviando mensaje "${mensaje.value}" desde ${id.accountName}`);
        if (socket && socket.connected) {
            socket.emit('mensaje',{
            userId: id.sessionId,
            mensaje: mensaje.value.trim()
            })
        } else {
            console.warn('⚠️ El socket no está listo aún');
        }

        mensaje.value = ''; // limpia el input
    };
    function desloguear() {
    console.log('Cerrar sesión')
    }
</script>
<template>
    <section class="options">
        <!-- Agregar línea -->
        <button @click="agregarLinea">➕</button>

        <!-- Ver chats -->
        <button @click="verChats">💬 Chats</button>

        <!-- Ver contactos -->
        <button @click="verContactos">👥 Contactos</button>

        <!-- Subir historia -->
        <button @click="subirHistoria">📷 Historia</button>

        <!-- Input mensaje -->
        <div class="input-container">
        </div>
        <!-- Desloguear -->
        <button @click="desloguear">🚪 Salir</button>
        <aside>
            <input type="text" v-model="mensaje" placeholder="Escribí un mensaje..." key.up="enviarMensaje" />
            <button @click="enviarMensaje">📤 Enviar</button>
        </aside>
    </section>

</template>
<style scoped>
.options{
    margin: 0 5vw ;
    display: flex;
    background-color: #fefefe18;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.429);
    border-radius: 10px;
    width: 90vw;
    flex-wrap: wrap;
    height: 30vh;
    justify-content: center;
    align-items: center;
}
aside{
    display:flex;
    justify-content: space-between;
    width: 70vw;
}

input[type="text"] {
  flex: 1;
  padding: 5px 10px;
  border: 2px solid #25d366;
  border-radius: 10px;
  font-size: 1rem;
  outline: none;
  height: 40px;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

input[type="text"]:focus {
  border-color: #128c4a;
  box-shadow: 0 0 5px rgba(18, 140, 74, 0.5);
}

button {
  background-color: #25d366;
  color: white;
  padding: 5px 10px;
  font-weight: 600;
  font-size: 1rem;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  height: 50px;
  margin: 0 10px;
}

button:hover {
  background-color: #128c4a;
  transform: scale(1.05);
}

</style>