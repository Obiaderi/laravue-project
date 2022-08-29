<script setup>
import axios from "axios";
import { ref, onMounted, reactive } from "vue";
import { Field, Form } from 'vee-validate';
import * as yup from 'yup';

const users = ref([]);

const editing = ref(false);

const formValues = ref();

const form = ref(null);


// Geting dynamic data from dataBase
const getUsers = () => {
    axios.get("/api/users").then((response) => {
        users.value = response.data;
    });
};

// Yup Validation
const createUserschema = yup.object({
    name: yup.string().required(),
    email: yup.string().email().required(),
    password: yup.string().required().min(8),
});

const editUserschema = yup.object({
    name: yup.string().required(),
    email: yup.string().email().required(),
    password: yup.string().when((password, schema) => {
        return password ? schema.required().min(8) : schema;
    })
});


const addUser = () => {
    form.value.resetForm()
    editing.value = false;
    $('#userFormModal').modal('show');
}



const createUser = (values, { resetForm, setErrors }) => {
    axios.post("/api/users", values)
        .then((response) => {
            users.value.unshift(response.data)

            $('#userFormModal').modal('hide');
            resetForm();
        })
        .catch((error) => {
            // setFieldError('email', error.response.data.errors.email[0]);
            if (error.response.data.errors) {
                setErrors(error.response.data.errors);
            }
        });

}


const editUser = (user) => {
    form.value.resetForm()
    editing.value = true;
    $('#userFormModal').modal('show');

    formValues.value = {
        id: user.id,
        name: user.name,
        email: user.email,
    };
}

const updateUser = (values, { resetForm, setErrors }) => {
    // console.log(values);

    axios.put("/api/users/" + formValues.value.id, values)
        .then((response) => {
            const index = users.value.findIndex(user => user.id === response.data.id);

            users.value[index] = response.data;

            $('#userFormModal').modal('hide');
            resetForm();
        }).catch((error) => {
            if (error.response.data.errors) {
                setErrors(error.response.data.errors);
            }
            console.log(error);
        });

}

const handleSubmit = (value, actions) => {
    // console.log(actions);
    if (editing.value) {
        updateUser(value, actions)
    } else {
        createUser(value, actions)
    }
}


// Posting dynamic data from dataBase
// const createUser = () => {
//     axios.post("/api/users", form).then((response) => {
//         users.value.unshift(response.data)
//         form.name = '';
//         form.email = '';
//         form.passwprd = '';

//         $('#userFormModal').modal('hide');
//     });
// };

// NB: We need to use the get User Func when the components mounts
onMounted(() => {
    getUsers();
});
</script>

<template>
    <div class="content-header">
        <div class="container-fluid">
            <div class="row mb-2">
                <div class="col-sm-6">
                    <h1 class="m-0">Users</h1>
                </div>
                <div class="col-sm-6">
                    <ol class="breadcrumb float-sm-right">
                        <li class="breadcrumb-item"><a href="#">Home</a></li>
                        <li class="breadcrumb-item active">Users</li>
                    </ol>
                </div>
            </div>
        </div>
    </div>

    <div class="content">
        <div class="container-fluid">
            <!-- Modal Button -->
            <button type="button" class="btn btn-primary mb-2" @click="addUser">
                Add New User
            </button>

            <div class="card">
                <div class="card-body">
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th style="width: 10px">#</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Registered Date</th>
                                <th>Role</th>
                                <th>Options</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(user, index) in users" :key="user.id">
                                <td>{{  index + 1  }}</td>
                                <td>{{  user.name  }}</td>
                                <td>{{  user.email  }}</td>
                                <td>-</td>
                                <td>-</td>
                                <td>
                                    <a href="#" @click.prevent="editUser(user)"><i class="fa fa-edit"></i></a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <!-- Modeal -->
    <div class="modal fade" id="userFormModal" data-backdrop="static" tabindex="-1" role="dialog"
        aria-labelledby="staticBackdropLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="staticBackdropLabel">
                        <span v-if="editing">Edit User</span>
                        <span v-else>Add New User</span>
                    </h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>

                <Form ref="form" @submit="handleSubmit" :validation-schema="editing ? editUserschema : createUserschema"
                    v-slot="{ errors }" :initial-values="formValues">
                    <div class="modal-body">
                        <div class="form-group">
                            <label for="name">Name</label>
                            <Field name="name" type="text" class="form-control" :class="{ 'is-invalid': errors.name }"
                                id="name" aria-describedby="nameHelp" placeholder="Enter full name" />
                            <span class="invalid-feedback">{{  errors.name  }}</span>
                        </div>

                        <div class="form-group">
                            <label for="email">Email</label>
                            <Field name="email" type="email" class="form-control"
                                :class="{ 'is-invalid': errors.email }" id="email" aria-describedby="nameHelp"
                                placeholder="Enter Email" />
                            <span class="invalid-feedback">{{  errors.email  }}</span>
                        </div>

                        <div class="form-group">
                            <label for="email">Password</label>
                            <Field name="password" type="password" class="form-control"
                                :class="{ 'is-invalid': errors.password }" id="password" aria-describedby="nameHelp"
                                placeholder="Enter password" />
                            <span class="invalid-feedback">{{  errors.password  }}</span>
                        </div>
                    </div>


                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">
                            Cancel
                        </button>
                        <button type="submit" class="btn btn-primary">Save</button>
                    </div>
                </Form>
            </div>
        </div>
    </div>
</template>
