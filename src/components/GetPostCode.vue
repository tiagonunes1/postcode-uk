<template>

    <div class="flex flex-col min-h-screen">

        <div class="card text-center m-3">
            <img class="mx-auto  mb-6" src="../assets/map-pin.svg" alt="github">

            <div class="flex justify-center">
                <div class="mb-3 xl:w-96">
                    <div class="form-group input-group relative flex flex-wrap items-stretch w-full mb-4">
                        <div class="flex absolute inset-y-0 left-0 items-center pl-3 pointer-events-none">
                            <svg class="w-5 h-5 text-gray-500 dark:text-gray-400" fill="none" stroke="currentColor"
                                viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                    d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
                            </svg>
                        </div>
                        <input @keyup.enter ="created" type="search" v-model="code"
                            class="block p-4 pl-10 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                            placeholder="Search" aria-label="Search" aria-describedby="button-addon2">
                        <button  @click="created"
                            class="text-white absolute right-2.5 bottom-2.5 bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-4 py-2 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
                            type="button" id="button-addon2">Search
                        </button>
                    </div>
                </div>
            </div>
            <h1 v-show="elementVisible" v-if="responseStatus === 404" class="flex justify-center mb-5">
                <div class="bg-orange-100 border-l-4 border-orange-500 text-orange-700 p-4" role="alert">
                    <p class="font-bold">Error!</p>
                    <p>No data was found for postcode: <strong>{{ code }}</strong></p>
                </div>
            </h1>

            <div v-if="postCode">
                <ul>
                    <li class="card-body"><strong>{{ nhs_ha }}</strong></li>
                    <li class="card-body"><strong>{{ postCode }}</strong></li>
                    <li class="card-body"><strong>{{ ccg }}</strong></li>
                    <li class="card-body">Latitude: <strong>{{ latitude }}</strong></li>
                    <li class="card-body">Longitude: <strong>{{ longitude }}</strong></li>
                    <li class="card-body">London Heathrow airport distance/Km: <strong>{{ airportDistanceKm
                    }}</strong></li>
                    <li class="card-body mb-10">London Heathrow airport distance/Mi: <strong>{{ airportDistanceMi
                    }}</strong></li>
                </ul>
            </div>


            <div v-if="postCodeCards.length > 0">
                <h2 class="font-bold mb-7">Search History</h2>
                <div class="grid grid-cols-3 gap-2 text-left flex justify-center items-center ml-10">
                    <div v-for="card in postCodeCards" :key="card.id"
                        class="mt-auto max-w-sm bg-white rounded-lg border border-gray-200 shadow-md dark:bg-gray-800 dark:border-gray-700 mb-6">
                        <a href="#">
                            <!-- <img class="rounded-t-lg"
                                src="https://i.pinimg.com/originals/83/26/96/832696c3ff680f405ddc47fead99fc5c.jpg"
                                alt="" /> -->
                        </a>
                        <div class="p-5">
                            <div class="card-body"><strong>{{ card.nhs_ha }}</strong></div>
                            <div class="card-body"><strong>{{ card.postCode }}</strong></div>
                            <div class="card-body"><strong>{{ card.ccg }}</strong></div>
                            <div class="card-body">Latitude: <strong>{{ card.lat }}</strong></div>
                            <div class="card-body">Longitude: <strong>{{ card.long }}</strong></div>
                            <div class="card-body">London Heathrow airport distance/Km: <strong>{{
                                    card.airportDistanceKm
                            }}</strong>
                            </div>
                            <div class="mb-7 card-body">London Heathrow airport distance/Mi: <strong>{{
                                    card.airportDistanceMi
                            }}</strong>
                            </div>
                            
                            <button @click="navigateGoogleMaps(card.lat,card.long)" class="mt-auto bg-blue-700 hover:bg-blue-800 text-white font-bold py-2 px-4 rounded-lg">
                                Navigate
                            </button>
                        </div>
                    </div>
                </div>
            </div>

        </div>
        <footer class="mt-auto">

            <ul class="  text-sm text-gray-500 dark:text-gray-400">
                <li>
                    <a href="https://github.com/tiagonunes1" class="mr-4 hover:underline md:mr-6 "><img class="mx-auto"
                            src="../assets/github.svg" alt="github"></a>
                </li>
            </ul>
        </footer>
    </div>
</template>

<script>
export default {
    name: "get-request-async-await",
    data() {
        return {
            latitude: '',
            longitude: '',
            airportDistanceKm: 0,
            airportDistanceMi: 0,
            postCode: '',
            responseStatus: 0,
            elementVisible: true,
            nhs_ha: '',
            ccg: '',
            postCodeCards: []

        };
    },
    methods: {

        async created() {
            // Checks if the error alert is visible
            if (!this.elementVisible) {
                this.elementVisible = true;
            }
            setTimeout(() => this.elementVisible = false, 4000) // Adds a 4 second timer for error alert

            // If the user doesn't input any postcode, we won't fetch data
            if (!this.code) {
                return;
            }

            // GET request using fetch with async/await
            const response = await fetch(`http://api.postcodes.io/postcodes/${this.code}`);
            this.responseStatus = response.status;
            if (response.status === 404) {
                return;
            }

            const data = await response.json(); // We get the data from the API 

            const longitude = data.result.longitude
            const nhs_ha = data.result.nhs_ha
            const ccg = data.result.ccg
            const latitude = data.result.latitude
            this.postCode = this.code
            this.nhs_ha = nhs_ha,
                this.ccg = ccg,
                this.latitude = latitude;
            this.longitude = longitude;

            // We attribute the distances both in miles and kilometers
            let factor = 0.621371
            this.airportDistanceMi = (airportDistance(51.4700223, -0.4542955, latitude, longitude) * factor).toFixed(2);
            this.airportDistanceKm = airportDistance(51.4700223, -0.4542955, latitude, longitude).toFixed(2);

            // Create an object array where we'll use to populate the search history cards 
            let searchCards = { postCode: this.postCode, nhs_ha: this.nhs_ha, ccg: this.ccg, long: this.longitude, lat: this.latitude, airportDistanceMi: this.airportDistanceMi, airportDistanceKm: this.airportDistanceKm };
            this.postCodeCards.push(searchCards); // We then push the data into the array

            //This function takes in latitude and longitude of two location and returns the distance between them (in km and mi)
            function airportDistance(lat1, lon1, lat2, lon2) {
                let R = 6371; // Radius of the earth in km
                let dLat = toRad(lat2 - lat1);
                let dLon = toRad(lon2 - lon1);
                let airportLat1 = toRad(lat1);
                let postCodeLat2 = toRad(lat2);

                let a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                    Math.sin(dLon / 2) * Math.sin(dLon / 2) * Math.cos(airportLat1) * Math.cos(postCodeLat2);
                let c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
                let d = R * c;
                return d;
            }
            // Converts numeric degrees to radians
            function toRad(Value) {
                return Value * Math.PI / 180;
            }
        },

        async navigateGoogleMaps(lat,long) {
            window.open(`https://maps.google.com/?q=${lat},${long}`,'_blank')
        }
    },

};
</script>