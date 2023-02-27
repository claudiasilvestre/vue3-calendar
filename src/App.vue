<script setup>
import Calendar from './components/Calendar.vue'
import SidebarMenu from './components/SidebarMenu.vue'
</script>

<template>
  <header>
    <SidebarMenu />
  </header>

  <main>
    <div class="sidebarContent">
        <Calendar />
    </div>
  </main>
</template>
