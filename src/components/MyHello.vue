<script setup>
defineProps({
    title: String,
    price: Number,
    isAdded: Boolean,
})
</script>


<template>
    <div class="product">
        <h2>Товар: {{title}}</h2>
        <h3>Цена: {{ price }}</h3>
        <h3>Добавлен: {{ isAdded }}</h3>
    </div>
</template>

<style scoped>

.product {
    border: 1px solid blue;
    padding: 10px;
    margin-bottom: 10px;
}
</style>
