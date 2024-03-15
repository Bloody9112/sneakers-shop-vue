<script setup>
import { reactive, watch, ref, onMounted } from 'vue'
import axios from 'axios'
import debounce from 'lodash.debounce'
import { inject } from 'vue'
import CardList from '../components/CardList.vue' // дві точки тому що вихзодим з pages і ідемо в components

const { cart, addToCart, removeFromCart } = inject('cart')

// state який хрнанить у собі усі наші кросівки. Ref для того щоб тримати у собі масив
const items = ref([])

// Ще 1 реактивний state який тримає у собі усі фільтри. Для обєктів Vue рекомендує використовувати reactive
const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

// функ. яка слідкує за змінами html селекту, вшивати його значення і викликати watch при зміні sortBy
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

// функ. яка слідкує за наших фільтрів
const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 250)

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      // перевірка чи був товар доданий, якщо ні создаєм його і прикручуєм ID
      const obj = {
        parentId: item.id,
        item
      }
      item.isFavorite = true
      const { data } = await axios.post(`https://606263a170d60566.mokky.dev/favorites`, obj)
      item.favoriteId = data.id
    } else {
      item.isFavorite = false
      await axios.delete(`https://606263a170d60566.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

// функц. яка запрашує закладки
const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://606263a170d60566.mokky.dev/favorites')

    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.parentId === item.id) // в масиві favorites найди обєкт кожного товара favorites який відноситься до items.

      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (err) {
    console.log(err)
  }
}

// функц. яка при зміні фільтра або при першому рендері відправляє запрос на бекенд
const fetchItems = async () => {
  // функц. яка запрашує список товарів.
  try {
    const params = {
      // За замовчуванням як тільки відбувається перший запрос на бекенд, завжди передається сортування
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      // При цьому перевіряється чи змінився параметр searchQuery.
      params.title = `*${filters.searchQuery}*` // Якщо було щось введено в інпут, змінюються параметри.
    }

    const { data } = await axios.get('https://606263a170d60566.mokky.dev/items', {
      //запит на бекенд
      params
    })
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    })) // як тільпи приходить відповідь з беку, відразу вшиваю його у title (const items = ref([]))
  } catch (err) {
    console.log(err)
  }
}

onMounted(async () => {
  //
  const localCart = localStorage.getItem('cart') // Читаєм данні з localStorage
  cart.value = localCart ? JSON.parse(localCart) : [] // Якщо немає, то створюєм пустий масив

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

watch(filters, fetchItems) // Слідкує за змінами sortBy, робитт запрос та оновлювати данні у value, а далі повертає результат.
</script>

<template>
  <div class="flex flex-col md:flex-row justify-between items-center">
    <h2 class="text-lg md:text-3xl font-bold mb-4 md:mb-0">Усі кросівки</h2>

    <div class="flex flex-col md:flex-row gap-4">
      <select
        @change="onChangeSelect"
        class="py-2 px-3 md:w-auto border rounded-md outline-none text-sm md:text-base"
      >
        <option value="name">Новинки</option>
        <option value="price">Ціна за зростанням</option>
        <option value="-price">Ціна за спаданням</option>
      </select>

      <div class="relative">
        <img class="absolute left-4 top-3" src="/search.svg" alt="Search" />
        <input
          @input="onChangeSearchInput"
          class="w-full md:w-auto border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400 placeholder:text-sm md:placeholder:text-base"
          type="text"
          placeholder="Пошук..."
        />
      </div>
    </div>
  </div>

  <div class="mt-6 md:mt-10">
    <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </div>
</template>
