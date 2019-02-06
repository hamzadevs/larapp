<template>
    <div class="container">
        <div class="row mt-3" v-if="$gate.isAdmin()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users</h3>

                <div class="card-tools">
                  <button class="btn btn-default btn-success" @click="newModal">Add New <i class="fa fa-user-plus"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Created At</th>
                    <th>Actions</th>
                  </tr>
                  
                  <tr v-for="user in users.data" :key="user.id">
                    <td>{{ user.id }}</td>
                    <td>{{ user.name | capitalize}}</td>
                    <td>{{ user.email}}</td>
                    <td>{{ user.type}}</td>
                    <td>{{ user.created_at | dateformat }}</td>
                    <td><a href="#" @click="editModal(user)"><i class="fa fa-edit"></i></a> | <a href="#" @click="deleteUser(user.id)"><i class="fa fa-trash text-danger"></i></a></td>
                  </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>
        <div v-if="!$gate.isAdmin()">
            <not-found></not-found>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="AddNewModal" tabindex="-1" role="dialog" aria-labelledby="AddNewModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="AddNewModalLabel">{{ editmode ? 'Edit' : 'Add New'}}</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <form @submit.prevent="editmode ? editUser() : createUser()" >
            <!-- Alert -->
            <alert-error :form="form"></alert-error>
                <div class="modal-body">
                    <div class="form-group">
                        <input v-model="form.name" type="text" name="name" placeholder="Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                        <input v-model="form.email" type="text" name="email" placeholder="Email Adress"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                    </div>
                    <div class="form-group">
                        <textarea v-model="form.bio" type="text" name="bio" placeholder="Short bio for the user(Optional)"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                        </textarea>
                        <has-error :form="form" field="bio"></has-error>
                    </div>
                    <div class="form-group">
                        <select v-model="form.type" type="text" name="type" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                            <option value="">Select User Role</option>
                            <option value="admin">Admin</option>
                            <option value="user">Standar User</option>
                            <option value="author">Author</option>
                        </select>
                        <has-error :form="form" field="type"></has-error>
                    </div>
                    <div class="form-group">
                        <input v-model="form.password" type="password" name="password" placeholder="Password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                        <has-error :form="form" field="password"></has-error>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                    <button type="submit" class="btn btn-primary">{{ editmode ? 'Edit' : 'Create'}}</button>
                </div>
            </form>
            </div>
        </div>
        </div>
    </div>
    
</template>

<script>
    export default {
        data () {
            return {
                editmode: false,
                users: {},
                // Create a new form instance
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',
                })
            }
        },
        methods:{
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
		    },
            editUser(id){
                //console.log('edit user function')
                this.$Progress.start();
                this.form.put('api/user/'+ this.form.id)
                .then(() => {
                    Fire.$emit('AfterCreated');
                    $("#AddNewModal").modal('hide');
                    Toast.fire({
                        type: 'success',
                        title: 'User updated successfully'
                    })
                    this.$Progress.finish();
                })
                .catch(() => {
                    this.$Progress.fail()
                    Toast.fire({
                        type: 'error',
                        title: 'Updated User failed'
                    })
                });
            },  
            editModal(user){
                this.editmode = true
                this.form.reset();
                $('#AddNewModal').modal('show');
                this.form.fill(user)
            },
            newModal(){
                this.editmode = false
                this.form.reset();
                $('#AddNewModal').modal('show');
            },
            deleteUser(id){
                Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    //send a delete requeste
                    if(result.value){
                        this.form.delete('api/user/'+id)
                        .then(() => {
                            Fire.$emit('AfterCreated');
                            Swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                            )
                        })
                        .catch(() => {
                            Swal.fire(
                            'Failed!',
                            'There was something wrong',
                            'warning'
                            )
                        });
                    }
                    
                })
            },
            loadUsers(){
                console.log(this.$gate.isAdmin())
                if(this.$gate.isAdmin()){
                    axios.get("api/user").then(({data}) => (this.users = data));
                }
            },
            createUser(){
                // Submit the form via a POST request.
                this.form.post('api/user')
                .then(()=>{
                    Fire.$emit('AfterCreated');
                    $("#AddNewModal").modal('hide');
                    Toast.fire({
                        type: 'success',
                        title: 'User created successfully'
                    })
                })
                .catch(()=>{
                    Toast.fire({
                        type: 'error',
                        title: 'Created User failed'
                    })
                });
                
            }
        },
        mounted() {
            console.log('Component mounted.')
            // Fetch initial results
            this.getResults();
            //progress
            this.$Progress.finish()
        },
        created () {
            this.loadUsers()
            //  [App.vue specific] When App.vue is first loaded start the progress bar
            this.$Progress.start()
            //  hook the progress bar to start before we move router-view
            this.$router.beforeEach((to, from, next) => {
            //  does the page we want to go to have a meta.progress object
            if (to.meta.progress !== undefined) {
                let meta = to.meta.progress
                // parse meta tags
                this.$Progress.parseMeta(meta)
            }
            //  start the progress bar
            this.$Progress.start()
            //  continue to next page
            next()
            })
            //  hook the progress bar to finish after we've finished moving router-view
            this.$router.afterEach((to, from) => {
            //  finish the progress bar
            this.$Progress.finish()
            })
            Fire.$on('AfterCreated', ()=>{
                this.loadUsers();
            })
            //setInterval(() => this.loadUsers(), 3000);
        }

    }
</script>
