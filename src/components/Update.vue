<template>
<div>
    <Header />
    <h1 class="text-center my-8 success--text">Hello {{ `${user.toUpperCase()}` }}, Welcome on Update page.</h1>

    <v-row justify="center">
        <v-col cols="12" md="6" sm="8">
            <div class=" v-card--material pa-5 elevation-10 v-card v-sheet v-card--material--has-heading">
                <div class="d-flex grow flex-wrap">
                    <div class="v-card--material__heading mb-n5 elevation-6 success pa-5 rounded-lg" style="width: 100%">
                        <div class="text-h4 text-center white--text font-weight-bold">
                            Update Student
                        </div>
                    </div>
                </div>
                <v-card-text class="mt-3">
                    <ValidationObserver ref="observer" v-slot="{ handleSubmit }">
                        <v-form @submit.prevent="handleSubmit(onSubmit)">
                            <v-row>
                                <v-col cols="12" class="mt-3">

                                    <ValidationProvider v-slot="{ errors }" name="Name" rules="required">
                                        <v-text-field color="success" maxlength="100" v-model="student.name" :error-messages="errors" label="Enter Name" type="text" :counter="100" required>
                                        </v-text-field>
                                    </ValidationProvider>

                                    <ValidationProvider v-slot="{ errors }" name="Address" rules="required">
                                        <v-text-field color="success" maxlength="100" v-model="student.address" :error-messages="errors" label="Enter Address" :counter="100" type="text" required>
                                        </v-text-field>
                                    </ValidationProvider>

                                    <ValidationProvider v-slot="{ errors }" name="Contact" rules="required|digits:10|customPhoneNumber">
                                        <v-text-field color="success" type="phone" maxlength="10" v-model="student.contact" :counter="10" :error-messages="errors" label="Enter Contact" required></v-text-field>
                                    </ValidationProvider>

                                </v-col>
                                <v-col cols="12">
                                    <v-btn color="success" outlined block type="submit">Update Student</v-btn>
                                </v-col>
                            </v-row>
                        </v-form>
                    </ValidationObserver>
                </v-card-text>
            </div>
        </v-col>
    </v-row>
</div>
</template>

<script>
import axios from 'axios'
import { required, digits } from "vee-validate/dist/rules";
import { extend, ValidationObserver, ValidationProvider, setInteractionMode } from "vee-validate";
import Header from './Header.vue'
import { mapActions } from 'vuex'

setInteractionMode("eager");

extend("required", {
    ...required,
    message: "{_field_} can not be empty",
});

extend("digits", {
    ...digits,
    message: "{_field_} needs to be {length} digits",
});

var errorPhoneNumber = "requires 10 digits";
extend("customPhoneNumber", {
    message: (field) => `The ${field} ${errorPhoneNumber}`,
    validate: (value) => {
        var mustContainTheseNumber = /^[1-9]{1}[0-9]{9}$/;
        var containsRequiredNumber = mustContainTheseNumber.test(value);
        if (containsRequiredNumber) {
            return true;
        } else {
            errorPhoneNumber = "is incorrect.";
            return false;
        }
    },
});

setInteractionMode("eager");
export default {
    components: {
        Header,
        ValidationObserver,
        ValidationProvider
    },
    data() {
        return {
            user: '',
            student: {
                name: '',
                address: '',
                contact: ''
            }

        }
    },
    methods: {

        ...mapActions(['updateStudent']),

        async onSubmit() {
            this.$refs.observer.validate();

            this.updateStudent({
                name: this.student.name,
                address: this.student.address,
                contact: this.student.contact
            });
            
            this.clear();
        },
        clear() {
            this.name = "";
            this.address = "";
            this.contact = "";
            this.$refs.observer.reset();
        },
    },
    async mounted() {

        const response = await axios.get("http://localhost:3000/student/" + this.$route.params.id);

        this.student = response.data;

        if (!(localStorage.getItem('user-info'))) {
            this.$router.push({ name: 'Signup' })
        } else {
            const user_name = JSON.parse(localStorage.getItem('user-info'))
            this.user = user_name.first_name + ' ' + user_name.last_name;
        }
    }
}
</script>
