<script setup lang="ts">
import { onBeforeMount, onMounted, ref } from 'vue';
import { io } from 'socket.io-client';
import layoutLinea from './components/Layout.linea.vue';
import Sessions from './components/Sessions.vue';
import NuevaLinea from './components/Nueva.Linea.vue';
import session from '../../Wavium/models/Session';
import QRLinea from './components/QR.linea.vue';
import EditLinea from './components/Edit.linea.vue';
const numeros = ref<string[]>([]);
const client = ref<any[]>([]);
const accounts = ref<any[]>([]);
const activeAccounts = ref<any[]>([]);
const nuevaLinea = ref<boolean>(false)
const qr = ref<string>('')
const currentSession = ref<string>('')
let socket: WebSocket; 
const host = "http://192.168.1.103:5000"
let id = ref<string>(''); // ahora reactivo para re-renderizar

onBeforeMount(() => {
  socket = io(`http://${location.hostname}:5000`, {
    query: {
      userId: 'frontend-panel'
    },
    transports: ['websocket'] // Fuerza usar WebSocket
  });
  socket.on('connect',async() => {
    console.log('🟢 WebSocket conectado desde el frontend');
    const info = await loadActiveSessions()
    client.value = info
    const nuevosNumeros = info
      .map(i => i.phoneNumber)
      .filter(i => 
        typeof i === 'string' && 
        !numeros.value.includes(i)
      );
    
    if (nuevosNumeros.length > 0) {
      numeros.value = [...numeros.value, ...nuevosNumeros];
    }
  });
  socket.on('clients', (data) => {
    const info = loadActiveSessions()
    console.log(info)
    const nuevosNumeros = data.clients
      .map(client => client.numero)
      .filter(numero => 
        typeof numero === 'string' && 
        !numeros.value.includes(numero)
      );
    
    if (nuevosNumeros.length > 0) {
      numeros.value = [...numeros.value, ...nuevosNumeros];
      console.log('Números actualizados:', numeros.value);
    }
    client.value = data.clients;
  });
  socket.on('disconnect', () => {
    console.log('🔴 Desconectado de Socket.io');
  });
  accounts.value = loadHistory()
});

async function loadActiveSessions() {
  try {
    const response = await fetch(`${host}/api/sessions/active`);
    if (!response.ok) throw new Error('Error al cargar sesiones');
    const sessions = await response.json();/* 
    renderSessions(sessions, 'active-sessions'); */
    activeAccounts.value = sessions
    return sessions
  } catch (error) {
    console.error('Error:', error);
    /* document.getElementById('active-sessions').innerHTML = `
      <div class="alert alert-danger">Error cargando sesiones activas: ${error.message}</div>
    `; */
  }
}

async function loadHistory() {
  try {
    const response = await fetch(`${host}/api/sessions`);
    console.log(response)
    if (!response.ok) throw new Error('Error al cargar historial');
    const s = await response.json();
    accounts.value = s
  } catch (error) {
    console.error('Error:', error);
  }
}

function startSession({ name, id }: { name: string, id: string }) {
  const sessionId = `${Date.now()}-${Math.random().toString(36).substring(2, 8)}`;;
  const accountName = name.trim();
  if (!sessionId || !accountName) {
    console.warn('Ambos campos son obligatorios');
    return;
  }
  socket.emit('start_session', { sessionId, accountName }, (response) => {
    if (response.error) {
      console.error('hubo un error',response.error)
    } else {
      nuevaLinea.value = false
      qr.value = ''
    }
  });
  socket.on('qr', (data) => {
    nuevaLinea.value = false
    qr.value = data.qr
  });
  socket.on('auth',({session})=>{
    qr.value = ''
    loadActiveSessions()
    loadHistory()
  })
  socket.on('disconnected', (data) => {
    if(qr.value){
      qr.value= ''
    }
  });

}

