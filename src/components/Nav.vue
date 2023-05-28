<script setup lang="ts">
import { useToggle } from '@vueuse/core'
import { useDarkGlobal } from '../utils'

const isDark = useDarkGlobal()
const toggleDark = useToggle(isDark)
</script>

<template>
  <nav class="w-[72px] h-screen flex flex-col border-r-[#393939] border-r border-solid">
    <div class="px-3 py-2 flex justify-between items-center border-b-[#393939] border-b border-solid">
      <a class="text-2xl font-bold text-gray-900 dark:text-white lg:text-3xl hover:text-gray-700 dark:hover:text-gray-300"
        href="#">J.P.</a>
    </div>
    <div class="flex-1"></div>
    <a class="border-t-[#393939] border-t border-solid justify-center flex py-5 relative text-gray-700 dark:text-gray-200 hover:text-gray-600 dark:hover:text-gray-300 hidden"
      href="#" @click.prevent="toggleDark()">X</a>

  </nav>
</template>

<style>
nav {
  height: var(--nav-height);
}
</style>
