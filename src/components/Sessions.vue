<script setup lang="ts">
    import { defineEmits } from 'vue';
    const emit = defineEmits<{(id: string,NL:string, Did:string,Eid:string,initD:string): void}>()
    const {accounts,tittle} = defineProps<{accounts: any, tittle:string}>()
    const formatDate = (iso) => {
        const date = new Date(iso)
        return date.toLocaleString()
    }
    const handleClick = (id: string): void =>{
        emit('id',id)
    }
    const handleAdd = (): void =>{
        emit('NL','nueva Linea')
    }
    const handleDelete = (id:string): void =>{
      emit('Did',id)
    }
    const handleEdit = (id:string): void =>{
      emit('Eid',id)
    }
    const handleDif = (): void =>{
        emit('initD',accounts)
    }
</script>
<template>
    <div class="main">
        <h2>{{ tittle }}</h2>
        <button class="add-button" @click="handleAdd" v-if="tittle === 'Sessiones Activas'" title="Agregar nueva linea">＋</button>
        <button class="Difu-button" @click="handleDif" v-if="tittle === 'Sessiones Activas'" title="Empezar difusion">👌</button>
        <div class="session-flex">
            <div v-for="account in accounts" :key="account._id" class="session-card">
                <button class="delete-button" @click="handleDelete(account._id)">
                🗑️
                </button>
                <button v-if="account.status === 'active'" class="edit-button" @click="handleEdit(account._id)">
                ✏️
                </button>
            <div class="session-content" @click="handleClick(account._id)">
                <h3>{{ account.accountName }}<br/> ({{ account.sessionId }})</h3>
                <p><strong>📱 Número:</strong> {{ account.phoneNumber }}</p>
                <p><strong>🟢 Estado:</strong> {{ account.status }}</p>
                <p><strong>📅 Creado:</strong> {{ formatDate(account.createdAt) }}</p>
                <p><strong>🔄 Última conexión:</strong> {{ formatDate(account.lastConnection) }}</p>
                <p class="id"><strong>ID:</strong> {{ account._id }}</p>
            </div>
        </div>
    </div>
</div>

</template>
<style scoped>
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
    margin: 2.5vw;
    display: flex;
    flex-direction: row;
}

h2 {
    font-size: 24px;
    width: 80%;
    margin: 1vh 2vw;
}

h3{
    margin: 0;
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
    margin: 2vh 1vw;
    width: 5%;
    margin-left: 0;
}

button:hover {
    background-color: #128c4a;
    transform: scale(1.05);
}

.session-flex {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 16px;
    margin: 5vh 2vw;
    width: 100%;
}

.session-card {
    position: relative;
    background-color: #f5f5f5;
    padding: 16px;
    border-radius: 12px;
    border: 1px solid #ccc;
    box-shadow: 2px 4px 8px rgba(0,0,0,0.05);
    transition: all 0.2s ease;
    width: min-content; /* Cambiado de min-content a un ancho fijo para mejor consistencia */
    min-height: 220px;
}

.session-content {
    cursor: pointer;
    height: 100%;
}

.session-card:hover {
    box-shadow: 4px 8px 16px rgba(0, 0, 0, 0.1);
}

.session-card h3 {
    font-size: 18px;
    margin-bottom: 8px;
    color: #333;
}

.session-card p {
    margin: 4px 0;
    color: #555;
}

.session-card .id {
    font-size: 12px;
    color: #888;
    word-break: break-word;
}

.delete-button {
    position: absolute;
    right: 0;
    background-color: #ff44440d;
    color: white;
    border: none;
    border-radius: 10px;
    padding: 8px 12px;
    font-weight: 600;
    font-size: 0.9rem;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    gap: 5px;
    width: min-content;
    margin-top: 0;
}

.delete-button:hover {
    background-color: #cc000062;
    transform: scale(1.05);
}

.delete-button:active {
    transform: scale(0.98);
}

.edit-button{
    position: absolute;
    right: 3vw;
    background-color: #ff44440d;
    color: white;
    border: none;
    border-radius: 10px;
    padding: 8px 12px;
    font-weight: 600;
    font-size: 0.9rem;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    gap: 5px;
    width: min-content;
    margin-top: 0;
}

.edit-button:hover {
    background-color: #0014cc62;
    transform: scale(1.05);
}

.edit-button:active {
    transform: scale(0.98);
}
/* Asegurar que el botón de eliminar no afecte el hover de la card */
.session-card:hover .delete-button {
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}
</style>