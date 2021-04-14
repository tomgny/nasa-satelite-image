<template>
    <div class="container center-block text-center">
        <div class="row">
            <div class="col-xs-12 col-sm-offset-2 col-md-offset-3">
                <h1>NASA Earth image</h1>
                <hr />
            </div>
        </div>
        <div class="row">
            <div class="col-xs-12 col-sm-offset-2 col-md-offset-3">
                <v-select
                    v-model="selectedLocation"
                    class="style-chooser"
                    placeholder="Enter the city"
                    :options="locationList"
                    label="display_name"
                    @search="fetchLocation"
                />
                <button class="btn btn-primary my-2" @click="fetchImage">
                    Get image!
                </button>
            </div>
        </div>
        <app-loading v-if="isLoading" />
        <div
            class="row"
            :style="{
                visibility: showMaps && !isLoading ? 'visible' : 'hidden'
            }"
        >
            <div
                class="col-xs-12 col-sm-8 col-sm-offset-2 col-md-6 col-md-offset-3 mx-auto d-block"
            >
                <img
                    id="nasa"
                    :src="imageUrl"
                    alt="NASA Image"
                    class="img-fluid"
                    style="max-height: 500px"
                />
                <div id="mapid" class="img-fluid my-2" style="height: 300px" />
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import Loading from './Loading';

export default {
    name: 'app',
    components: {
        appLoading: Loading
    },
    data() {
        return {
            apiKey: 'fQfbALf3bePjznQrwTMlf3TRLpHoEFQCfBmG4s4U',
            mapboxToken:
                'pk.eyJ1IjoidG9tZ255IiwiYSI6ImNrbmg5MG11aDJ4M2sydm1xNTIzNDkxMmUifQ.LbrdI3kKs-_3jl-C7oyebQ',
            myMap: null,
            nasaString: 'https://api.nasa.gov/planetary/earth/imagery',
            nominatimString: `https://nominatim.openstreetmap.org/search`,
            lat: 0,
            lon: 0,
            imageUrl: null,
            locationList: [],
            selectedLocation: null,
            showMaps: false,
            isLoading: false
        };
    },
    watch: {
        selectedLocation(value) {
            if (value) {
                this.lat = value.lat;
                this.lon = value.lon;
            }
        }
    },
    methods: {
        async fetchImage() {
            try {
                if (!this.selectedLocation) {
                    return alert('Select the city!');
                }

                this.isLoading = true;
                const { data } = await axios.get(this.nasaString, {
                    responseType: 'arraybuffer',
                    params: {
                        lat: this.lat,
                        lon: this.lon,
                        dim: 0.1,
                        api_key: this.apiKey
                    }
                });

                if (this.myMap) {
                    this.myMap.remove();
                }

                this.myMap = L.map('mapid').setView([this.lat, this.lon], 13);
                L.tileLayer(
                    'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}',
                    {
                        attribution:
                            'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
                        maxZoom: 18,
                        id: 'mapbox/streets-v11',
                        tileSize: 512,
                        zoomOffset: -1,
                        accessToken: this.mapboxToken
                    }
                ).addTo(this.myMap);

                this.imageUrl = window.URL.createObjectURL(new Blob([data]));
                this.showMaps = true;
                this.isLoading = false;
            } catch (error) {
                console.log(error);
            }
        },
        fetchLocation(city) {
            if (this.timer) {
                clearTimeout(this.timer);
                this.timer = null;
            }

            this.timer = setTimeout(async () => {
                try {
                    this.isLoading = true;
                    const { data } = await axios.get(this.nominatimString, {
                        params: {
                            city,
                            format: 'json',
                            polygon_geocodejson: 1
                        }
                    });

                    this.locationList = data.filter(
                        (value, index, array) =>
                            array.findIndex(
                                t => t.display_name === value.display_name
                            ) === index
                    );
                    this.isLoading = false;
                } catch (error) {
                    console.log(error);
                }
            }, 500);
        }
    }
};
</script>

<style scoped>
.style-chooser .vs__search::placeholder,
.style-chooser .vs__dropdown-toggle,
.style-chooser .vs__dropdown-menu {
    background: #dfe5fb;
    border: none;
    color: #394066;
    text-transform: lowercase;
    font-variant: small-caps;
}

.style-chooser .vs__clear,
.style-chooser .vs__open-indicator {
    fill: #394066;
}
</style>
