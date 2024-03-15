<script setup>
import { defineProps } from 'vue'

const props = defineProps({
  displayName: {
    type: String,
    required: true
  },
  formattedAddress: {
    type: String,
    required: true
  },
  websiteUri: {
    type: String,
  },
  phoneNumber: {
    type: String,
  }
})
</script>

<template>
    <div class="card">
        <h2>{{ displayName }}</h2>
        <p>{{ formattedAddress }}</p>
        <p>{{ phoneNumber }}</p>
        <a :href="websiteUri" target="_blank" >{{ websiteUri }}</a>
    </div>
</template>

<style scoped>
.card {
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 20px;
    margin: 20px;
    width: 400px;
    display: flex;
    flex-direction: column;
    overflow-wrap: break-word;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

</style>