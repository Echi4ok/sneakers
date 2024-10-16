<script setup>
 import axios from 'axios'
 import Card from '../components/Card.vue'
 import { ref, onMounted } from 'vue'

 const favorites = ref([]);
 onMounted(async () => {
    try {
        const {data} = await axios.get('https://14611ee5d238b870.mokky.dev/favorites')
        
        data.forEach((obj) => {
            favorites.value.push(obj.item); 
        });

    }  catch (err) {
        console.log(err)
    }
        

        
 });

console.log(favorites.value)
</script>


<template>
    <h1 v-if="favorites.length === 0">Мои закладки</h1>
    <!-- надо реализовать страницу что нет закладок -->
    <div v-else>
        <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>
            <div class="grid grid-cols-4 gap-5" v-auto-animate>
            <Card v-for="element in favorites"
                :key="element.id"
                :price="element.price"
                :title="element.title"
                :image-url="element.imageUrl"
                :isAdded="null"
                :isFavorite="null"
                />
            </div>
    </div>
</template>