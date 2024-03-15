<script setup>
import { defineProps } from 'vue';

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
    <h3>{{ displayName }}</h3>
    <ul>
      <li>{{ formattedAddress }}</li>
      <li>Phone: {{ phoneNumber }}</li>
      <li>website: 
        <a :href="websiteUri" target="_blank">{{ websiteUri }}</a>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.card {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 20px;
  margin: 20px;
  width: 600px;
  display: flex;
  align-items: center;
  flex-direction: column;
  overflow-wrap: break-word;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

li {
  list-style: none;
}
</style>