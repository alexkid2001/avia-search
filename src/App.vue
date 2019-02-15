<template>
    <div id="app">
        <img alt="Vue logo" src="./assets/logo.png">
        <div class="search-panel">
            <iata-select
                  :requestHeader="requestHeader"
                  :label="labelDeparture"
                  @checkAirport="checkIataDeparture">

            </iata-select>
            <iata-select
                  :requestHeader="requestHeader"
                  :label="labelArrival"
                  @checkAirport="checkIataArrival">

            </iata-select>
            <div class="class-select">
            <template v-for="(fClass, index) in flightClasses">
                <label class="class-select__item">
                    <input type="radio" :value="fClass" v-model="selectedClass">
                    <span class="class-select__name" >{{fClass.name}}</span>
                </label>
            </template>
            </div>
            <datepicker v-model="date"></datepicker>
            <div class="button" @click="searchClick">Поиск</div>
        </div>
        <div class="search-params">
            <div class="search-params__title">Параметры поиска</div>
            <div class="search-params__item">Departure  - {{ iataDeparture }} </div>
            <div class="search-params__item">Arriaval - {{ iataArrival }}</div>
            <div class="search-params__item">Date - {{ searchDate }}</div>
            <div class="search-params__item">Flight Class - {{ selectedClass.name }} ( {{selectedClass.code}} )</div>
        </div>

        <offers-block :request_id="request_id" >

        </offers-block>

    </div>
</template>

<script>
import Offers from './components/Offers'
import IATASelect from './components/IATAselect'
import Datepicker from 'vuejs-datepicker'

const app_id = '8e71ac9f18';
const date = new Date();

export default {
    name: 'app',
    components: {
    // HelloWorld
        'iata-select': IATASelect,
        'offers-block': Offers,
        'datepicker': Datepicker
    },
    data () {
        return  {
            loginResult: [],
            api_key: '',
            request_id: '',
            requestHeader: [],
            iataDeparture: '',
            iataArrival:'',
            flightClass: '',
            flightClasses: [
                {name: 'Эконом', code: 'E'},
                {name: 'Бизнес', code: 'B'},
            ],
            selectedClass: [],
            labelDeparture: 'IATA departure',
            labelArrival: 'IATA arrival',
            date: date,
            searchQuery: {
                directions: [
                    {
                        departure_code: '',
                        arrival_code: '',
                        date: "2019-02-21",
                        time: "M"
                    }
                ],
                adult_qnt: 1,
                child_qnt: 0,
                infant_qnt: 0,
                passenger_category: "",
                class: "E",
                direct: true,
                flexible: true,
                max_price: '10000000',
                airlines: [],
                fare_types: []
            },
            urlSearch: 'https://api-stage.etm-system.com/api/air/search',
        }
    },
    created: function () {
        if(!localStorage.api_key) {
            this.$http.get('https://api-stage.etm-system.com/api/login/' + app_id)
                .then(function (response) {
                    localStorage.api_key = response.data.etm_auth_key;
                })
        }
        else {
            this.api_key = localStorage.api_key;
        }
        this.requestHeader = { headers: { "etm-auth-key": this.api_key }};

        console.log(this.api_key);
        console.log(this.requestHeader);
    },
    methods: {
        transmitRequestID(id) {
            this.request_id = id;
            console.log('Родитель получил request_id - ' + this.request_id);
        },
        checkIataDeparture(iata) {
            this.iataDeparture = iata;
        },
        checkIataArrival(iata) {
            this.iataArrival = iata;
        },
        searchClick () {
            console.log('Search request');
            console.log(this.requestHeader);
            console.log('Departure - ' + this.iataDeparture);
            console.log('Arrival - ' + this.iataArrival);
            console.log('Date - ' + this.searchDate);
            this.searchQuery.directions[0].departure_code = this.iataDeparture;
            this.searchQuery.directions[0].arrival_code = this.iataArrival;
            this.searchQuery.directions[0].date = this.searchDate;
            this.searchQuery.class = this.selectedClass.code;
            console.log(this.searchQuery);
            this.$http.post(this.urlSearch, this.searchQuery, this.requestHeader)
                .then(response => {
                    console.log('status - ' + response.body.status);
                    console.log('request_id - ' + response.body.request_id);
                    this.request_id = response.body.request_id;
                    // this.$emit('updateRequestID', this.request_id);
                })
                .catch(err => {
                    console.log(err);
                });
        },
    },
    computed: {
        searchDate () {
            return this.date.getFullYear() + '-' + ( '0' + (this.date.getMonth() + 1)).slice(-2) + '-' + ('0' + this.date.getDate()).slice(-2);
        }
    }

}
</script>

<style lang="scss">
    @import "assets/style";
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
