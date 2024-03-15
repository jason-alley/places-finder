<script setup>
import PlacesCard from './PlacesCard.vue';
import { Parser } from '@json2csv/plainjs';

import { ref, onMounted } from 'vue';

const API_KEY = import.meta.env.VITE_API_KEY;

const search = ref('');
const postalCode = ref('');
const results = ref([]);
let filteredResults = ref([]);
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
const filterPlaces = () => {
    // Split the postal code string into an array of postal codes
    const newPostalCodes = postalCode.value.split(',');

    // Filter the places based on whether any postal code in the array is included in the place's formatted address
    const filteredData = results.value.places.filter(place => newPostalCodes.some(postalCode => place.formattedAddress.includes(postalCode)));

    // Update the filtered results with the filtered data
    filteredResults.value = filteredData;

    console.log(filteredResults.value);
    // Return the filtered data
    return filteredData, filteredResults.value;
};

/**
 * @description This function makes a request to the Google Places API to search for places based on the users input, then returns and array of places with the defined fields.
 */
const searchPlaces = () => {
    const apiKey = API_KEY;
    const url = 'https://places.googleapis.com/v1/places:searchText';
    const requestBody = {
        textQuery: search.value,
    };

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
            'X-Goog-FieldMask': 'places.displayName,places.formattedAddress,places.websiteUri,places.nationalPhoneNumber',
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
            localStorage.setItem('searchResults', JSON.stringify(data));
        })
        .catch(error => {
            console.error('There was a problem with the fetch operation:', error);
        });

}

/**
 * @description This function exports the filtered search results to a CSV file.
 */
 const dataCsvExport = () => {
    const options ={
        fields: [
            {
                label: 'Address',
                value: 'formattedAddress',
            },
            {
                label: 'Website',
                value: 'websiteUri',
            },
            {
                label: 'Name',
                value: 'displayName.text',
            },
            {
                label: 'Phone Number',
                value: 'nationalPhoneNumber',
            }
        ],
        header: true,
    }
    const parser = new Parser(options);
    const csv = parser.parse(filteredResults.value);
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'places.csv';
    a.click();
    window.URL.revokeObjectURL(url);
 }

</script>

<template>
    <form class="form" v-on:submit.prevent="searchPlaces">
        <fieldset class="">
            <input type="text" v-model="search" placeholder="Search for places" />
            <input type="text" v-model="postalCode" placeholder="Enter postal code" />
            <button type="submit" class="form-btn">Search</button>
            <button class="form-btn" @click="filterPlaces(), displayFiltered = !displayFiltered">Filter
            </button>
        </fieldset>
    </form>
    <section class="section">
        <button v-if="displayFiltered" class="csv-btn warning" @click="dataCsvExport" >Export to CSV</button>
        <h2>Results</h2>
        <p v-if="results.length  <= 0" >
            No results found at the moment.
        </p>
        <div v-if="displayFiltered">
            <PlacesCard v-for="place in filteredResults"
                :key="filteredResults.indexOf(place)"
                :displayName="place.displayName.text"
                :formattedAddress="place.formattedAddress"
                :phoneNumber="place.nationalPhoneNumber"
                :websiteUri="place.websiteUri" />
        </div>
        <div v-else>
            <PlacesCard v-for="place in results.places"
                :key="results.places.indexOf(place)"
                :displayName="place.displayName.text"
                :formattedAddress="place.formattedAddress"
                :phoneNumber="place.nationalPhoneNumber"
                :websiteUri="place.websiteUri" />
        </div>
    </section>
</template>

<style scoped>

.form {
    margin-top: 6rem;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 1rem;
    display: flex;
}

.form input {
    margin-right: 1rem;
    width: 250px;
}

.form-btn {
    width: 100px;
    margin: 0 1rem;
}

.csv-btn {
    width: 300px;
    margin: 0 auto;
}

.section {
    margin-top: 1rem;
    width: 720px;
    display: flex;
    flex-direction: column;
}
</style>
