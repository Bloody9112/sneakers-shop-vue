<script setup>
defineProps({
  totalPrice: Number
})

const emit = defineEmits(['openDrawer']) //Відкриваємо корзину
</script>

<template>
  <header
    class="flex flex-col md:flex-row justify-between border-b border-slate-200 px-4 md:px-8 py-4 gap-4"
  >
    <router-link to="/" class="flex items-center gap-4">
      <img src="/logo.png" alt="Logo" class="w-8 md:w-10" />
      <div>
        <h2 class="text-lg md:text-xl font-bold uppercase">Vue Sneakers</h2>
        <p class="text-slate-400 text-sm md:text-base">Магазин кращих кросівок</p>
      </div>
    </router-link>

    <ul class="flex justify-center md:justify-end items-center gap-3 md:gap-7">
      <li
        @click="() => emit('openDrawer')"
        class="flex items-center cursor-pointer text-gray-500 hover:text-black transition gap-1.5 md:gap-3"
      >
        <img src="/cart.svg" alt="Cart" class="w-4 md:w-5" />
        <b class="text-xs md:text-lg">{{ totalPrice }} грн.</b>
      </li>

      <router-link to="/favorites">
        <li
          class="flex items-center cursor-pointer text-gray-500 hover:text-black transition gap-1.5 md:gap-3"
        >
          <img src="/heart.svg" alt="Cart" class="w-4 md:w-5" />
          <span class="text-xs md:text-lg">Закладки</span>
        </li>
      </router-link>

      <li
        class="flex items-center cursor-pointer text-gray-500 hover:text-black transition gap-1.5 md:gap-3"
      >
        <img src="/profile.svg" alt="Cart" class="w-4 md:w-5" />
        <span class="text-xs md:text-lg">Профіль</span>
      </li>
    </ul>
  </header>
</template>
