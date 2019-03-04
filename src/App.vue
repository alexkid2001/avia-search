<template>
    <div id="app">
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
            <div class="etm-input">
                <div class="etm-input__title">Дата вылета</div>
                <datepicker v-model="date"></datepicker>
            </div>
            <div class="button" @click="searchClick">Поиск</div>
        </div>
        <div class="search-params">
            <h3>Входные параметры запроса</h3>
            <div class="search-params__item">Departure  - <b>{{ iataDeparture }}</b> </div>
            <div class="search-params__item">Arriaval - <b>{{ iataArrival }}</b></div>
            <div class="search-params__item">Date - <b>{{ searchDate }}</b></div>
            <div class="search-params__item">Flight Class - <b>{{ selectedClass.name }} ( {{selectedClass.code}} ) </b></div>

            <h3>Результаты поиска</h3>
            <div class="search-params__item">Request ID - <b>{{ request_id }}</b></div>


        </div>


        <offers-block   :request_id="request_id"
                        :requestHeader="requestHeader" >

        </offers-block>

    </div>
</template>

<script>
import Offers from './components/Offers'
import IATASelect from './components/IATAselect'
import Datepicker from 'vuejs-datepicker'

const app_id = '8e71ac9f18';
const date = new Date();
const timeLimit = 1300000;

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
                // passenger_category: "YCD",
                class: "E",
                // direct: true,
                // flexible: true,
                max_price: '50',
                // airlines: [],
                fare_types: ["PUB", "NEG"]
            },
            urlSearch: 'https://api-stage.etm-system.com/api/air/search',
        }
    },
    created: function () {
        let now = new Date;
        if((!localStorage.api_key) || ((now.getTime() - localStorage.time) > timeLimit)) {
            this.$http.get('https://api-stage.etm-system.com/api/login/' + app_id)
                .then(function (response) {
                    localStorage.api_key = response.data.etm_auth_key;
                    localStorage.time = now.getTime();
                })
        }
        else {
            this.api_key = localStorage.api_key;
        }
        this.requestHeader = { headers: {
                "Accept": "application/json, text/plain, */*",
                "Content-Type": "application/json;charset=utf-8",
                "X-Requested-With": "XMLHttpRequest",
                "etm-auth-key": this.api_key }};
        console.log('Request Header :');
        console.log(this.requestHeader);


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
            console.log('urlSearch - ' + this.urlSearch);
            this.searchQuery.directions[0].departure_code = this.iataDeparture;
            this.searchQuery.directions[0].arrival_code = this.iataArrival;
            this.searchQuery.directions[0].date = this.searchDate;
            this.searchQuery.class = this.selectedClass.code;
            console.info('searchQuery: ');
            console.info(this.searchQuery);
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
