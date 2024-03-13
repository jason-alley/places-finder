<script>
import PlacesCard from './PlacesCard.vue';


export default {
    data() {
        return {
            filtered: false,
            search: '',
            results: '',
            filteredResults: {places:[]},
            postalCode: ''
        }
    },
    components: {
        PlacesCard
    },

    methods: {
        /**
         * This method filters the places data based on the provided postal code.
         * 
         */
        filterPlaces() {
            this.filtered = true;
            const filteredData = this.results.places.filter(obj => obj.formattedAddress.includes(this.postalCode));
            this.filteredResults.places = filteredData;
        },

        searchPlaces() {
            const apiKey = 'AIzaSyAEAPVrCILuoMxjcn0V2sVS_qVgJ6LFsDQ';
            const url = 'https://places.googleapis.com/v1/places:searchText';
            const requestBody = {
                textQuery: this.search,
            };

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
                })
                .catch(error => {
                    console.error('There was a problem with the fetch operation:', error);
                });

        }
    },
    mounted() {
        // this.searchPlaces();
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
            <PlacesCard v-for="place in filteredResults.places" :key="filteredResults.places.indexOf(place)" :displayName="place.displayName.text" :formattedAddress="place.formattedAddress" />
        </div>
        <div v-else>
            <PlacesCard v-for="place in results.places" :key="results.places.indexOf(place)" :displayName="place.displayName.text" :formattedAddress="place.formattedAddress" />
        </div>
        
    </section>
</template>

<style scoped>
.pure-button-primary {
   margin: 0 5px;
}
</style>
 
