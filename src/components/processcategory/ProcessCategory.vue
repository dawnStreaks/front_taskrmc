<template>
    <div class="app animated fadeIn">
        <loading :active.sync="this.$store.state.isLoading"
                 :can-cancel="true"
                 :is-full-page="this.$store.state.fullPage"></loading>
        <b-row>
            <b-col cols="12" xl="12" ref="formContainer">
                <transition name="slide">
                    <b-card>
                        <div slot="header">
                            <b class="h4">{{ get_text('title@process_cat@l') }}</b>
                            <b-button variant="primary" size="md" class="btn float-right" @click="showingModal= true" v-if="$can('prctype-create')">
                                <b><i class="icon-plus icons font-1xl"></i> {{ this.get_text('add@prcType@l') }}</b></b-button>
                        </div>
                        <div v-if="!$auth.ready() || !this.$store.state.loading">
                            <div style="text-align:center; padding-top:50px;">
                                {{ this.get_text('loading@prcType@l') }}
                            </div>
                        </div>
                        <div v-if="$auth.ready() && this.$store.state.loading">
                            <b-row>
                                <b-col md="4" class="my-1">
                                    <b-form-group horizontal :label="this.get_text('filter@l')" class="mb-0">
                                        <b-input-group>
                                            <b-form-input v-model="filter" :placeholder="get_text('type_to_search@l')"/>
                                            <b-input-group-append>
                                                <b-btn :disabled="!filter" @click="filter = ''">{{ get_text('clear@l') }}</b-btn>
                                            </b-input-group-append>
                                        </b-input-group>
                                    </b-form-group>
                                </b-col>
                                <b-col md="4" class="my-1">
                                    <b-form-group horizontal :label="this.get_text('sort@l')" class="mb-0">
                                        <b-input-group>
                                            <b-form-select v-model="sortBy" :options="sortOptions">
                                                <option slot="first" :value="null">{{ get_text('none@l') }}</option>
                                            </b-form-select>
                                            <b-form-select :disabled="!sortBy" v-model="sortDesc" slot="append">
                                                <option :value="false">{{ get_text('asc@l') }}</option>
                                                <option :value="true">{{ get_text('desc@l') }}</option>
                                            </b-form-select>
                                        </b-input-group>
                                    </b-form-group>
                                </b-col>
                                <b-col md="4" class="my-1">
                                    <b-form-group horizontal :label="get_text('per_page@l')" class="mb-0">
                                        <b-form-select :options="pageOptions" v-model="perPage"/>
                                    </b-form-group>
                                </b-col>
                            </b-row>
                            <b-table :hover="hover" :striped="striped" :bordered="bordered" :small="small"
                                     :fixed="fixed" responsive="sm" :items="categories" :fields="fields"
                                     :current-page="currentPage" :per-page="perPage" :filter="filter"
                                     :sort-by.sync="sortBy"
                                     :sort-desc.sync="sortDesc"
                                     :sort-direction="sortDirection"
                                     @filtered="onFiltered">
                                <template slot="Actions" slot-scope="data">
                                    <b-row class="text-center">
                                        <b-col cols="2" sm="4" md="2" xl="1" class="mb-3 mb-xl-0" v-if="$can('prctype-delete')">
                                            <b-button variant="primary" size="sm"
                                                      @click="editCategory(data.item, 'view') "><b><i
                                                    class="icon-eye icons font-1xl"></i></b></b-button>
                                        </b-col>&nbsp;
                                        <b-col cols="2" sm="4" md="2" xl="1" class="mb-3 mb-xl-0" v-if="$can('prctype-view')">
                                            <b-button variant="warning" size="sm"
                                                      @click="editCategory(data.item, 'edit')"><b><i
                                                    class="icon-pencil icons font-1xl"></i></b></b-button>
                                        </b-col>&nbsp;
                                        <b-col cols="2" sm="4" md="2" xl="1" class="mb-xl-0" v-if="$can('prctype-edit')"><b-button variant="danger" size="sm"
                                                      @click="showingdeleteModal = true;  selectCategory(data.item)"><b><i class="icon-trash icons font-1xl"></i></b></b-button>
                                        </b-col>&nbsp;
                                    </b-row>
                                </template>
                            </b-table>
                            <nav>
                                <b-pagination size="sm" :total-rows="getRowCount" :per-page="perPage"
                                              v-model="currentPage" :prev-text="this.get_text('prev@l')"                                              :next-text="this.get_text('next@l')"
                                              hide-goto-end-buttons/>
                            </nav>
                        </div>
                    </b-card>
                </transition>
                <b-modal :title="get_text('add@prcType@l')" size="lg" class="modal-primary" v-model="showingModal" hide-footer>
                         <!-- @close="setCategoryData" > -->
                    <b-form @submit.prevent="createcategory">
                        <b-row>
                                                          <b-col sm="6">
                                <b-form-group>
                                    <label for="category_name">{{ get_text('category_name@process_cat@l')}}<i class="fa fa-asterisk danger fa-xs"
                                                               style="font-size:10px;color:red;"></i></label>
                                    <b-form-input type="text" id="category_name1" v-validate="'required'" name="category_name"
                                                  v-model="categoryData.category_name" :placeholder="get_text('enter_prc_cat_name@process_cat@l')"
                                                  :class="{ 'is-invalid': updated && errors.has('prc_name') }"></b-form-input>
                                    <div v-if="updated && errors.has('category_name')" class="invalid-feedback">{{
                                        errors.first('category_name') }}
                                    </div>
                                </b-form-group>
                            </b-col>

                        </b-row>
                        <b-button class="float-right" variant="primary" size="lg" type="submit">{{ get_text('save@prcType@l')}}</b-button>
                    </b-form>
                </b-modal>
                <b-modal :title="get_text('edit@prcType@l')" size="lg" ref="vueModel" class="modal-primary" v-model="showingeditModal"
                         hide-footer>
                    <b-form @submit.prevent="updateCategory">
                        <b-row>
                            <b-col sm="6">
                                <b-form-group>
                                    <label for="category_name">{{ get_text('category_name@process_cat@l')}}<i class="fa fa-asterisk danger fa-xs"
                                                               style="font-size:10px;color:red;"></i></label>
                                    <b-form-input type="text" id="category_name2" v-validate="'required'" name="category_name"
                                                  v-model="categoryData.category_name" :placeholder="get_text('enter_prc_cat_name@process_cat@l')"
                                                  :class="{ 'is-invalid': updated && errors.has('prc_name') }"></b-form-input>
                                    <div v-if="updated && errors.has('category_name')" class="invalid-feedback">{{
                                        errors.first('category_name') }}
                                    </div>
                                </b-form-group>
                            </b-col>

                        </b-row>
                        <b-button class="float-right" variant="primary" size="lg" type="submit">{{ get_text('update@prcType@l') }}</b-button>
                    </b-form>
                </b-modal>
                <b-modal :title="get_text('delete@prcType@l')" size="lg" class="modal-danger" v-model="showingdeleteModal" hide-footer>
                    <div class="modalcontent">
                        <h3 class="text-center">{{ get_text('are_you_sure_to_delete@prcType@l') }} <b>{{categoryData.name}}</b>?</h3>
                        <b-col md="12" class="text-center">

                            <button class="btn btn-outline-danger left"
                                    @click="showingdeleteModal = false; removeCategory(categoryData)">{{ get_text('yes@prcType@l') }}
                            </button>&nbsp;
                            <button class="btn btn-outline-success right" @click="showingdeleteModal = false;">{{ get_text('no@prcType@l') }}
                            </button>
                        </b-col>
                    </div>
                </b-modal>
                <b-modal :title="get_text('information@prcType@l')" size="lg" class="modal-info" v-model="showingViewModal" hide-footer>
                         <!-- ok-variant="info"> -->
                    <div class="modalcontent" v-if="this.categoryInfo">
                        <b-row v-for="(item, index) in this.categoryInfo" :key="index">
                            <b-col sm="6">{{ get_text(index+'@process_cat'+'@c') }}</b-col>
                            <b-col sm="6">{{ item }}</b-col>
                        </b-row>
                    </div>
                </b-modal>
            </b-col>
        </b-row>
    </div>
