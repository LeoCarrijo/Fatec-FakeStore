<template>
    <div class="categories-container">
        <div class="header">
            <h1>Categories</h1>
        </div>

        <div class="filters">
            <select v-model="selectedCategory">
                <option value="">Todas as categorias</option>
                <option v-for="category in categories" :key="category" :value="category">
                    {{ category }}
                </option>
            </select>
        </div>

        <ul>
            <li class="prod__item" v-for="prod in filteredProducts" :key="prod.id">
                <div class="prod__header" @click="expandedItem(prod)">
                    <h2>{{ prod.title }}</h2>
                    <span>${{ prod.price }}</span>
                </div>
                
                <div v-show="prod.expanded" class="prod__content">
                    <p>{{ prod.description }}</p>
                </div>
            </li>
        </ul>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const products = ref(null)
const selectedCategory = ref('')

function expandedItem(item) {
    item.expanded = !item.expanded 
}

async function fetchData() {
    const res = await fetch('https://fakestoreapi.com/products')
    const data = await res.json()
    products.value = data.map(item => ({
        ...item,
        expanded: false
    }))
}

const categories = computed(() => {
    if (!products.value) return []
    return [...new Set(products.value.map(item => item.category))]
})

const filteredProducts = computed(() => {
    if (!products.value) return []
    
    if (selectedCategory.value) {
        return products.value.filter(prod => prod.category === selectedCategory.value)
    }
    
    return products.value
})

onMounted(() => {
    fetchData()
})
</script>

<style scoped>
.prod__item {
    cursor: pointer;
    margin-bottom: 10px;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
}

.prod__content {
    margin-top: 10px;
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

.prod__header {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: space-between;
}
</style>
