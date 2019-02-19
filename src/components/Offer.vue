<template>
    <div class="offers">
        <div class="offers__airline airline" v-on:click="offerShowToggle">
            <div class="airline__left-block">
                <div class="airline__logo"><img v-bind:src="offer.carrier_logo" alt=""></div>
                <div class="airline__name">{{offer.carrier_name}}</div>
            </div>
            <div class="airline__minprice">Цена от {{offer.min_price}}</div>
        </div>

        <div class="offers__offer offer" v-bind:key="soIndex" v-for="(singleOffer, soIndex) in offer.offers" v-if="offerShow">
            <div class="offer__segments" v-bind:key="segmentsIndex" v-for="(segments, segmentsIndex) in singleOffer.segments">
                <div class="segments">
                    <div class="segments_segment" :key="segmentIndex" v-for="(segment, segmentIndex) in segments.flights_info">
                        <div class="segment">
                            <div class="segment__flight">
                                <div class="segment__flight-number">{{segment.flight_number_print}}</div>
                                <div class="segment__airplane-name">{{segment.airplane_name}}</div>
                            </div>
                            <div class="segment__departure">
                                <div class="segment__departure-airport">{{segment.departure_airport}}</div>
                                <div class="segment__departure-city">{{segment.departure_city}}</div>
                                <div class="segment__departure-time">{{segment.departure_local_time}}</div>
                            </div>
                            <div class="segment__duration-time">{{segment.duration_formated}}</div>
                            <div class="segment__arrival">
                                <div class="segment__arrival-airport">{{segment.arrival_airport}}</div>
                                <div class="segment__arrival-city">{{segment.arrival_city}}</div>
                                <div class="segment__arrival-time">{{segment.arrival_local_time}}</div>
                            </div>
                        </div>
                        <div class="segment__stop" v-if="segment.stop_time">Пересадка - {{segment.stop_time}}</div>
                    </div>
                </div>
                <div class="segments__price">{{ segments.price}} p</div>
            </div>
        </div>
    </div>
    
</template>

<script>
    export default {
        name: "offer",
        props: [
            'offer'
        ],
        data() {
            return {
                offerShow: false
            }
        },
        methods: {
            offerShowToggle() {
                this.offerShow = !this.offerShow;
            }
        }
    }
</script>

<style>

</style>