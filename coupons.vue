<template>
    <main> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h1>Online Offers</h1>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row margin_40" v-if="couponList.length > 0">
        		        <div class="col-md-12 clearfix">
        		            <!--<router-link  :to="{ name: 'couponBasket', params: { selected: selected_coupon_id }}">-->
        		                <div @click="addCookies()" class="animated_btn coupon_btn">
        		                    My Basket ({{ basketItems }})
        		                </div>    
        		            <!--</router-link>-->
        		        </div>
        		    </div>
                    <div class="row" v-if="couponList.length > 0">
                        <div v-for="(item, index) in couponList" class="col-md-6 col-sm-6 col-xs-12">
                            <div :id="item.id" class="row coupon_container">
                                <div class="col-md-6 col-sm-6 col-xs-12">
                                    <div class="coupon_img">
                                        <img class="img_max" :src="item.store_logo" :alt="item.store.name + ' Logo'" />
                                    </div>
                                </div>	
                                <div class="col-md-6 col-sm-6 col-xs-12">
                                    <div class="coupon_content">
                                        <span style="visibility: hidden">{{ selectedCoupon }}</span>
                                        <span @click="selectCoupon(item)">
                                            <i v-if="item.is_in_cart"  class="fas fa-check"></i>
                                            <i v-else class="fas fa-shopping-basket"></i>    
                                        </span>
                                        <div>
                                            <p v-if="item.store">{{ item.store.name }}</p>
                                        	<h2 class="coupon_title">{{ item.name_short }}</h2>
                                        	<p class="coupon_dates"><span v-if="isMultiDay(item)">{{ item.start_date | moment("MM/DD/YYYY", timezone)}} - {{ item.end_date | moment("MM/DD/YYYY", timezone)}}</span><span v-else>{{ item.start_date | moment("MM/DD/YYYY", timezone)}}</span></p>
                                            <router-link :to="{ name: 'couponDetails', params: { id: item.slug }}">
                                                <p class="event_link coupon_link">Coupon Details <i class="fas fa-angle-double-right"></i></p>
                                            </router-link>
                                        </div>
                                    </div>
                                </div>
                            </div> 
                        </div>
                    </div>
                    <div class="row margin_40" v-else>
                        <div class="col-xs-12">
                            <p>There are no coupons posted at this time. Please check back soon!</p>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </main>
</template>
              
