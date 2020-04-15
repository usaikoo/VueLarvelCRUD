<template>
    <div class="container">
       <div class="row mt-5">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User </h3>

                <div class="card-tools">
                <button class="btn btn-success" @click="newModel"> Add New <i class="fa fa-user-plus fa-fw"></i></button>
                </div>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Created At</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for = " user in users " :key=" user.id ">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upText}}</td>
                      <td>{{user.created_at | createdDate }}</td>
                      <td>
                        <a href="#" @click="editModel(user)"> <i class="fa fa-edit"></i></a>
                        <a href="#" @click="deleteUser(user.id)"> <i class="fa fa-trash red"></i></a>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            
            <!-- /.card -->
          </div>
        </div>


        <!-- Modal -->
        <div class="modal fade" id="addNewScrollable" tabindex="-1" role="dialog" aria-labelledby="addNewScrollableTitle" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" v-show="editmode" id="addNewScrollableTitle">Update User Info</h5>
                <h5 class="modal-title" v-show="!editmode" id="addNewScrollableTitle">Add New</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
        <form @submit.prevent="editmode ? UpdateUser() :createUser()">
            <div class="modal-body">
              
                    <div class="form-group">
                        <input v-model="form.name" id="name" type="text" name="name"
                            placeholder = "Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                        <input v-model="form.email" id="email" type="text" name="email"
                            placeholder = "Email"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                    </div>
                    <div class="form-group">
                        <textarea v-model="form.bio" id="bio" name="bio" placeholder = "Short Bio for user (Optional)" class="form-control" :class="{ 'is-invalid': form.errors.has('name') }"> </textarea>
                        <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                        <select v-model="form.type" id="type"  name="type"
                            placeholder = "Short Bio for user (Optional)"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <option value=""> Select user role</option>
                            <option value="admin"> Admin</option>
                            <option value="user"> Standard user</option>
                            <option value="author"> Author</option>
                            
                            </select>
                        <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                        <input v-model="form.password" type="password" name="password" id="password"
                            placeholder = "Password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                        <has-error :form="form" field="password"></has-error>
                    </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
            </div>
        </form>
            </div>
        </div>
        </div>
    </div>
</template>

<script>

    export default {
        data(){
            return {
                editmode:false,
                users:{},
                form: new Form({
                    id: '',
                    name : '',
                   email: '',
                   password: '',
                   type:'',
                   bio:'',
                   photo:''
                })
            }
        },
        methods:{
            UpdateUser(){
            this.$Progress.start()

                // console.log('edit');
            this.form.put('/api/user/'+this.form.id)
            .then(()=>{
                $('#addNewScrollable').modal('hide');
                 Swal.fire(
                        'Updated!',
                        'User has been Updated.',
                        'success'
                        )
                        Fire.$emit('AfterActionDone');

            })
            .catch(()=>{
            this.$Progress.fail()

            })
            },
            editModel(user){
                this.editmode = true;
                this.form.reset();
                $('#addNewScrollable').modal('show');
                this.form.fill(user);

            },
            newModel(){
                this.editmode = false;
                this.form.reset();
                $('#addNewScrollable').modal('show');

            },

            deleteUser(id){
                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {
                        //send ajax request server 
                    if (result.value) {
                    this.form.delete('api/user/'+id)
                    .then(()=>{
                    
                        Swal.fire(
                        'Deleted!',
                        'User has been deleted.',
                        'success'
                        )
                        this.$Progress.finish();
                        Fire.$emit('AfterActionDone');
                    
                    }).catch(()=>{
                         Swal.fire(
                        'Fail!',
                        'There was someting wrong.',
                        'warning'
                        )
                    })};
                   
                    });
            },
            loadusers(){
                axios.get("api/user").then(({data})=>(this.users = data.data))
            },
            createUser(){
                this.$Progress.start()
                this.form.post('api/user')
                .then(()=>{
                Fire.$emit('AfterActionDone');
                $('#addNewScrollable').modal('hide');
               Toast.fire({
                    icon: 'success',
                    title: 'User Created successfully'
                    })
                this.$Progress.finish()
                })
                .catch(()=>{
                this.$Progress.fail();
                });
                

            }
        },
        mounted() {
            console.log('Component mounted.')
        },
        created(){
            this.loadusers();
            Fire.$on('AfterActionDone',()=>{
                this.loadusers();
            });
        }

    }
</script>
