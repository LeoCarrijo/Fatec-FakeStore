<template>
  <div class="carts-container">
    <header>
      <h1>Carts</h1>
      <button class="btn-add" @click="openModal()">Adicionar Carrinho</button>
    </header>
  </div>

  <div class="filters">
    <select v-model="selectedDate">
      <option value="">Todas as Datas</option>
      <option v-for="date in dates" :key="date" :value="date">
        {{ date }}
      </option>
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
        <h2>{{ cart.date }}</h2>
        <span>{{ cart.userId }}</span>
      </div>

      <div v-show="cart.expanded" class="cart__content">
        <!-- TODO - Conteúdo do Carrinho -->
        <h2>Conteúdo do Carrinho</h2>
        <h2>Conteúdo do Carrinho</h2>
        <h2>Conteúdo do Carrinho</h2>
      </div>
    </li>
  </ul>

  <!-- TODO - Modal para editar carrinho -->
</template>

<script setup>
import { ref } from 'vue'

const cart = ref(null)

async function fetchCarts() {
  const res = await fetch('http://fakestoreapi.com/carts')
  const data = await res.json()
  cart.value = data.map((cart) => ({
    ...cart,
    expanded: false,
  }))
  return cart.value
}
</script>

<style scoped></style>
