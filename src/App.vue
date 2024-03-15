<template>
  <Navbar />
  <div class="wrapper">
    <Search />
  </div>
</template>

<script setup>
import Navbar from './components/Navbar.vue'
import Search from './components/Search.vue'
</script>

<style scoped> 
.wrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>