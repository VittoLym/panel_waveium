<script setup lang="ts">
    const {id, socket} = defineProps<{ id: object, socket: WebSocket }>()
    const emit = defineEmits<{(act:string): void}>()
    import { ref, onMounted } from 'vue'
    const mensaje = ref<string>('');
    const chats = ref<object>(null);
    const chats_length = ref<number>(0);
    const contacts = ref<object>(null);
    const contacts_length = ref<number>(0);
    onMounted(()=>{
        const controlDifusion = (action: string) => {
            socket.emit('control_difu', { action });
        };
        if (socket && socket.connected) {
            socket.emit('chats',{userId: id.sessionId,},(response) => {
                if (response.error) {
                    console.error('hubo un error',response.error)
                } else {
                    chats_length.value = response.length
                }
            })
            socket.emit('contacts',{userId: id.sessionId,},(response) => {
                if (response.error) {
                    console.error('hubo un error',response.error)
                } else {
                    contacts_length.value = response.length
                }
            })
        }
    })

    function verChats() {
        if(chats.value !== null){
            chats.value = null
            return
        }
        if (socket && socket.connected) {
            socket.emit('chats',{userId: id.sessionId,},(response) => {
                if (response.error) {
                    console.error('hubo un error',response.error)
                } else {
                    console.log(response)
                    chats.value = response
                    chats_length.value = response.length
                }
  })
        } else {
            console.warn('⚠️ El socket no está listo aún');
        }
    }

    function verContactos() {
        if(contacts.value !== null){
            contacts.value = null
            return
        }
        if (socket && socket.connected) {
            socket.emit('contacts',{userId: id.sessionId,},(response) => {
                if (response.error) {
                    console.error('hubo un error',response.error)
                } else {
                    contacts.value = response
                    contacts_length.value = response.length
                    console.log(contacts.value)
                }
            })
        } else {
            console.warn('⚠️ El socket no está listo aún');
        }
    }

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
        if (socket && socket.connected) {
            socket.emit('logout',{user: id,},(response) => {
                if (response.error) {
                    console.error('hubo un error',response.error)
                } else {
                    console.log(response)
                    emit('act','acti')
                }
            })
        } else {
            console.warn('⚠️ El socket no está listo aún');
        }
    }

    const getStatus = (lastMessage: object) => {
        if (!lastMessage) return ''; // Si no hay mensaje, no muestra estado
        return lastMessage.ack === 3 ? '✓ visto' : '✗ no visto';
    };
</script>
<template>
    <section class="options">
        <button @click="verChats">💬 Chats</button>
        <button @click="verContactos">👥 Contactos</button>
        <button @click="desloguear">🚪 Salir</button>
        <aside>
            <input type="text" v-model="mensaje" placeholder="Escribí un mensaje..." key.up="enviarMensaje" />
            <button @click="enviarMensaje">📤 Enviar</button>
        </aside>
        <div class="chats-container" v-if="chats !==null">
            <article v-for="c in chats">
                <p><strong>{{ c.name }}:</strong></p>
                <p>{{ c?.lastMessage?.body }}</p>
                <p>({{ getStatus(c.lastMessage) }})</p>
            </article>
        </div>
        <div class="chats-container" v-if="contacts !==null">
            <article v-for="c in contacts">
                <p><strong>{{ c.number }} - {{ c.pushname || c.name || 'No name'}} :</strong></p>
                <p>Es empresa: {{ c.isBusiness }}</p>
            </article>
        </div>
        <div class="chats_length">
            <p>Chats: <span>{{ chats_length }}</span></p>
        </div>
        <div class="chats_length">
            <p>Contacts: <span>{{ contacts_length }}</span></p>
        </div>
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
    height: min-content;
    justify-content: center;
    align-items: center;
    padding: 5vh 0;
}
aside{
    display:flex;
    justify-content: space-between;
    width: 70vw;
    margin: 2vh 6vw;
}
.chats-container{
    display: flex;
    flex-direction: column;
    width: 100%;
    align-items: center;
    min-height: min-content;
}
.chats_length{
    background-color: #1e1e2f;
    color: white;
    padding: 10px 16px;
    border-radius: 8px;
    width: fit-content;
    box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    font-family: sans-serif;
    margin: 0 2vw;
}
.chats_length span {
  color: #4ade80; /* verde */
  font-weight: bold;
}
article{
    width: 40%;
    display: flex;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.429);
    border-radius: 10px;
    margin: 2vh 0;
    padding: 0 2vw;
}
article p{
    padding: 10px 15px;
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