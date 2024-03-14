<script setup>
import PlacesCard from './PlacesCard.vue';

import { ref, onMounted } from 'vue';

const search = ref('');
const postalCode = ref('');
const results = ref([]);
const filteredResults = ref([]);
let displayFiltered = ref(false);

onMounted(() => {
    // on mounted clear local storage.
    localStorage.clear();
});



/**
 * Filter places based on postal codes
 *
 * @returns {Array} - The filtered array of places
 */
const filterPlaces = async () => {
    // Split the postal code string into an array of postal codes
    const newPostalCodes = postalCode.value.split(',');

    // Filter the places based on whether any postal code in the array is included in the place's formatted address
    const filteredData = results.value.places.filter(place => newPostalCodes.some(postalCode => place.formattedAddress.includes(postalCode)));

    // Update the filtered results with the filtered data
    filteredResults.value = filteredData;

    // Return the filtered data
    return filteredData;
};

/**
 * @description This function makes a request to the Google Places API to search for places based on the users input, then returns and array of places with the defined fields.
 */
const searchPlaces = () => {
    const apiKey = 'AIzaSyAEAPVrCILuoMxjcn0V2sVS_qVgJ6LFsDQ';
    const url = 'https://places.googleapis.com/v1/places:searchText';
    const requestBody = {
        textQuery: search.value,
    };

    // If filtered results are displayed, clear them.
    // if (displayFiltered) {
    //     filteredResults.value = [];
    // }

    // if the search results are already in local storage, use them instead of making a new request.
    if (localStorage.getItem('searchResults') === results.value) {
        results.value = JSON.parse(localStorage.getItem('searchResults'));
        return;
    }
    fetch(url, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-Goog-Api-Key': apiKey,
            'X-Goog-FieldMask': 'places.displayName,places.formattedAddress,places.websiteUri,'
        },
        body: JSON.stringify(requestBody)
    })
        .then(response => {
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            return response.json();
        })
        .then(data => {
            results.value = data;
            console.log(results.value);
            localStorage.setItem('searchResults', JSON.stringify(data));
        })
        .catch(error => {
            console.error('There was a problem with the fetch operation:', error);
        });

}

/**
 * @description This function exports the filtered search results to a CSV file.
 */
const exportToCsv = () => {
    const items = filteredResults.value.places;
    const replacer = (key, value) => value === null ? '' : value;
    const header = Object.keys(items[0]);
    let csv = items.map(row => header.map(fieldName => JSON.stringify(row[fieldName], replacer)).join(','));
    csv.unshift(header.join(','));
    csv = csv.join('\r\n');
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.setAttribute('hidden', '');
    a.setAttribute('href', url);
    a.setAttribute('download', 'places.csv');
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
};

</script>

<template>
    <form class="pure-form" v-on:submit.prevent="searchPlaces">
        <fieldset>
            <legend>Places Finder</legend>
            <input type="text" v-model="search" placeholder="Search for places" />
            <input type="text" v-model="postalCode" placeholder="Enter postal code" />
            <button type="submit" class="pure-button pure-button-primary">Search</button>
            <button class="pure-button pure-button-primary"
                @click="filterPlaces(), displayFiltered = !displayFiltered">Filter</button>
        </fieldset>
    </form>
    <section>
        <h2>Results</h2>
        <div v-if="displayFiltered">
            <PlacesCard v-for="place in filteredResults" :key="filteredResults.indexOf(place)"
                :displayName="place.displayName.text" :formattedAddress="place.formattedAddress"
                :websiteUri="place.websiteUri" />
        </div>
        <div v-else>
            <PlacesCard v-for="place in results.places" :key="results.places.indexOf(place)"
                :displayName="place.displayName.text" :formattedAddress="place.formattedAddress"
                :websiteUri="place.websiteUri" />
        </div>

    </section>
</template>

<style scoped>
.pure-button-primary {
    margin: 0 5px;
}
</style>
