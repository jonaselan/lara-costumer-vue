<template>
    <div class="customer-new">
        <form @submit.prevent="add">
            <table class="table">
                <tr>
                    <th>Name</th>
                    <td>
                        <input type="text" class="form-control" v-model="customer.name" placeholder="Customer Name"/>
                    </td>
                </tr>
                <tr>
                    <th>Email</th>
                    <td>
                        <input type="email" class="form-control" v-model="customer.email" placeholder="Customer Email"/>
                    </td>
                </tr>
                <tr>
                    <th>Phone</th>
                    <td>
                        <input type="text" class="form-control" v-model="customer.phone" placeholder="Customer Phone"/>
                    </td>
                </tr>
                <tr>
                    <th>Website</th>
                    <td>
                        <input type="text" class="form-control" v-model="customer.website" placeholder="Customer Website"/>
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <image-uploader></image-uploader>
                    </td>
                </tr>
                <tr>
                    <td>
                        <router-link to="/customers" class="btn">Cancel</router-link>
                    </td>
                    <td class="text-right">
                        <input type="submit" value="Create" class="btn btn-primary">
                    </td>
                </tr>
            </table>

        </form>
        <div class="errors" v-if="errors">
            <ul>
                <li v-for="(fieldsError, fieldName) in errors" :key="fieldName">
                    <strong>{{ fieldName }}</strong> {{ fieldsError.join('\n') }}
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
    import validate from 'validate.js';
    import ImageUploader from "./ImageUploader";

    export default {
        name: 'create',
        components: {
            ImageUploader
        },
        data() {
            return {
                customer: {
                    name: '',
                    email: '',
                    phone: '',
                    website: ''
                },
                errors: null
            };
        },
        computed: {
            currentUser() {
                return this.$store.getters.currentUser;
            }
        },
        methods: {
            add() {
                this.errors = null;
                const constraints = this.getConstraints();
                const errors = validate(this.$data.customer, constraints);
                if(errors) {
                    this.errors = errors;
                    return;
                }

                const formData = new FormData();

                this.$store.getters.files.forEach(file => {
                    formData.append('images[]', file, file.name);
                });

                formData.append('email', this.$data.customer.email);
                formData.append('name', this.$data.customer.name);
                formData.append('phone', this.$data.customer.phone);
                formData.append('website', this.$data.customer.website);

                axios.post('/api/customers', formData)
                    .then((response) => {
                        this.$router.push('/customers');
                    });
            },
            getConstraints() {
                return {
                    name: {
                        presence: true,
                        length: {
                            minimum: 3,
                            message: 'Must be at least 3 characters long'
                        }
                    },
                    email: {
                        presence: true,
                        email: true
                    },
                    phone: {
                        presence: true,
                        numericality: true,
                        length: {
                            minimum: 10,
                            message: 'Must be at least 10 digits long'
                        }
                    },
                    website: {
                        presence: true,
                        url: true
                    }
                };
            }
        }
    }
</script>

<style>
  .errors {
      background: lightcoral;
      border-radius:5px;
      padding: 21px 0 2px 0;
  }
</style>
