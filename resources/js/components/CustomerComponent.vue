<template>
    <div class="customers">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h4>All Customers</h4>
                        <button class="btn btn-primary" @click="CreateCustomer" style="position: absolute; right: 120px; top: 10px">
                            <i class="fas fa-plus"></i>
                            <span>Add Customer</span>
                        </button>
                        <button class="btn btn-success" @click="reload()" style="position: absolute; right: 10px; top: 10px">
                            <i class="fas fa-sync"></i>
                            <span>Reload</span>
                        </button>
                    </div>
                    <div class="card-body">
                        <div class="row">
                            <div class="col-md-2 pt-2">
                                <h6><strong>Search By</strong></h6>
                            </div>
                            <div class="col-md-3 pb-2">
                                <select class="form-control" v-model="searchField">
                                    <option value="name">Name</option>
                                    <option value="email">Email</option>
                                    <option value="phone">Phone</option>
                                    <option value="address">Address</option>
                                    <option value="total">Total</option>
                                </select>
                            </div>
                            <div class="col-md-7 pb-2">
                                <input type="text" v-model="query" class="form-control" placeholder="Search">
                            </div>

                        </div>
                        <table class="table table-striped table-bordered table-hover">
                            <thead>
                            <tr>
                                <th width="5%">Serial</th>
                                <th width="15%">Name</th>
                                <th width="10%">Email</th>
                                <th width="15%">Phone</th>
                                <th width="20%">Address</th>
                                <th width="10%">Total</th>
                                <th width="25%">Action</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-show="customers.length" v-for="(customer, index) in customers" :key="customer.id">
                                <td>{{index + 1}}</td>
                                <td>{{customer.name}}</td>
                                <td>{{customer.email}}</td>
                                <td>{{customer.phone}}</td>
                                <td>{{customer.address}}</td>
                                <td>{{customer.total}}</td>
                                <td>
                                    <button type="button" class="btn btn-primary btn-sm" title="Show" @click="show(customer)">
                                        <i class="fas fa-eye"></i>
                                    </button>
                                     <button type="button" class="btn btn-primary btn-sm" title="Edit" @click="edit(customer)">
                                         <i class="fas fa-edit"></i>
                                     </button>
                                    <button type="button" class="btn btn-danger btn-sm" title="Delete" @click="destroy(customer)">
                                        <i class="fas fa-trash"></i>
                                    </button>
                                </td>
                            </tr>
                            <tr v-show="!customers.length">
                                <td colspan="7">
                                    <div class="alert alert-danger" role="alert">
                                       Oops!! Data Not Found
                                    </div>
                                </td>
                            </tr>
                            </tbody>
                        </table>
                        <pagination-component v-if="pagination.last_page > 1"
                              :pagination="pagination"
                              :offset="5"
                              @paginate="query==='' ? getCustomers():searchData() "
                        ></pagination-component>
                    </div>
                </div>
            </div>
        </div>
        <vue-progress-bar></vue-progress-bar>
        <vue-snotify></vue-snotify>

        <!--Add Customer-->
        <div class="modal fade" id="addCustomerModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLongTitle">{{editMode ? 'Edit' : 'Add'}} Customer</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? update() : addCustomer()" @keydown="form.onKeydown($event)">
                    <div class="modal-body">
                        <alert-error :form="form"></alert-error>
                            <div class="form-group">
                                <label>Name</label>
                                <input v-model="form.name" type="text" name="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Email</label>
                                <input v-model="form.email" type="email" name="email"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Phone</label>
                                <input v-model="form.phone" type="text" name="phone"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }">
                                <has-error :form="form" field="phone"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Address</label>
                                <textarea rows="5" name="address" v-model="form.address" class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }"></textarea>
                                <has-error :form="form" field="address"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Total</label>
                                <input v-model="form.total" type="text" name="total"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('total') }">
                                <has-error :form="form" field="total"></has-error>
                            </div>
                    </div>
                    <div class="modal-footer">
                        <button :disabled="form.busy" type="submit" class="btn btn-primary" v-if="!editMode">Save</button>
                        <button :disabled="form.busy" type="submit" class="btn btn-primary" v-else>Update</button>
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    </div>
                    </form>
                </div>
            </div>
        </div>

        <!--Show Customer-->
        <div class="modal fade" id="showCustomerModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLongTitle">Customer Details</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                        <div class="modal-body">
                            <alert-error :form="form"></alert-error>
                            <div class="form-group">
                                <label>Name</label>
                                <input v-model="form.name" type="text" name="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }" readonly>
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Email</label>
                                <input v-model="form.email" type="email" name="email"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }" readonly>
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Phone</label>
                                <input v-model="form.phone" type="text" name="phone"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }" readonly>
                                <has-error :form="form" field="phone"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Address</label>
                                <textarea rows="5" name="address" v-model="form.address" class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }" readonly></textarea>
                                <has-error :form="form" field="address"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Total</label>
                                <input v-model="form.total" type="text" name="total"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('total') }" readonly>
                                <has-error :form="form" field="total"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    export default {
        data(){
            return{
                searchField: 'name',
                query:'',
                customers:[],
                pagination:{
                    current_page:1
                },
                form:new Form({
                    id:'',
                    name:'',
                    email:'',
                    phone:'',
                    address:'',
                    total:'',
                }),
                editMode:false,

            }
        },
        mounted() {

        },
        created(){
           this.getCustomers()
        },

        computed:{

        },

        watch:{
            query:function (newQ, old) {
               if (newQ === ''){
                   this.getCustomers()
               }else{
                  this.searchData()
               }
            }
        },
        
        methods:{
            addCustomer(){
                this.$Progress.start()
                this.form.busy = true
                this.form.post('/api/customers')
                    .then(response => {
                        this.getCustomers()
                        $('#addCustomerModal').modal('hide');
                        if(this.form.successful){
                            this.$Progress.finish()
                            this.$snotify.success('Customer Added Successfully','Success')
                        }else{
                            this.$Progress.fail()
                            this.$snotify.error('Something Went Wrong','Error')
                        }
                    })
                    .catch( err => {
                        console.log(err)
                        this.$Progress.fail()
                    })
            },
            getCustomers(){
                this.$Progress.start()
                axios.get('/api/customers?page='+this.pagination.current_page)
                    .then( (response)=>{
                        this.customers = response.data.data
                        this.pagination = response.data.meta
                        this.$Progress.finish()
                    })
                    .catch( (error)=> {
                        console.log(error)
                        this.$Progress.fail()
                    })
            },
            searchData(){
                this.$Progress.start()
                axios.get('/api/search/customers/'+this.searchField+'/'+this.query+'?page='+this.pagination
                    .current_page)
                    .then( (response)=>{
                        this.customers = response.data.data
                        this.pagination = response.data.meta
                        this.$Progress.finish()
                    })
                    .catch( (error)=> {
                        console.log(error)
                        this.$Progress.fail()
                    })
            },

            CreateCustomer(){
                this.form.clear()
                this.form.reset()
                $('#addCustomerModal').modal('show');
                this.editMode=false
            },

            edit(customer){
                this.editMode =true
                this.form.clear()
                this.form.reset()
                this.form.fill(customer)
                $('#addCustomerModal').modal('show');
            },
            show(customer){
                this.form.clear()
                this.form.reset()
                this.form.fill(customer)
                $('#showCustomerModal').modal('show');
            },
            update(){
                this.$Progress.start()
                this.form.busy = true
                this.form.put('/api/customers/'+this.form.id)
                    .then(response => {
                        this.getCustomers()
                        $('#addCustomerModal').modal('hide');
                        if(this.form.successful){
                            this.$Progress.finish()
                            this.$snotify.success('Customer Updated Successfully','Success')
                        }else{
                            this.$Progress.fail()
                            this.$snotify.error('Something Went Wrong','Error')
                        }
                    })
                    .catch( err => {
                        console.log(err)
                        this.$Progress.fail()
                    })
            },

            reload(){
                this.searchField= 'name'
                this.query=''
                this.getCustomers()
                this.$snotify.success('Page Reload Successfully', 'Success')
            },
            destroy(customer){
                this.$snotify.clear();
                this.$snotify.confirm(
                    "You will not be able to recover this data!",
                    "Are you sure?",
                    {
                        showProgressBar: false,
                        closeOnClick: false,
                        pauseOnHover: true,
                        buttons: [
                            {
                                text: "Yes",
                                action: toast => {
                                    this.$snotify.remove(toast.id);
                                    this.$Progress.start();
                                    axios.delete("/api/customers/" + customer.id)
                                        .then(response => {
                                            this.getCustomers();
                                            this.$Progress.finish();
                                            this.$snotify.success(
                                                "Customer Successfully Deleted",
                                                "Success"
                                            );
                                        })
                                        .catch(e => {
                                            this.$Progress.fail();
                                            console.log(e);
                                        });
                                },
                                bold: true
                            },
                            {
                                text: "No",
                                action: toast => {
                                    this.$snotify.remove(toast.id);
                                },
                                bold: true
                            }
                        ]
                    }
                );
            }
        }
    }
</script>
