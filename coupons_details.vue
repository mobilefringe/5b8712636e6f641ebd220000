<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Coupons</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div v-if="currentCoupon">
                        <div class="row">
                            <div class="col-md-2">
                                <img :src="currentCoupon.store_logo" :alt="currentCoupon.name + ' Logo'" class="margin_20 img_max"/>    
                            </div>
                            <div class="col-md-10">
                                <p v-if="currentCoupon.store" class="event_store_name">{{ currentCoupon.store.name }}</p>
                                <p v-else class="event_store_name">{{ property.name }}</p>
                                <h4 class="event_name">{{ currentCoupon.name }}</h4>
                                <p class="event_dates">
                                    <span v-if="isMultiDay(currentCoupon)">{{ currentCoupon.start_date | moment("MM/DD/YYY", timezone)}} - {{ currentCoupon.end_date | moment("MM/DD/YYY", timezone)}}</span>
                                    <span v-else>{{ currentCoupon.start_date | moment("MM/DD/YYY", timezone)}}</span>
                                </p>
                                <div class="event_dates">
                                    <img v-if="!_.includes(this.currentCoupon.promo_image_url_abs, 'missing')" :src="currentCoupon.image_url" :alt="currentCoupon.name" />
                                </div>
                                <div class="event_desc event_details" v-html="currentCoupon.rich_description"></div>
                                <div class="row margin_30">
                                    <div class="col-md-12">
                                        <div class="animated_btn coupon_detail_btn pull-left" v-if="!alreadyAdded" @click="addToBasket">Add To Basket</div>
                		                <div class="animated_btn coupon_detail_btn pull-left" v-if="alreadyAdded" @click="removeFromBasket">Remove From Basket</div>
                		                <div @click="printPage()" class="animated_btn coupon_detail_btn pull-left">Print Coupon</div>
                                    </div>
                                </div>
                                <social-sharing v-if="currentCoupon" :url="shareURL(currentCoupon.slug)" :title="currentCoupon.title" :description="currentCoupon.body" :quote="truncate(currentCoupon.body)" :media="currentCoupon.image_url" inline-template>
                                    <div class="social_share margin_60">
                                        <network network="facebook">
                                            <i class="fab fa-facebook"></i>
                                        </network>
                                    
                                        <network network="email">
                                            <i class="fas fa-envelope"></i>
                                        </network>
                                    </div>
                                </social-sharing>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
              
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "lightbox", "vue-lazy-load",  "vue-social-sharing", "json!site.json", 'js-cookie'], function(Vue, Vuex, moment, tz, VueMoment, Lightbox, VueLazyload, SocialSharing, site, Cookies) {
        Vue.use(VueLazyload);
        Vue.component('social-sharing', SocialSharing);
        return Vue.component("coupon-details-component", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    currentCoupon: null,
                    siteInfo: site,
                    alreadyAdded: false,
                    selected_coupon_id: [],
                }
            },
            created() {
                this.loadData().then(response => {
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
                    
					this.currentCoupon = this.findCouponBySlug(this.id);
					if (this.currentCoupon === null || this.currentCoupon === undefined) {
						this.$router.replace({ name: 'coupons' });
					}
					else {
                        if (this.currentCoupon.promotionable_type === "Store"){
                            if (_.includes(this.currentCoupon.store.store_front_url_abs, 'missing')) {
                                this.currentCoupon.store_logo = this.property.default_logo
                            } else {
                                this.currentCoupon.store_logo = this.currentCoupon.store.store_front_url_abs;
                            }
                            
                            // if (_.includes(this.currentCoupon.promo_image_url_abs, 'missing')) {
                            //     this.currentCoupon.image_url = "//codecloud.cdn.speedyrails.net/sites/5b71fb226e6f645093080000/image/png/1529532181000/promoplaceholder2@2x.png";
                            // } else {
                            //     this.currentCoupon.image_url = this.currentCoupon.promo_image_url_abs
                            // }
                        } else {
                            // if  (_.includes(this.currentCoupon.promo_image_url_abs, 'missing')) {
                            //     this.currentCoupon.image_url = "//codecloud.cdn.speedyrails.net/sites/5b71fb226e6f645093080000/image/png/1529532181000/promoplaceholder2@2x.png";    
                            // }
                        }
                        if(_.includes(this.selected_coupon_id, this.currentCoupon.id)){
                            this.alreadyAdded = true;
                        }
					}
					this.$breadcrumbs[2].meta.breadcrumb = this.currentCoupon.name
					
					this.dataLoaded = true;
					
					//get coupons from cookie
					if(Cookies.get('coupon_ids') !== null && Cookies.get('coupon_ids') !== undefined && Cookies.get('coupon_ids').length > 0){
                        this.selected_coupon_id = JSON.parse(Cookies.get('coupon_ids'));
                    }
                });
				
			},
			watch: {
                currentCoupon : function (){
                    
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findCouponBySlug',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "coupons"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
				isMultiDay(currentCoupon) {
					var timezone = this.timezone
					var start_date = moment(currentCoupon.start_date).tz(timezone).format("MM-DD-YYYY")
					var end_date = moment(currentCoupon.end_date).tz(timezone).format("MM-DD-YYYY")
					if (start_date === end_date) {
						return false
					} else {
						return true
					}
				},
			    printPage(currentCoupon) {
			        var timezone = this.timezone
			        var start_date = moment(this.currentCoupon.start_date).tz(timezone).format("MM/DD/YYYY");
					var end_date = moment(this.currentCoupon.end_date).tz(timezone).format("MM/DD/YYYY");
					var dates = "";
					if (start_date === end_date) {
						dates = start_date + " to " + end_date;
					} else {
						dates = start_date;
					}
					
                    var w = window.open();

                    var headers = this.currentCoupon.name;
                    var field = this.currentCoupon.promo_image_url_abs;
                    var field2 = dates;
                    var field3 = this.currentCoupon.description;
                    
                    var html = "<!DOCTYPE HTML>";
                    html += '<html lang="en-us">';
                    html += '<head></head>';
                    html += "<body><div style='text-align:center;'><img style='max-width: 250px; padding: 15px 0;' src='//codecloud.cdn.speedyrails.net/sites/5b71fb226e6f645093080000/image/png/1535042524544/dtl_600x180_logo.png' style='width:200px;'/></div><div style='margin: 0 auto; max-width: 700px; min-height: 300px; border-top: solid 1px #ccc; border-bottom: solid 1px #ccc;'><div style='width:50%; display: inline-block; vertical-align: top; text-align: left;'>";
                
                    //check to see if they are null so "undefined" doesnt print on the page. <br>s optional, just to give space
                    if(field != null && !_.includes(this.currentCoupon.promo_image_url_abs, 'missing')) html += "<img style='padding: 20px; max-width: 300px;' src=" + field + "></div><div style='width:50%; display: inline-block;'>";
                    if(headers != null) html += "<h4>" + headers + "</h4>";
                    if(field2 != null) html += "<h4>" + field2 + "</h4>";
                    if(field3 != null) html += "<p>" + field3 + "</p></div>";
                    html += "</div></body>";
                    
                    w.document.write(html);
                    setTimeout(function(){ w.window.print(); }, 150);
                    
                    w.document.close();
                },
				truncate(val_body) {
                    var truncate = _.truncate(val_body, { 'length': 99, 'separator': ' ' });
                    return truncate;
                },
				shareURL(slug) {
                    var share_url = window.location.href
                    return share_url
                },
                addToBasket(){
                    var vm = this;
                    this.selected_coupon_id.push(vm.currentCoupon.id);
                    this.alreadyAdded = true;
                    this.updateCookie()
                },
                removeFromBasket(){
                    
                    var vm = this;
                    //remove coupon from list variable
                    this.selected_coupon_id = $.grep(vm.selected_coupon_id, function(val, i) {
                        return (val !== vm.currentCoupon.id);
                    });
                    this.alreadyAdded = false;
                    this.updateCookie()
                },
                updateCookie(){
                    var vm = this;
                    // Add updated coupon list to localstorage
                    Cookies.set('coupon_ids', '');
                    Cookies.set('coupon_ids', JSON.stringify(vm.selected_coupon_id));
                }
			}
        });
    });
</script>