<template>
  <div class="users-container">
    <header>
      <h1>Users</h1>
      <button class="btn-add" @click="openModal()">Adicionar Usuário</button>
    </header>
    <div class="filters">
      <select v-model="sortOrder">
        <option value="desc">Mais Recentes</option>
        <option value="asc">Mais Antigos</option>
      </select>
      <select v-model="selectedUserId">
        <option value="">Todos os Usuários</option>
        <option v-for="userId in userIds" :key="userId" :value="userId">
          {{ userId }}
        </option>
      </select>
    </div>
    <ul>
      <li class="user__item" v-for="user in filteredAndSortedUsers" :key="user.id">
        <div class="user__header" @click="expandedItem(user)">
          <h2>{{ user.name.firstname }} {{ user.name.lastname }}</h2>
          <span>{{ user.email }}</span>
        </div>
        <div v-show="user.expanded" class="user__content">
          <h2>Detalhes</h2>
          <p>ID: {{ user.id }}</p>
          <p>Username: {{ user.username }}</p>
          <p>Telefone: {{ user.phone }}</p>
          <h3>Endereço:</h3>
          <ul>
            <li>Cidade: {{ user.address.city }}</li>
            <li>Rua: {{ user.address.street }}</li>
            <li>Número: {{ user.address.number }}</li>
            <li>Zipcode: {{ user.address.zipcode }}</li>
            <h4>Geolocalização</h4>
            <ul>
              <li>Lagitude: {{ user.address.geolocation.lat }}</li>
              <li>Longitude: {{ user.address.geolocation.long }}</li>
            </ul>
          </ul>

          <div class="user__actions">
            <button @click.stop="openModal(user)" class="btn-edit">Editar</button>
            <button @click.stop="deleteUser(user.id)" class="btn-delete">Excluir</button>
          </div>
        </div>
      </li>
    </ul>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h2>{{ editingUser ? 'Editar Usuário' : 'Novo Usuário' }}</h2>
        <form @submit.prevent="saveUser">
          <div class="form-group">
            <label>Primeiro Nome:</label>
            <input v-model="userForm.name.firstname" required />
          </div>
          <div class="form-group">
            <label>Último Nome:</label>
            <input v-model="userForm.name.lastname" required />
          </div>
          <div class="form-group">
            <label>Email:</label>
            <input v-model="userForm.email" required />
          </div>
          <div class="form-group">
            <label>Nome de Usuário:</label>
            <input v-model="userForm.username" required />
          </div>
          <div class="form-group">
            <label>Telefone:</label>
            <input type="number" v-model="userForm.phone" required />
          </div>
          <div class="form-group">
            <label>Cidade:</label>
            <input v-model="userForm.address.city" required />
          </div>
          <div class="form-group">
            <label>Rua:</label>
            <input v-model="userForm.address.street" required />
          </div>
          <div class="form-group">
            <label>Número:</label>
            <input v-model="userForm.address.number" required />
          </div>
          <div class="form-group">
            <label>Zipcode:</label>
            <input v-model="userForm.address.zipcode" required />
          </div>
          <div class="form-group">
            <label>Latitude:</label>
            <input
              type="number"
              step="0.000001"
              v-model="userForm.address.geolocation.lat"
              required
            />
          </div>
          <div class="form-group">
            <label>Longitude:</label>
            <input
              type="number"
              step="0.000001"
              v-model="userForm.address.geolocation.long"
              required
            />
          </div>
          <div class="modal-actions">
            <button type="button" @click="showModal = false">Cancelar</button>
            <button type="submit">Salvar</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'

const users = ref([])
const selectedUserId = ref('')
const sortOrder = ref('desc')
const showModal = ref(false)
const editingUser = ref(null)
const userForm = ref({
  name: { firstname: '', lastname: '' },
  email: '',
  username: '',
  phone: '',
  address: {
    city: '',
    street: '',
    number: '',
    zipcode: '',
    geolocation: {
      lat: '',
      long: '',
    },
  },
})
const userIds = ref([])

async function fetchUsers() {
  const res = await fetch('https://fakestoreapi.com/users')
  const data = await res.json()
  users.value = data.map((user) => ({
    ...user,
    expanded: false,
  }))
  userIds.value = [...new Set(users.value.map((user) => user.id))]
  return users.value
}

onMounted(() => {
  fetchUsers()
})

const filteredAndSortedUsers = computed(() => {
  if (!users.value) return []
  let filteredUsers = users.value
  if (selectedUserId.value) {
    filteredUsers = filteredUsers.filter((user) => user.id === selectedUserId.value)
  }
  filteredUsers.sort((a, b) => {
    if (sortOrder.value === 'desc') {
      return new Date(b.date) - new Date(a.date)
    }
    return new Date(a.date) - new Date(b.date)
  })
  return filteredUsers
})

function expandedItem(item) {
  item.expanded = !item.expanded
}

function openModal(user = null) {
  editingUser.value = user
  if (user) {
    userForm.value = {
      name: { ...user.name },
      email: user.email,
      username: user.username,
      phone: user.phone,
      address: {
        city: user.address.city,
        street: user.address.street,
        number: user.address.number,
        zipcode: user.address.zipcode,
        geolocation: {
          lat: user.address.geolocation.lat,
          long: user.address.geolocation.long,
        },
      },
    }
  } else {
    userForm.value = {
      name: { firstname: '', lastname: '' },
      email: '',
      username: '',
      phone: '',
      address: {
        city: '',
        street: '',
        number: '',
        zipcode: '',
        geolocation: {
          lat: '',
          long: '',
        },
      },
    }
  }
  showModal.value = true
}

async function saveUser() {
  try {
    if (editingUser.value) {
      const index = users.value.findIndex((u) => u.id === editingUser.value.id)
      users.value[index] = {
        ...userForm.value,
        id: editingUser.value.id,
        expanded: false,
      }
    } else {
      const newUser = {
        ...userForm.value,
        id: Math.max(...users.value.map((u) => u.id)) + 1,
        expanded: false,
      }
      users.value.push(newUser)
    }
    showModal.value = false
  } catch (error) {
    console.error('Erro ao salvar usuário:', error)
    alert('Erro ao salvar usuário')
  }
}

function deleteUser(id) {
  if (!confirm('Tem certeza que deseja excluir este usuário?')) return
  try {
    users.value = users.value.filter((u) => u.id !== id)
  } catch (error) {
    console.error('Erro ao excluir usuário:', error)
    alert('Erro ao excluir usuário')
  }
}
</script>

<style scoped>
.user__item {
  cursor: pointer;
  margin-bottom: 10px;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
.user__content {
  margin-top: 5px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}
.filters {
  margin: 20px 0;
  display: flex;
  gap: 10px;
}
select {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}
.user__actions {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}
.btn-add,
.btn-edit,
.btn-delete {
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
.btn-add {
  background-color: #4caf50;
  color: white;
  border: none;
}
.btn-edit {
  background-color: #2196f3;
  color: white;
  border: none;
}
.btn-delete {
  background-color: #f44336;
  color: white;
  border: none;
}
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-content {
  background-color: grey;
  padding: 20px;
  border-radius: 8px;
  width: 500px;
  max-width: 90%;
}
.form-group {
  margin-bottom: 15px;
}
.form-group label {
  display: block;
  margin-bottom: 5px;
}
.form-group input,
.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}
</style>
