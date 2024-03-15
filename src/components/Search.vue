<script setup>
import PlacesCard from './PlacesCard.vue';
import { Parser } from '@json2csv/plainjs';

import { ref, onMounted } from 'vue';

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
    const apiKey = 'AIzaSyAEAPVrCILuoMxjcn0V2sVS_qVgJ6LFsDQ';
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
            localStorage.setItem('searchResults', JSON.stringify(data));
        })
        .catch(error => {
            console.error('There was a problem with the fetch operation:', error);
        });

}


const testObject = [
    {
        "formattedAddress": "5048 Dundas St W, Etobicoke, ON M9A 1B9, Canada",
        "websiteUri": "https://thehvacservice.ca/etobicoke/",
        "displayName": {
            "text": "The HVAC Service",
            "languageCode": "en"
        }
    },
    {
        "formattedAddress": "3 Michael Power Pl #1105, Etobicoke, ON M9A 0A2, Canada",
        "displayName": {
            "text": "MMS HVAC",
            "languageCode": "en"
        }
    },
    {
        "formattedAddress": "70 Annie Craig Dr, Etobicoke, ON M8V 0G2, Canada",
        "websiteUri": "http://www.sepair.ca/",
        "displayName": {
            "text": "Sep Air HVAC Services",
            "languageCode": "en"
        }
    },
    {
        "formattedAddress": "80 Marine Parade Dr unit # 1505, Etobicoke, ON M8V 0A3, Canada",
        "websiteUri": "http://jskheating.com/",
        "displayName": {
            "text": "J.S.K. Heating & Cooling LTD",
            "languageCode": "en"
        }
    },
    {
        "formattedAddress": "59 Burlington St, Toronto, ON M8V 2L3, Canada",
        "websiteUri": "https://armanch.ca/",
        "displayName": {
            "text": "Armanch Inc",
            "languageCode": "en"
        }
    }
]
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
    <form class="pure-form" v-on:submit.prevent="searchPlaces">
        <fieldset>
            <legend>Places Finder</legend>
            <input type="text" v-model="search" placeholder="Search for places" />
            <input type="text" v-model="postalCode" placeholder="Enter postal code" />
            <button type="submit" class="pure-button pure-button-primary">Search</button>
            <button class="pure-button pure-button-primary"
                @click="filterPlaces(), displayFiltered = !displayFiltered">Filter
            </button>
            <button v-if="displayFiltered" class="pure-button button-success" @click="dataCsvExport" >Export to CSV</button>
        </fieldset>
    </form>
    <section>
        <h2>Results</h2>
        <p v-if="results.length  <= 0" >
            No results found, atm the moment.
        </p>
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
