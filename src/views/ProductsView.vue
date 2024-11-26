<template>
    <h1>Products</h1>
    <ul>
        <li v-for="produto in produtos" :key="produto.id">
            <span>Nome: {{ produto.title }}</span>
            <span>Valor: {{ produto.price }}</span>
            <span>Categoria: {{ produto.category }}</span>
            <img :src="produto.image" >
        </li>
    </ul>
    
</template>

<script setup>
import { ref, onMounted } from 'vue'

const produto = ref(null)

async function fetchData() {
    const res = await fetch('https://fakestoreapi.com/products')
    produto.value = await res.json()
    return produto.value
}

onMounted(() => {
    fetchData()
})
</script>

<style scoped>

</style>