<script>
    define(["Vue", "vuex", "jquery", 'js-cookie', "moment", "moment-timezone", "vue-moment", "vue-lazy-load"], function (Vue, Vuex, $, Cookies, moment, tz, VueMoment, VueLazyload) {
        Vue.use(VueLazyload);
        return Vue.component("coupons-component", {
            template: template, // the variable template will be injected,
            data: function () {
                return {
                    dataLoaded: false,
                    selectedCoupon: false,
                    selected_coupon_id: [],
                    cached_coupon_id: null,
                    basketItems: 0,
                    events: [],
                    moreEvents: [],
                    moreEventsFetched: false,
                    noMoreEvents: false,
                    noEvents: false
                }
            },
            beforeRouteUpdate (to, from, next) {
                this.initalizeCouponsfromCookies();
                next();
            },
            created (){
                this.loadData().then(response => {
                    // Cookies.remove('coupon_ids');
                    // this.cachedCoupons();
                    // this.selectedCoupons = Cookies.get('coupon_ids');
                    var temp_repo = this.findRepoByName('Coupons Banner');
                    if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b71eb886e6f6450013c0000/image/jpeg/1529532304000/insidebanner2.jpg"
                        }
                    }
                    this.initalizeCouponsfromCookies();
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedCoupons',
                    'findCouponById',
                    'findRepoByName'
                ]),
                couponList: function coupons() {
                    var vm = this;
                    var showCoupons = [];
                    _.forEach(this.processedCoupons, function(value, key) {
                        var today = moment.tz(this.timezone).format();
                        var showOnWebDate = moment.tz(value.show_on_web_date, this.timezone).format();
                        if (today >= showOnWebDate) {
                            if (_.includes(value.store.store_front_url_abs, 'missing')) {
                                value.store_logo = vm.property.default_logo;
                            } else {
                                value.store_logo = value.store.store_front_url_abs;
                            }
                            
                            if (_.includes(value.image_url, 'missing')) {
                                value.image_url = vm.property.default_logo;
                            }
                            
                            value.name_short = _.truncate(value.name, { 'length': 30, 'separator': ' ' });

                            value.is_in_cart = false; 
                                
                            showCoupons.push(value);
                        }
                    });
                    
                    var cached_coupon_id = Cookies.get('coupon_ids');
                    if(cached_coupon_id){
                        cached_coupon_id = JSON.parse(cached_coupon_id);
                        _.forEach(showCoupons, function(value, key) {
                            var coupon = value
                            _.forEach(cached_coupon_id, function(value, key) {
                                if (_.includes(value, coupon.id)) {
                                    coupon.is_in_cart = true;
                                }  
                            });
                        });
                        
                        this.basketItems = cached_coupon_id.length;
                    } else {
                        this.cached_coupon_id = null;
                    }
                    
                    var sortedCoupons = _.orderBy(showCoupons, [function(o) { return o.end_date; }]);
                    
                    return sortedCoupons;    
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "coupons"),this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                isMultiDay(item) {
                    var timezone = this.timezone
                    var start_date = moment(item.start_date).tz(timezone).format("MM-DD-YYYY")
                    var end_date = moment(item.end_date).tz(timezone).format("MM-DD-YYYY")
                    if (start_date === end_date) {
                        return false
                    } else {
                        return true
                    }
                },
                selectCoupon(item){
                    var vm = this;
                    this.$nextTick(function() {
                        var current_coupon_id = item.id;
                        item.is_in_cart = !item.is_in_cart
                        vm.selectedCoupon = !vm.selectedCoupon
                        if (item.is_in_cart) {
                            vm.basketItems = this.basketItems + 1
                            vm.selected_coupon_id.push(current_coupon_id);
                        } else {
                            vm.basketItems = vm.basketItems - 1
                            //remove coupon from list variable
                            vm.selected_coupon_id = $.grep(vm.selected_coupon_id, function(val, i) {
                                return (val !== current_coupon_id);
                            });
                        }
                        
                        // var cached_coupon_id = Cookies.get('coupon_ids');
                        // if(cached_coupon_id){
                        //     cached_coupon_id = JSON.parse(cached_coupon_id);
                        //     console.log("Cached Coupons ", cached_coupon_id)
                        // }
                        
                        // Add updated coupon list to localstorage
                        Cookies.set('coupon_ids', '');
                        Cookies.set('coupon_ids', JSON.stringify(vm.selected_coupon_id));
                        // console.log("Cookies ", JSON.stringify(vm.selected_coupon_id))
                    });
                },
                addCookies() {
                    var vm = this;
                    // var cached_coupon_id = Cookies.get('coupon_ids');
                    // if(cached_coupon_id){
                    //     cached_coupon_id = JSON.parse(cached_coupon_id);
                    // }
                        
                    // Add updated coupon list to localstorage
                    Cookies.set('coupon_ids', '');
                    Cookies.set('coupon_ids', JSON.stringify(vm.selected_coupon_id));
                
                    vm.$router.push({ name: 'couponBasket', params: { selected: vm.selected_coupon_id }});
                },
                initalizeCouponsfromCookies (){
                    if(Cookies.get('coupon_ids') !== null && Cookies.get('coupon_ids') !== undefined && Cookies.get('coupon_ids').length > 0){
                        this.selected_coupon_id = JSON.parse(Cookies.get('coupon_ids'));
                    }
                }
            }
        });
    });
</script>
