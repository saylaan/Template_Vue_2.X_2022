<template>
  <v-layout class="mt-5" v-if="isUserLoggedIn && admin" v-on:keyup.enter="create({name: 'dashboard'})" justify-center>
    <v-flex xs6>
      <panel title="Create User">
                  <v-text-field label="Last name" type="lastname" v-model="userview.lastname"
          outline clearable>
            <template v-slot:prepend>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-icon v-on="on">help</v-icon>
                </template>
                Your last name must be valid
              </v-tooltip>
            </template>
            <template v-slot:append>
              <v-fade-transition leave-absolute>
                <v-icon>email</v-icon>
              </v-fade-transition>
            </template>
          </v-text-field>
        <br>
                  <v-text-field label="First name" type="firstname" v-model="userview.firstname"
          outline clearable>
            <template v-slot:prepend>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-icon v-on="on">help</v-icon>
                </template>
                Your firstname must be valid
              </v-tooltip>
            </template>
            <template v-slot:append>
              <v-fade-transition leave-absolute>
                <v-icon>email</v-icon>
              </v-fade-transition>
            </template>
          </v-text-field>
        <br>
          <v-text-field label="Email" type="name" v-model="userview.email"
          outline clearable>
            <template v-slot:prepend>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-icon v-on="on">help</v-icon>
                </template>
                Your email must be valid
              </v-tooltip>
            </template>
            <template v-slot:append>
              <v-fade-transition leave-absolute>
                <v-icon>email</v-icon>
              </v-fade-transition>
            </template>
          </v-text-field>
        <br>
          <v-text-field label="Confirm email" type="name" v-model="confirmEmail"
          outline clearable>
            <template v-slot:prepend>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-icon v-on="on">help</v-icon>
                </template>
                Your email must be the same
              </v-tooltip>
            </template>
            <template v-slot:append>
              <v-fade-transition leave-absolute>
                <v-icon>email</v-icon>
              </v-fade-transition>
            </template>
          </v-text-field>
          <br>
        <v-switch v-if="isManager() == false" large color="red" v-model="userview.admin" label="Set Admin">
          <template v-slot:prepend>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-icon v-on="on">help</v-icon>
                </template>
                Its for seeting the user to admin
              </v-tooltip>
            </template>
            <template v-slot:append>
              <v-fade-transition leave-absolute>
                <v-icon>supervised_user_circle</v-icon>
              </v-fade-transition>
            </template>
        </v-switch>
        <br>
        <v-switch v-if="isAdmin() == false" large color="red" v-model="userview.manager" label="Set Manager">
          <template v-slot:prepend>
              <v-tooltip bottom>
                <template v-slot:activator="{ on }">
                  <v-icon v-on="on">help</v-icon>
                </template>
                Its for seeting the user to manager
              </v-tooltip>
            </template>
            <template v-slot:append>
              <v-fade-transition leave-absolute>
                <v-icon>supervised_user_circle</v-icon>
              </v-fade-transition>
            </template>
        </v-switch>
        <br>
        <v-layout column align-center justify-center class="mb-4">
        <span class="danger-alert">{{error}}</span>
        <v-layout class="mt-2" row justify-center align-center>
          <v-btn elevation-24 large class="grey darken-1 mb-4 font-weight-bold" 
          @click="create({name: 'dashboard'})">
          Create</v-btn>
          <v-btn elevation-24 large class="grey darken-1 ml-1 mb-4 font-weight-bold" 
          @click="cancel()">
          Cancel</v-btn>
        </v-layout>
        </v-layout>
      </panel>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapState } from "vuex";
import UserService from "@/services/User/UserService"

export default {
  data() {
    return {
      max: 50,
      confirmEmail: null,
      userview: {
        lastname: null,
        firstname: null,
        email: null,
        manager: false,
        admin: false,
        password: "ChangeMe123"
      },
      error: null,
      required: value => !!value || "Required."
    };
  },
  computed: {
    ...mapState(["isUserLoggedIn", "user", "admin", "manager"])
  },
  methods: {
    isAdmin() {
      return (this.userview.admin)
    },
    isManager() {
      return (this.userview.manager)
    },
    async create(route) {
      this.error = null;
      const areAllFieldsFilledIn = Object.keys(this.userview).every(
        key => {
          if (key === 'admin' || key === 'manager' || key === 'password') {
            return (true)
          }
          return (!!this.userview[key])
        }
      );
      if (!areAllFieldsFilledIn || !this.confirmEmail) {
        this.error = "Please fill in all the required fields.";
        return;
      }
      if (this.confirmEmail !== this.userview.email) {
        this.error = "The emails don't match"
        return;
      }
      try {
        await UserService.post(this.userview);
        this.$router.push(route);
      } catch (err) {
        console.log(err);
      }
    },
    async cancel() {
      this.$router.push({name: 'users'})
    }
  }
};
</script>

<style scoped>
.danger-alert {
  color: red;
}
.padding-input {
  margin: auto;
  /* margin-left: 32px; */
}
</style>