async function editSession(session: object) {
  const originalSession = client.value.find(c => c._id ===  session._id);
  if (!originalSession) return
  const id = originalSession._id
  try {
      const response = await fetch(`${host}/api/sessions/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ session, originalSession })
      });
      const data = await response.json()
      if (!response.ok) throw new Error(await response.text());
      loadActiveSessions();
      loadHistory();
      startSession({name: data.accountName, id: data.sessionId})
    } catch (error) {
      alert('Error al actualizar: ' + error.message);
  }
}
async function deleteSession(sessionId: string) {
  try {
    const response = await fetch(`${host}/api/sessions/${sessionId}`, {
      method: 'DELETE'
    });
    
    const result = await response.json();
    
    if (!response.ok) {
      throw new Error(result.error || 'Error al eliminar la sesión');
    }
    
    // Mostrar mensaje de éxito
    alert(result.message || 'Sesión eliminada correctamente');
    
    // Recargar las listas
    loadActiveSessions();
    loadHistory();
    
  } catch (error) {
    console.error('Error:', error);
    alert(error.message);
  }
}
const analize_NL = (): void => {
  nuevaLinea.value = true
}
const handleModle = (e: string): void => {
  nuevaLinea.value = false
  currentSession.value = ''
}
const handleDelete = async(id: string): void => {
  console.log(id,'este es el id')
  await deleteSession(id)
}
const handleEdit = async(id:string): void =>{
  const variable = client.value.find(c => c._id === id);
    if (variable) {
      currentSession.value = variable
    }
}
const analize_click = (id_number:string): void  => {
  const variable = client.value.find(c => c._id === id_number);
    if (variable) {
      id.value = variable;
      console.log('📤 Seleccionado:', id.value);
    }
}
const handleDif = (e:object): void  => {
  if (socket && socket.connected) {
    socket.emit('difu',{
      accounts: e
    })
  }
  else {
    console.warn('⚠️ El socket no está listo aún');
  }
}
const handleactu = ()=>{
  loadActiveSessions()
  loadHistory()
  id.value = ''
}
</script>

<template>
  <div>
    <h1>Panel de Control</h1>
    <Sessions v-if="activeAccounts.length > 0" :accounts="activeAccounts" :tittle="'Sessiones Activas'" @id="analize_click" @NL="analize_NL"@Did="handleDelete" @Eid="handleEdit" @initD="handleDif"/>
    <div v-else class="main">
      <span></span>
      <article>
        <p class="text"> No hay líneas Detectadas aun.</p>
      </article>
      <button class="add-button" @click="analize_NL">＋</button>
    </div>
    <div v-if="id">
      <h3>Enviar mensaje desde {{ id.accountName }}</h3>
      <layoutLinea v-if="id" :id='id' :socket='socket' @act="handleactu"/>
    </div>
    <Sessions v-if="accounts.length > 0" :accounts="accounts" :tittle="'Sessiones Historicas'" @NL="analize_NL" @id="analize_click" @Did="handleDelete" @Eid="handleEdit"/>
    <p v-else class="text"> No hay líneas Detectadas aun.</p>
    <NuevaLinea v-if="nuevaLinea" @close="handleModle" @submit="startSession"/>
    <QRLinea v-if="qr" :qr="qr"/>
    <EditLinea v-if="currentSession" :session="currentSession" @save="editSession" @close="handleModle"/>
  </div>
</template>

<style scoped>

h1{
  font-size: 1.3rem;
}
.add-button {
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
    margin: 2vh 1vw;
    width: 5%;
}

.add-button:hover {
    background-color: #128c4a;
    transform: scale(1.05);
}
span{
  width: 2vw;
  height: 1vh;
}
.main {
    width: 90vw;
    background-color: #fefefe18;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.429);
    border-radius: 10px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    justify-content: space-between;
    gap: 12px;
    flex-wrap: wrap;
    white-space: nowrap;
    margin: 2.5vw 5vw;
    display: flex;
    flex-direction: row;
}
.container {
  width: 90vw;
  margin: 20px 5vw;
  background-color: #fefefe18;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.429);
  border-radius: 10px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
  overflow-x: auto; /* scroll horizontal si no entran */
  white-space: nowrap;
}

.numero {
  background-color: #25d366; /* verde WhatsApp */
  color: white;
  padding: 12px 18px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 1.1rem;
  cursor: pointer;
  user-select: none;
  transition: background-color 0.3s ease, transform 0.2s ease;
  white-space: normal;
  min-width: 120px;
  text-align: center;
}

.numero:hover {
  background-color: #128c4a; /* verde oscuro al pasar mouse */
  transform: translateY(-4px);
}
.input-container {
  display: flex;
  gap: 10px;
  margin-top: 15px;
  align-items: center;
  flex-wrap: wrap;
}

</style>
