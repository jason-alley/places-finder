<script>
export default {
  name: 'PlacesCard',
  props: {
    displayName: {
      type: String,
      required: true
    },
    formattedAddress: {
      type: String,
      required: true
    }
  }
}
</script>

<template>
    <div class="card">
        <h2>{{ displayName }}</h2>
        <p>{{ formattedAddress }}</p>
    </div>
</template>

<style scoped>
.card {
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 20px;
    margin: 20px;
    width: 300px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

</style>