<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header">
                    <div class="main_container position_relative">
                        <h2>Newsletter</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                            <div v-if="pageContent" v-html="pageContent.body"></div>
                            <form class="newsletter_form form-horizontal" action="//mobilefringe.createsend.com/t/d/s/vjklyu/" method="post" @submit.prevent="validateBeforeSubmit">
                                <div class="row">
                                    <div class="col-sm-6" >
                                        <label for="fieldfihudt" class="accessibility">First Name</label>
                                        <input v-model="form_data.first_name" required class="margin_20 form-control" id="fieldfihudt" name="cm-f-fihudt" type="text" placeholder="First Name">
                                    </div>
                                    <div class="col-sm-6" >
                                        <label for="fieldfihudi" class="accessibility">Last Name</label>
                                        <input v-model="form_data.last_name" required class="margin_20 form-control" id="fieldfihudi" name="cm-f-fihudi" type="text" placeholder="Last Name">
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-sm-6">
                                        <label for="newsletter_email" class="accessibility">Email</label>
                                        <input v-model="form_data.email" required class="margin_20 form-control" name="cm-vjklyu-vjklyu" type="email" placeholder="Email" id="newsletter_email">
                                    </div>
                                    <div class="col-sm-6">
                                        <label for="fieldfihudd" class="accessibility">Phone Number</label>
                                        <input id="fieldfihudd" v-model="form_data.phone" required class="margin_20 form-control" name="cm-f-fihudd" type="text" placeholder="Phone Number" />
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-sm-12">
                                        <div style="margin-left: 20px">
                                            <label class="checkbox">
                                                <input name="agree_newsletter" required  type="checkbox">
                                                    I agree to receive communications from {{ property.name }}.
                                            </label>
                                        </div>
            					    </div>
            					</div>
        					    <div class="margin_40 clearfix"></div>
        					    <div class="row">
                                    <div class="col-xs-12">
                                        <button class="animated_btn" type="submit" :disabled="formSuccess">Subscribe</button>
                                    </div>
                                </div>
                                <div id="send_contact_success" class="alert alert-success" role="alert" v-show="formSuccess">
                                    <span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
                                    <span class="sr-only">{{$t("newsletter_page.success")}} : </span>
                                    Thank you! Your subscription has been confirmed.
                                </div>
                                <div id="send_contact_error" class="alert alert-danger" role="alert" v-show="formError">
                                    <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                                    <span class="sr-only">{{$t("newsletter_page.error")}} : </span>
                                    There was an error when trying to submit your request. Please try again later.
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
    define(["Vue", "vuex", "jquery", "vee-validate", "json!site.json"], function(Vue, Vuex, $, VeeValidate, site) {
        Vue.use(VeeValidate);
        return Vue.component("newsletter-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataLoaded: true,
                    pageContent: null,
                    siteInfo: site,
                    form_data : {},
                    formSuccess : false,
                    formError: false
                }
            },
            mounted () {
                this.form_data.first_name = this.$route.query.name;
                $("#fieldfihudt").val(this.form_data.first_name);
                this.form_data.email = this.$route.query.email;
                $("#newsletter_email").val(this.form_data.email);
            },
            watch : {
                $route () {
                    this.form_data.first_name = this.$route.query.name;
                    $("#fieldfihudt").val(this.form_data.first_name);
                    this.form_data.email = this.$route.query.email;
                    $("#newsletter_email").val(this.form_data.email);
                }
            },
            created() {
                this.loadData().then(response => {
                    this.pageContent = response[0].data;
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property'
                ])
            },
            methods: {
                loadData: async function () {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    try {
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/peninsula-newsletter.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmit(form) {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            
                            if(errors.length > 0) {
                                console.log("Error");
                                this.formError = true;
                            }
                            else {
                                form.preventDefault();
                                console.log("No Error", form);
                                var vm = this;
                                $.getJSON(
                                form.target.action + "?callback=?",
                                $(form.target).serialize(),
                                function (data) {
                                    if (data.Status === 400) {
                                       vm.formError = true;
                                    } else { // 200
                                        vm.formSuccess = true;
                                    }
                                });
                                
                            }
                        }
                    })
                }
            }
        });
    });
</script>