<template>
  <div class="carts-container">
    <header>
      <h1>Carts</h1>
      <button class="btn-add" @click="openModal()">Adicionar Carrinho</button>
    </header>
  </div>
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
    <li class="cart__item" v-for="cart in filteredAndSortedCarts" :key="cart.id">
      <div class="cart__header" @click="expandedItem(cart)">
        <h2>{{ formatDate(cart.date) }}</h2>
        <span>{{ cart.userId }}</span>
      </div>
      <div v-show="cart.expanded" class="cart__content">
        <h2>Produtos</h2>
        <div v-for="produto in cart.products" :key="produto.id">
          <p>ID: {{ produto.productId }}</p>
          <p>Quantidade: {{ produto.quantity }}</p>
        </div>
        <div class="cart__actions">
          <button @click.stop="openModal(cart)" class="btn-edit">Editar</button>
          <button @click.stop="deleteCart(cart.id)" class="btn-delete">Excluir</button>
        </div>
      </div>
    </li>
  </ul>
  <div v-if="showModal" class="modal">
    <div class="modal-content">
      <h2>{{ editingCart ? 'Editar Carrinho' : 'Novo Carrinho' }}</h2>
      <form @submit.prevent="saveCart">
        <div class="form-group">
          <label>Data:</label>
          <input type="date" v-model="cartForm.date" required />
        </div>
        <div class="form-group">
          <label>ID do Usuário:</label>
          <input v-model="cartForm.userId" required />
        </div>
        <div class="form-group">
          <label>Produtos:</label>
          <div v-for="(product, index) in cartForm.products" :key="index" class="product-item">
            <input v-model="product.productId" placeholder="ID do Produto" required />
            <input v-model="product.quantity" type="number" placeholder="Quantidade" required />
            <button type="button" @click="removeProduct(index)" class="btn-delete">Remover</button>
          </div>
          <button type="button" @click="addProduct" class="btn-add">Adicionar Produto</button>
        </div>
        <div class="modal-actions">
          <button type="button" @click="showModal = false">Cancelar</button>
          <button type="submit">Salvar</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'

const cart = ref(null)
const selectedUserId = ref('')
const sortOrder = ref('desc') // Adiciona a propriedade reativa para a ordem de classificação
const showModal = ref(false)
const editingCart = ref(null)
const cartForm = ref({
  date: '',
  userId: '',
  products: [],
})
const sortCriteria = ref('date') // Adiciona a propriedade reativa para o critério de ordenação
const userIds = ref([]) // Adiciona a propriedade reativa para os IDs de usuário

async function fetchCarts() {
  const res = await fetch('https://fakestoreapi.com/carts')
  const data = await res.json()
  cart.value = data.map((cart) => ({
    ...cart,
    expanded: false,
  }))
  // Preenche a lista de IDs de usuário
  userIds.value = [...new Set(cart.value.map((cart) => cart.userId))]
  return cart.value
}

onMounted(() => {
  fetchCarts()
})

const filteredAndSortedCarts = computed(() => {
  if (!cart.value) return []
  let filteredCarts = cart.value
  if (selectedUserId.value) {
    filteredCarts = filteredCarts.filter((cart) => cart.userId === selectedUserId.value)
  }
  if (sortCriteria.value === 'date') {
    filteredCarts.sort((a, b) => {
      if (sortOrder.value === 'desc') {
        return new Date(b.date) - new Date(a.date)
      }
      return new Date(a.date) - new Date(b.date)
    })
  } else if (sortCriteria.value === 'userId') {
    filteredCarts.sort((a, b) => a.userId.localeCompare(b.userId))
  }
  return filteredCarts
})

function expandedItem(item) {
  item.expanded = !item.expanded
}

function openModal(cart = null) {
  editingCart.value = cart
  if (cart) {
    cartForm.value = {
      date: cart.date,
      userId: cart.userId,
      products: [...cart.products],
    }
  } else {
    cartForm.value = {
      date: '',
      userId: '',
      products: [],
    }
  }
  showModal.value = true
}

function addProduct() {
  cartForm.value.products.push({
    productId: '',
    quantity: 1,
  })
}

function removeProduct(index) {
  cartForm.value.products.splice(index, 1)
}

async function saveCart() {
  try {
    if (editingCart.value) {
      const index = cart.value.findIndex((c) => c.id === editingCart.value.id)
      cart.value[index] = {
        ...cartForm.value,
        id: editingCart.value.id,
        expanded: false,
      }
    } else {
      const newCart = {
        ...cartForm.value,
        id: Math.max(...cart.value.map((c) => c.id)) + 1,
        expanded: false,
      }
      cart.value.push(newCart)
    }
    showModal.value = false
  } catch (error) {
    console.error('Erro ao salvar carrinho:', error)
    alert('Erro ao salvar carrinho')
  }
}

function deleteCart(id) {
  if (!confirm('Tem certeza que deseja excluir este carrinho?')) return
  try {
    cart.value = cart.value.filter((c) => c.id !== id)
  } catch (error) {
    console.error('Erro ao excluir carrinho:', error)
    alert('Erro ao excluir carrinho')
  }
}

function formatDate(dateString) {
  const date = new Date(dateString)
  return date.toLocaleDateString('pt-BR', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
  })
}
</script>

<style scoped>
.cart__item {
  cursor: pointer;
  margin-bottom: 10px;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
.cart__content {
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
.cart__actions {
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
.product-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.product-item input {
  flex: 1;
}

.product-item .btn-delete {
  padding: 4px 8px;
}
</style>
