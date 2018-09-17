<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Contact Us</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-12">
                            <!--<div v-if="main" class="margin_60" v-html="main.body"></div>-->
                            <div class="margin_60">
                                Pellentesque vel commodo ligula. Suspendisse faucibus nec tellus eget dictum. Vivamus aliquam mi in mi sagittis, eu egestas dolor feugiat. Nunc sit amet mauris tortor. Vestibulum congue sed lectus nec accumsan. Proin non dui imperdiet, lacinia metus et, scelerisque turpis. Morbi vel quam sapien. Integer iaculis ac lorem eget maximus. Nunc accumsan metus eu nulla consequat, nec interdum nunc volutpat. Sed eu suscipit est. Curabitur eget facilisis est, et cursus nunc.
                            </div>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-4">
                            <div v-if="contactInfo" class="margin_60 padding_60" v-html="contactInfo.body"></div>    
                        </div>
                        <div class="col-md-8">
                            <transition name="fadeIn">
                                <div>
                                    <div id="send_contact_success" class="alert alert-success" role="alert" v-show="formSuccess">
                                        <span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
                                        <span class="sr-only">Success</span>
                                        Thank you for contacting us. A member from our team will contact you shortly.
                                    </div>
                                    <div id="send_contact_error" class="alert alert-danger" role="alert" v-show="formError">
                                        <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                                        <span class="sr-only">Error:</span>
                                        There was an error when trying to submit your request. Please try again later.
                                    </div>
                                </div>
                            </transition>
                            <form id="contact_form" class="form-horizontal clearfix" action="form-submit" v-on:submit.prevent="validateBeforeSubmit">
                                <div class="row">
                                    <div class="col-xs-12 col-md-6" :class="{'has-error': errors.has('name')}">
                                        <label class="accessibility" for="name">Name</label>
                                        <input id="name" v-model="form_data.name" v-validate="'required|alpha_spaces'" class="form-control" :class="{'input': true}" name="name" type="text" data-vv-delay="1000" placeholder="Name *">
                                        <span v-show="errors.has('name')" class="form-control-feedback">{{ errors.first('name') }}</span>
                                    </div>
                                    <div class="col-xs-12 col-md-6" :class="{'has-error': errors.has('email')}">
                                        <label class="accessibility" for="email">Email</label>
                                        <input id="email" v-model="form_data.email" v-validate="'required|email'" class="form-control" :class="{'input': true}" name="email" type="email" data-vv-delay="1000" placeholder="Email *">
                                        <span v-show="errors.has('email')" class="form-control-feedback">{{ errors.first('email') }}</span>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-xs-12 col-md-6" :class="{'has-error': errors.has('phone')}">
                                        <label class="accessibility" for="phone">Phone</label>
                                        <input id="phone" v-model="form_data.phone" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="phone" type="text" data-vv-delay="1000" placeholder="Phone *">
                                        <span v-show="errors.has('phone')" class="form-control-feedback">{{ errors.first('phone') }}</span>
                                    </div>
                                    <div class="col-xs-12 col-md-6" :class="{'has-error': errors.has('subject')}">
                                        <label class="accessibility" for="subject">Subject</label>
                                        <input id="subject" v-model="form_data.subject" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="subject" type="text" data-vv-delay="1000" placeholder="Subject *">
                                        <span v-show="errors.has('subject')" class="form-control-feedback">{{ errors.first('subject') }}</span>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-xs-12" :class="{'has-error': errors.has('message')}">
                                        <label class="accessibility" for="message">Message</label>
                                        <textarea id="message" v-model="form_data.message" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="message" type="text" data-vv-delay="1000" placeholder="Message *"></textarea>
                                        <span v-show="errors.has('message')" class="form-control-feedback">{{ errors.first('message') }}</span>
                                    </div>
                                </div>
                                <div class="row margin_60">
                                    <div class="col-xs-12">
                                        <button class="submit animated_btn" type="submit" :disabled="formSuccess">Submit</button>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "vue-meta", "vee-validate", "json!site.json"], function (Vue, Vuex, Meta, VeeValidate, Site) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("contact-component", {
            template: template, // the variable template will be injected
            data: function data() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    main: null,
                    contactInfo: null,
                    form_data: {},
                    loginPending: null,
                    formSuccess: false,
                    formError: false,
                    time: new Date()
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Contact Us Banner');
                    if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b71eb886e6f6450013c0000/image/jpeg/1529532304000/insidebanner2.jpg"
                        }
                    }
                    if(response && response[0]){
                        this.main = response[0].data
                        if(response[0].data && response[0].data.subpages){
                           this.contactInfo = response[0].data.subpages[0];
                        }
                    }
                    
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function () {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    try {
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/"+Site.subdomain+"-contact-us.json" }), this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmit() {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            send_data = {};
                            send_data.form_data = JSON.stringify(this.serializeObject(this.form_data));
                            this.$store.dispatch("CONTACT_US", send_data).then(res => {
                                this.formSuccess = true;
                            }).catch(error => {
                                try {
                                    if (error.response.status == 401) {
                                        console.log("Data load error: " + error.message);
                                        this.formError = true;
                                    } else {
                                        console.log("Data load error: " + error.message);
                                        this.formError = true;
                                    }
                                } catch (e) {
                                    console.log("Data load error: " + error.message);
                                    this.formError = true;
                                }
                            })
                        }
                    })
                },
                serializeObject(obj) {
                    var newObj = [];
                    _.forEach(obj, function (value, key) {
                        var tempVal = {};
                        tempVal.name = key;
                        tempVal.value = value;
                        newObj.push(tempVal);
                    });
                    return newObj;
                }
            }
        });
    });
</script>