</template>

<script>
    import {store} from '@/components/processcategory/store/store'
    import Vue from 'vue'
    import 'element-ui/lib/theme-chalk/index.css';
    import Element from 'element-ui'
    //import {Notification} from 'element-ui'
    import Loading from 'vue-loading-overlay';
    import 'vue-loading-overlay/dist/vue-loading.css';

    Vue.use(Element);
    Vue.use(Loading);
    export default {
        store: store,
        props: {
            hover: {
                type: Boolean,
                default: true
            },
            striped: {
                type: Boolean,
                default: true
            },
            bordered: {
                type: Boolean,
                default: false
            },
            small: {
                type: Boolean,
                default: false
            },
            fixed: {
                type: Boolean,
                default: false
            }
        },
        created() {
            //this.$store.dispatch('translation');
            this.$store.dispatch('loadCategoriesTable');
           // this.$store.dispatch('loadGroups')
                 this.$store.dispatch('loadCategory');

        },
        data: () => {
            return {
                selected: null,
                fields: [],
                button: '',
                currentPage: 1,
                perPage: 5,
                totalRows: 0,
                categoryData: {
                    id:null,
                    category_name: null,
                    tenant:null,
                },
                categoryInfo: null,
                showingeditModal: false,
                showingViewModal: false,
                showingModal: false,
                showingdeleteModal: false,
                submitted: false,
                updated: false,
                pageOptions: [10, 25, 50,100],
                sortBy: null,
                sortDesc: false,
                sortDirection: 'asc',
                filter: null,
                translation:[]
            }
        },
        beforeMount() {
            if (localStorage.getItem('translation')) {
                this.translation = JSON.parse(localStorage.getItem('translation'));
            }
        },
     
        computed: {
            categories() {
                this.fields=[
                    {key: 'id', label: '#', sortable: true, sortDirection: 'desc'},
                    {key: 'name', label: this.get_text('category_name@process_cat@c'), sortable: true, sortDirection: 'desc'},
                    {key: 'tenant_id', label: this.get_text('tenant@prcType@c')},
                    {key: 'Actions',label: this.get_text('actions@l'),thStyle: { textAlign : this.$i18n.locale=='ar'?'right':""}},
                    //{key: 'Delete', thClass: 'text-center', tdClass: 'text-center'},
                    //{key: 'View', thClass: 'text-center', tdClass: 'text-center'}
                ];
                return this.$store.getters.categories;
            },
            getRowCount() {
                return this.$store.getters.getRowCount
            },
            sortOptions() {
                // Create an options list from our fields
                return this.fields
                    .filter(f => f.sortable)
                    .map(f => {
                        return {text: f.label, value: f.key}
                    })
            }
        },
        watch: {
            showingeditModal: function () {
                if (this.showingeditModal == false) {
                    this.$store.dispatch('loadCategoriesTable')
                    this.categoryData = {}
                }
            }
        },
        components: {
            'loading': Loading
        },
        methods: {
            setCategoryData() {
                this.categoryData = {
                    category_name:'',
                }
            },
            get_text(textbit) {
                var transtext = this.translation[this.$i18n.locale][textbit];
                if (transtext) {
                    return transtext;
                } else {
                    var spliteText = (textbit.split("@"));
                    if(spliteText.length > 2) {
                        var newTxt = (spliteText[0]+'@'+'l');
                        var transtext1 = this.translation[this.$i18n.locale][newTxt];
                        if (transtext1) {
                            return transtext1;
                        } else {
                            return "!" + textbit;
                        }
                    } else {
                        return "!" + textbit;
                    }
                }

            },
            $can(permissionName) {
                // alert(this.$auth.user().allPermissions);
                return this.$auth.user().allPermissions.indexOf(permissionName) !== -1;
            },
            createcategory() {
                this.submitted = true;
                this.$validator.validate().then(valid => {
                    if (valid) {
                        this.resetPasswordBtn = true,
                            this.showingModal = false
                        this.$store.dispatch('createCategory', {
                            data: {
                                //  id:this.categoryData.id,
                                category_name: this.categoryData.category_name,
                                // Category: this.categoryData.Category,
                                // Tenant:this.categoryData.Tenant,
                            }
                        })
                        this.categoryData = {
                            // id:null,
                            category_name: null,
                            // Category: null,
                            // Tenant:null,
                        }
                        this.$validator.reset();
                    }
                });
            },

            
            selectCategory(category) {

                this.categoryData = category;

                   
            },
            editCategory(category, value = null) {
                this.$store.state.isLoading = true;
                Vue.axios.get(`/categorylist1/${category.id}`, category.data)
                    .then((response) => {
                        if (response.data.code == 401) {
                            Vue.prototype.$notify.error({
                            title: 'Error',
                            message: response.data.msg,
                            type: 'Error',
                            position: 'top-right'
                                                    })
                        } else {
                            if (value == 'view') {
                                this.showingViewModal = true;
                                this.categoryInfo = response.data.data
                            } else {
                                this.showingeditModal = true
                                this.categoryData = response.data.data
                            }
                        }
                        this.$store.state.isLoading = false;
                    })
                    .catch((error) => {
                        console.log(error)
                    })
            },
            updateCategory() {
                   this.updated = true;
                this.$validator.validate().then(valid => {
                    if (valid) {
                        this.showingeditModal = false
                        this.$store.dispatch('updateCategory', {
                            where: this.categoryData.id,
                            data: {
                                // id:this.processData.id,
                                 category_name: this.categoryData.category_name,
                                tenant:this.categoryData.tenant,
                            }
                        })
                        this.categoryData = {
                          //  id:null,
                             category_name: null,
                            // Tenant:null
                        }
                        this.$validator.reset();
                    }
                });
            },
            removeCategory(categoryData) {
                console.log(categoryData);
                this.$store.dispatch('removeCategory', categoryData)
            },
           
           onFiltered(filteredItems) {
                // Trigger pagination to update the number of buttons/pages due to filtering
                this.totalRows = filteredItems.length
                this.currentPage = 1
            }
        },
    }
</script>
<style>
    .modal-header .close {
        padding: 0 !important;
        margin: 0 !important;
    }
</style>
<style scoped>
    .card-body >>> table > tbody > tr > td {
        cursor: pointer;
    }
</style>
