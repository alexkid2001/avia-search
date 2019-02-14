<template>
    <div class="iata-select">
        <label class="etm-input etm-input_airport-name iata-select">
            <span class="etm-input__title">{{label}}</span>
            <span class="etm-input__wrap">
                <input type="text"  v-model="airportName" @input="onChange" placeholder="Название аэропорта">
                <span class="etm-input__iata">{{ airportIata }}</span>
            </span>
            <span class="drop-down" v-show="showDropDown">
                <ul class="airlines-list">
                    <li class="airlines-list__item"
                        v-bind:key="index"
                        v-bind:name = "airport.name"
                        v-bind:iata = "airport.code"
                        @click = "checkIATA($event, airport.name, airport.code)"
                        v-for="(airport, index) in sortIATA()">
                        <span class="airlines-list__name">{{airport.name}}</span>
                        <span class="airlines-list__iata">{{airport.code}}</span>
                    </li>

                </ul>
            </span>
        </label>
    </div>
</template>

<script>
    const urlAirport = 'https://api-stage.etm-system.com/api/dictionaries/airports?filter=';
    export default {
        name: "iata-select",
        props: [
            'requestHeader',
            'label'
        ],
        data() {
            return {
                airports:[],
                airportName: '',
                airportIata: '',
                showDropDown: false
            }
        },
        methods: {
            checkIATA(e, name, iata) {
                this.airportName = name;
                this.airportIata = iata;
                this.showDropDown = false;
                this.$emit('checkAirport', this.airportIata);
            },
            sortIATA() {
                return this.airports.filter(airportItem => airportItem.name.toLowerCase().indexOf(this.airportName.toLowerCase()) !== -1);
            },
            onChange () {
                if((this.airportName.length === 4) && !this.isUpdate) {
                    this.isUpdate = true;
                    this.$http.get(urlAirport + this.airportName, this.requestHeader)
                        .then(response => {
                            console.log(response.data.data);
                            this.airports = response.data.data;
                            // console.log(this.airlines)
                            this.showDropDown = true;
                        })
                        .catch(err => {
                            console.log(err);
                            if(err.status === 403) {
                                localStorage.removeItem('api_key');
                                alert('Перегрузите страница');
                            }
                            this.showDropDown = false;
                        })
                    return this.airports;
                }
                else {
                    if(this.airportName.length < 3) {
                        this.isUpdate = false;
                    }
                }
            }
        }
    }
</script>

<style scoped>

</style>