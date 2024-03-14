<script>
import PlacesCard from './PlacesCard.vue';


export default {
    data() {
        return {
            filtered: false,
            search: '',
            results: '',
            filteredResults: { places: [] },
            postalCode: ''
        }
    },
    components: {
        PlacesCard
    },

    methods: {

        // This methods exports filteredResults to a csv file.
        exportToCsv() {
            const items = this.filteredResults.places;
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
        },

        /**
         * This method filters the places data based on the provided postal code.
         * 
         */
        filterPlaces() {
            this.filtered = true;
            const filteredData = this.results.places.filter(obj => obj.formattedAddress.includes(this.postalCode));
            this.filteredResults.places = filteredData;
            return filteredData;
        },

        searchPlaces() {
            this.filtered = false;
            const apiKey = 'AIzaSyAEAPVrCILuoMxjcn0V2sVS_qVgJ6LFsDQ';
            const url = 'https://places.googleapis.com/v1/places:searchText';
            const requestBody = {
                textQuery: this.search,
            };

            // if the search results are already in local storage, use them instead of making a new request.
            if (localStorage.getItem('searchResults') === this.results) {
                this.results = JSON.parse(localStorage.getItem('searchResults'));
                return;
            }
            fetch(url, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'X-Goog-Api-Key': apiKey,
                    'X-Goog-FieldMask': 'places.displayName,places.formattedAddress'
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
                    console.log('searching', data);
                    this.results = data;
                    localStorage.setItem('searchResults', JSON.stringify(data)); // Adding data to local storage
                    console.log(localStorage.getItem('searchResults'));
                })
                .catch(error => {
                    console.error('There was a problem with the fetch operation:', error);
                });
        },
        mounted() {
            // clear local storage on page load.
            localStorage.clear();
        }
    }
    };  
</script>

<template>
    <form class="pure-form" v-on:submit.prevent="onSubmit">
        <fieldset>
            <legend>Places Finder</legend>
            <input type="text" v-model="search" placeholder="Search for places" />
            <input type="text" v-model="postalCode" placeholder="Enter postal code" />
            <button type="submit" class="pure-button pure-button-primary" @click="searchPlaces()">Search</button>
            <button type="submit" class="pure-button pure-button-primary" @click="filterPlaces()">Filter</button>
        </fieldset>
    </form>
    <section>
        <h2>Results</h2>
        <div v-if="filtered">
            <PlacesCard v-for="place in filteredResults.places" :key="filteredResults.places.indexOf(place)"
                :displayName="place.displayName.text" :formattedAddress="place.formattedAddress" />
        </div>
        <div v-else>
            <PlacesCard v-for="place in results.places" :key="results.places.indexOf(place)"
                :displayName="place.displayName.text" :formattedAddress="place.formattedAddress" />
        </div>

    </section>
</template>

<style scoped>
.pure-button-primary {
    margin: 0 5px;
}
</style>
