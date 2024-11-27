<template>
    <div class="products-container">
        <div class="header">
            <h1>Products</h1>
            <button class="btn-add" @click="openModal()">Adicionar Produto</button>
        </div>

        <div class="filters">
            <select v-model="selectedCategory">
                <option value="">Todas as categorias</option>
                <option v-for="category in categories" :key="category" :value="category">
                    {{ category }}
                </option>
            </select>
            
            <select v-model="sortBy">
                <option value="">Ordenar por</option>
                <option value="price-asc">Preço: Menor para Maior</option>
                <option value="price-desc">Preço: Maior para Menor</option>
            </select>
        </div>

        <ul>
            <li class="prod__item" v-for="prod in filteredAndSortedProducts" :key="prod.id">
                <div class="prod__header" @click="expandedItem(prod)">
                    <h2>{{ prod.title }}</h2>
                    <span>${{ prod.price }}</span>
                </div>
                
                <div v-show="prod.expanded" class="prod__content">
                    <p>{{ prod.description }}</p>
                    <div class="prod__actions">
                        <button @click.stop="openModal(prod)" class="btn-edit">Editar</button>
                        <button @click.stop="deleteProduct(prod.id)" class="btn-delete">Excluir</button>
                    </div>
                </div>
            </li>
        </ul>

        <div v-if="showModal" class="modal">
            <div class="modal-content">
                <h2>{{ editingProduct ? 'Editar Produto' : 'Novo Produto' }}</h2>
                <form @submit.prevent="saveProduct">
                    <div class="form-group">
                        <label>Título:</label>
                        <input v-model="productForm.title" required>
                    </div>
                    <div class="form-group">
                        <label>Descrição:</label>
                        <textarea v-model="productForm.description" required></textarea>
                    </div>
                    <div class="form-group">
                        <label>Preço:</label>
                        <input type="number" step="0.01" v-model.number="productForm.price" required>
                    </div>
                    <div class="form-group">
                        <label>Categoria:</label>
                        <input v-model="productForm.category" required>
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
import { ref, onMounted, computed } from 'vue'

const produto = ref(null)
const selectedCategory = ref('')
const sortBy = ref('')

function expandedItem(item) {
    item.expanded = !item.expanded 
}

async function fetchData() {
    const res = await fetch('https://fakestoreapi.com/products')
    const data = await res.json()
    produto.value = data.map(item => ({
        ...item,
        expanded: false
    }))
    return produto.value
}

const categories = computed(() => {
    if (!produto.value) return []
    return [...new Set(produto.value.map(item => item.category))]
})

const filteredAndSortedProducts = computed(() => {
    if (!produto.value) return []
    
    let filtered = produto.value

    if (selectedCategory.value) {
        filtered = filtered.filter(prod => prod.category === selectedCategory.value)
    }

    if (sortBy.value === 'price-asc') {
        filtered = [...filtered].sort((a, b) => a.price - b.price)
    } else if (sortBy.value === 'price-desc') {
        filtered = [...filtered].sort((a, b) => b.price - a.price)
    }

    return filtered
})

onMounted(() => {
    fetchData()
})

const showModal = ref(false)
const editingProduct = ref(null)
const productForm = ref({
    title: '',
    description: '',
    price: 0,
    category: ''
})

function openModal(product = null) {
    editingProduct.value = product
    if (product) {
        productForm.value = { ...product }
    } else {
        productForm.value = {
            title: '',
            description: '',
            price: 0,
            category: ''
        }
    }
    showModal.value = true
}

async function saveProduct() {
    try {
        if (editingProduct.value) {
            const index = produto.value.findIndex(p => p.id === editingProduct.value.id)
            produto.value[index] = { 
                ...productForm.value, 
                id: editingProduct.value.id,
                expanded: false 
            }
        } else {
            const newProduct = {
                ...productForm.value,
                id: Math.max(...produto.value.map(p => p.id)) + 1,
                expanded: false
            }
            produto.value.push(newProduct)
        }
        
        showModal.value = false
    } catch (error) {
        console.error('Erro ao salvar produto:', error)
        alert('Erro ao salvar produto')
    }
}

function deleteProduct(id) {
    if (!confirm('Tem certeza que deseja excluir este produto?')) return
    
    try {
        produto.value = produto.value.filter(p => p.id !== id)
    } catch (error) {
        console.error('Erro ao excluir produto:', error)
        alert('Erro ao excluir produto')
    }
}
</script>

<style scoped>
.prod__item {
    cursor: pointer;
    margin-bottom: 10px;
    padding: 8px;
    border-radius: 4px;
    border: 1px solid #ccc;
}

.prod__detalhes {
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

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}

.prod__actions {
    display: flex;
    gap: 10px;
    margin-top: 10px;
}

.btn-add, .btn-edit, .btn-delete {
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
}

.btn-add {
    background-color: #4CAF50;
    color: white;
    border: none;
}

.btn-edit {
    background-color: #2196F3;
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
