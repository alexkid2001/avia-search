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
                <input type="text" :value="request_id">
            </label>
            <div class="button button_inline" @click="localDataShow">Показать другой результат</div>
        </div>

        <div class="offers-block__wrap">
            <div class="offers-block__filters">
                <div class="filter">
                    <div class="filter__name">Фильтр по цене:</div>
                    <div class="filter__rule">
                        <vue-slider
                                ref="slider"
                                v-bind="price"
                                v-model="price.value"
                                @input="maxPrice">
                        </vue-slider>
                    </div>
                </div>
            </div>

            <div class="offers-block__offers">

                <div>Найдено {{offersCnt}} предложений</div>
                <div class="offers-block__list" v-bind:key="index" v-for="(offer, index) in filteredPrice()">
                    <offers
                        :offer="offer"
                        :price="price.value">
                    </offers>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import Offer from './Offer'
    import vueSlider from 'vue-slider-component'

    export default {
        name: "offers-block",
        components: {
            'offers': Offer,
            vueSlider
        },
        props: [
            'request_id',
            'requestHeader'
        ],
        data() {
            return {
                offers:[],
                requestStatus: '',
                requestMsg: '',
                // maxPrice: 0,
                max: 0,
                price: {
                    value: [0, 5500],
                    min: 0,
                    max: 100,
                    width: 'auto'
                },
                offersCnt: ''
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
                // let json = require('data/16707600.json');
                console.log('add json data...');
                this.$http.get('data/16707600.json')
                    .then(response => {
                        this.offers = response.body.offers;
                        console.log(this.offers);
                        this.maxminPrices(this.offers);
                    })
                    .catch(err => {
                        console.log('status - ' + err.status);
                        console.log('error - ' + err.error);
                })
                // this.request_id = '16707825';
            },

            maxminPrices(obj) {
                let maxPrice = 0;
                obj.forEach(function(offer, i) {
                    // console.log('Offer - ' + i);
                    offer.offers.forEach(function(item) {
                        item.segments.forEach(function(segment, j) {
                            // console.log('Offer - ' + i + ' - segment - ' + j);
                            // console.log(segment.price);
                            if( segment.price > maxPrice ) {
                                maxPrice = segment.price;
                            }
                        })
                    })
                });
                this.price.max = maxPrice;
                this.price.value = [0, maxPrice];
            },
            filteredPrice() {
                let maxPrice = this.price.value[1];
                let minPrice = this.price.value[0];
                let arrOffers;

                arrOffers = this.offers.filter( offer => {
                    let segment = offer.offers.filter( segment => {

                        let arrItem = segment.segments.filter( item => {
                            return item.price < maxPrice && item.price > minPrice
                        })
                        return arrItem.length > 0;
                    })

                    return segment.length > 0;
                })
                this.offersCnt = this.countOffer(arrOffers);
                return arrOffers;
            },
            maxPrice() {
               this.max = this.price.value[1];
            },
            countOffer(obj) {
                let cnt = 0;
                obj.forEach(item => {
                    item.offers.forEach( offer => {
                        cnt += offer.segments.length;
                    })
                })
                return cnt;
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