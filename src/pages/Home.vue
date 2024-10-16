<script setup>
import CardList from '../components/CardList.vue'
import axios from 'axios'
import debounce from 'lodash.debounce'
import { inject, reactive, watch, ref, onMounted } from 'vue'

const { cart, addToCart, removeFromCart } = inject('cart');

const items = ref([]);

const filters = reactive({
  sortBy: 'title',
  searchQuery: '',
})

// добавление товара в корзину
const onClickAddPlus = (item) => {
  if(!item.isAdded) {
    addToCart(item)
    item.isAdded = true; // если товар не добавлен то доавляем товар 
  } else {
    removeFromCart(item)
  }
}

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value;
}

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value;
}, 2000);

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const { data } = await axios.post('https://14611ee5d238b870.mokky.dev/favorites', {
        parentId: item.id,
        item,
      });
      item.isFavorite = true;
      item.favoriteId = data.id;
    } else {
      await axios.delete(`https://14611ee5d238b870.mokky.dev/favorites/${item.favoriteId}`);
      item.isFavorite = false;
      item.favoriteId = null;
    }
    await fetchFavorites(); // Обновляем список закладок после изменения
  } catch (err) {
    console.log(err);
  }
};

// запрос закладок
const fetchFavorites = async () => {
  try {
  const {data : favorites} = await axios.get(`https://14611ee5d238b870.mokky.dev/favorites`)
  
    
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.parentId === item.id);

      if(!favorite) {
        return item;
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      }
    })
  } catch (err)  {
    console.log(err);
  }
}


// запрос на бэк 
const fetchItems = () => {
  const params = {
    sortBy: filters.sortBy, 
    //тут мы не передаем инпут потому что он всегда пустой
  }
// если в serachQuery не пустой то добавляем в params значение инпута из filters
  if(filters.searchQuery) {
    //если filters.searchQuery не пустой, то мы добавляем новое свойство title в объект params
    params.title = `*${filters.searchQuery}*`
  }
  axios.get('https://14611ee5d238b870.mokky.dev/items', {
    params
  })
  .then((res) => {
    let response = res.data;
    items.value = response.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false,
    }));
  }).catch((err) => {
    console.log(err)
  })
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart');
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems();
  await fetchFavorites();

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id),
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false,
  }))
})

watch(filters, fetchItems)
</script>


<template>
    <div class="flex justify-between items-center">
          <h2 class="text-3xl font-bold Drawermb-8">Все кроссовки</h2>

          <div class="flex gap-4">
            <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
                <option value='name'>По названию</option>
                <option value='price'>По цене (дешевые)</option>
                <option value='-price'>По цене (дорогие)</option>
          </select>


          <div class="relative">
            <img class="absolute left-4 top-3" src="/search.svg" alt="Лупа"/>
            <input 
                @input="onChangeSearchInput"
                class="border rounded-md py-2 pl-12 pr-4 outline-none focus:border-gray-400" 
                placeholder="Поиск..."
                type="text"
            />
          </div>
        </div>
    </div>

        <div class="mt-10">
          <CardList :items="items" @addToFavorite="addToFavorite" @addToCart="onClickAddPlus"/>
        </div>
</template>