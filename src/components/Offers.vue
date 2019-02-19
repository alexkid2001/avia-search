<template>
    <div class="offers-block container">
        <div class="offers-block__result-status">
            Статус поиска: {{ requestStatus }}
        </div>

        <div class="local-data">
            <div class="local-data__descr">
                Если нет результатов, попробуйте ID предыдущих запросов
            </div>
            <label class="etm-input">
                <input type="text" v-bind:value="request_id">
            </label>
            <div class="button button_inline" @click="localDataShow">Показать другой результат</div>
        </div>

        <div class="offers-block__wrap">
            <div class="offers-block__list" v-bind:key="index" v-for="(offer, index) in this.offers">
                <offers :offer="offer"></offers>
                <!--<div class="accordion">-->
                    <!--<div class="offers__airline airline accordion__visible">-->
                        <!--<div class="airline__logo"><img v-bind:src="offer.carrier_logo" alt=""></div>-->
                        <!--<div class="airline__name">{{offer.carrier_name}}</div>-->
                        <!--<div class="airline__minprice">Цена от {{offer.min_price}}</div>-->
                    <!--</div>-->
                    <!--<div class="accordion__hide">-->
                        <!--<div class="offers__offer offer" v-bind:key="soIndex" v-for="(singleOffer, soIndex) in offer.offers">-->
                            <!--<div class="offer__flight-number">Рейс - {{singleOffer.flight_number}}</div>-->
                            <!--<div class="offer__airline-name">{{singleOffer.flight_carrier_name}}</div>-->
                            <!--<div class="offer__departure-airport">{{singleOffer.departure_airport}}</div>-->
                            <!--<div class="offer__departure-time">{{singleOffer.departure_timestamp}}</div>-->
                            <!--<div class="offer__arrival-airport">{{singleOffer.arrival_airport}}</div>-->
                            <!--<div class="offer__arrival-time">{{singleOffer.arrival_timestamp}}</div>-->
                            <!--<div class="offer__duration-time">{{singleOffer.duration_formated}}</div>-->
                        <!--</div>-->

                    <!--</div>-->
                <!--</div>-->
            </div>
        </div>
    </div>
</template>

<script>
    import Offer from './Offer'

    export default {
        name: "offers-block",
        components: {
            'offers': Offer
        },
        props: [
            'request_id',
            'requestHeader'
        ],
        data() {
            return {
                offers:[],
                requestStatus: '',
                requestMsg: ''
            }
        },
        methods: {
            updateSource: function () {
                console.log('Search request');
                this.$http.get(this.urlOffers, this.requestHeader)
                    .then(response => {
                        console.log(response.body);
                        this.requestStatus = response.body.status;
                        this.requestMsg = response.body.message;
                        this.offers = response.body.offers;
                        console.log('status - ' + this.requestStatus);
                        console.log('message - ' + this.requestMsg);
                        console.log('search results - ' + this.offers);
                    })
                    .catch(err => {
                        console.log('status - ' + err.status);
                        console.log('error - ' + err.error);
                    })
            },
            localDataShow() {
                // // let json = require('data/16707600.json');
                // console.log('add json data...');
                // this.$http.get('http://localhost:8080/data/16707600.json')
                //     .then(response => {
                //         this.offers = response.body.offers;
                //         console.log(this.offers);
                //     })
                //     .catch(err => {
                //     console.log('status - ' + err.status);
                //     console.log('error - ' + err.error);
                // })
                this.request_id = '16707825';
            }
        },
        watch: {
            request_id: function () {
                this.urlOffers = 'https://api-stage.etm-system.com/api/air/offers?request_id=' + this.request_id + '&sort=price';
                console.log('request_id - ' + this.request_id);
                console.log('URL - ' + this.urlOffers);
                console.log('headers - ' + this.requestHeader);
                this.updateSource();
            }

        }
    }
</script>


<style scoped lang="scss">

.offers {
    font-family: Roboto;

    &__results-status {
        font-size: 16px;
        font-weight: 500;
        text-align: center;
    }
}

</style>