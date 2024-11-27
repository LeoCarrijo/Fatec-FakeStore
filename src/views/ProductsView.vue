<template>
    <h1>Products</h1>
    <ul>
        <li class="prod__item" v-for="prod in produto" :key="prod.id" @click="expandedItem(prod)">
            <span>{{ prod.title }}</span>
            <div class="prod__detalhes" v-show="prod.expanded">
                <span>{{ prod.category }}</span>
                <span>{{ prod.description }}</span>
                <span>{{ prod.price }}</span>
            </div>
        </li>
    </ul>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const produto = ref(null)

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

onMounted(() => {
    fetchData()
})
</script>

<style scoped>
.prod__item {
    cursor: pointer;
    margin-bottom: 10px;
}

.prod__detalhes {
    margin-top: 5px;
    display: flex;
    flex-direction: column;
    gap: 5px;
    background-color: red;
}
</style>
