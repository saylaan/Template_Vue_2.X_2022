<template>
  <v-layout v-on:keyup.enter="saveSettings">
    <v-flex xs12 md10>

  <panel v-if="isUserLoggedIn" title="Setting User">
    <form class="mt-5">
    <!-- <v-text-field
      outline
      clearable
      v-if="admin"
      label="email"
      v-model="adminview.email"
      :rules="[required]"
    >
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
    </v-text-field> -->
          <!-- <template v-slot:append>
        <v-fade-transition leave-absolute>
          <v-icon>domain</v-icon>
        </v-fade-transition>
      </template>
    </v-text-field>
    <v-text-field
      class="padding-input"
      outline
      clearable
      v-if="admin"
      label="First name"
      v-model="adminview.firstname"
      :rules="[required]"
    >
      <template v-slot:append>
        <v-fade-transition leave-absolute>
          <v-icon>account_box</v-icon>
        </v-fade-transition>
      </template>
    </v-text-field>
    <v-text-field
      class="padding-input"
      outline
      clearable
      v-if="admin"
      label="Last name"
      v-model="adminview.lastname"
      :rules="[required]"
    >
      <template v-slot:append>
        <v-fade-transition leave-absolute>
          <v-icon>account_box</v-icon>
        </v-fade-transition>
      </template>
    </v-text-field> -->
    <v-text-field
      label="Current password"
      v-model="oldPassword"
      type="password"
      :rules="[required]"
      outline
      clearable
    >
      <template v-slot:append>
        <v-fade-transition leave-absolute>
          <v-icon>https</v-icon>
        </v-fade-transition>
      </template>
    </v-text-field>
    <v-text-field
      label="New password"
      v-model="newPassword"
      type="password"
      outline
      clearable
    >
      <template v-slot:prepend>
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-icon v-on="on">help</v-icon>
          </template>
          Your password must contain at least 8 characters
        </v-tooltip>
      </template>
      <template v-slot:append>
        <v-fade-transition leave-absolute>
          <v-icon>https</v-icon>
        </v-fade-transition>
      </template>
    </v-text-field>
    <v-text-field
      outline
      clearable
      label="Confirm your new password"
      v-model="confirmPassword"
      type="password"
    >
      <template v-slot:append>
        <v-fade-transition leave-absolute>
          <v-icon>https</v-icon>
        </v-fade-transition>
      </template>
    </v-text-field>
    <v-layout column class="mb-4" justify-center align-center>
      <span class="danger-alert">{{error}}</span>
      <v-layout class="mt-2" justify-center align-center row>
        <v-btn elevation-24 large @click="saveSettings()" class="grey darken-1 font-weight-bold">Save</v-btn>
        <v-btn elevation-24 large @click="cancelSettings()" class="grey darken-1 font-weight-bold ml-1">Cancel</v-btn>
        <v-icon @click="undoSettings()">refresh</v-icon>
      </v-layout>
    </v-layout>
    </form>
  </panel>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapState } from "vuex";
import UserService from "@/services/User/UserService";
import crypto from 'crypto'

export default {
  data() {
    return {
      userview: {
        hash: null,
        salt: null,
        password: null
      },
      adminview: {
        hash: null,
        salt: null,
        password: null
      },
      oldPassword: null,
      newPassword: null,
      confirmPassword: null,
      error: null,
      required: value => !!value || "Required."
    };
  },
  computed: {
    ...mapState(["isUserLoggedIn", "user", "admin", "manager"])
  },
  methods: {
    async saveSettings() {
      this.error = null;
      if (this.admin) {
        const areAllFieldsFilledIn = Object.keys(this.adminview).every(
          key => {
            if (key === 'hash' || key === 'salt' || key === 'password') {
              return (true)
            }
            return (!!this.adminview[key])
          }
        );
        if (!areAllFieldsFilledIn) {
          this.error = "Please fill in all the required fields.";
          return;
        }
        if (this.confirmPassword && this.newPassword) {
          if (this.confirmPassword !== this.newPassword) {
            this.error = "The new passwords don't match"
            return;
          }
        }
        try {
          const hash = crypto.pbkdf2Sync(this.oldPassword, this.user.salt, 1000, 64, `sha512`).toString(`hex`)
          const isPasswordValid = this.user.hash === hash
          if (isPasswordValid) {
            this.adminview.salt = crypto.randomBytes(16).toString(`hex`)
            this.adminview.hash = crypto.pbkdf2Sync(this.newPassword, this.adminview.salt,
              1000, 64, `sha512`).toString(`hex`)
            await UserService.put(this.adminview);
            this.$router.push({ name: "users" });
          } else {
            this.error = "You entered the wrong password"
          }
        } catch (err) {
          console.log(err);
        }
      } else {
        const areAllFieldsFilledIn = Object.keys(this.userview).every(
          key => {
            if (key === 'hash' || key === 'salt' || key === 'admin' || key === 'email' || key === 'password') {
              return (true)
            }
            return (!!this.userview[key])
          }
        );
        if (!areAllFieldsFilledIn) {
          this.error = "Please fill in all the required fields.";
          return;
        }
        if (this.confirmPassword && this.newPassword) {
          if (this.confirmPassword !== this.newPassword) {
            this.error = "The new passwords don't match"
            return;
          }
        }
        try {
          const hash = crypto.pbkdf2Sync(this.oldPassword, this.user.salt, 1000, 64, `sha512`).toString(`hex`)
          const isPasswordValid = this.user.hash === hash
          if (isPasswordValid) {
            this.userview.salt = crypto.randomBytes(16).toString(`hex`)
            this.userview.hash = crypto.pbkdf2Sync(this.newPassword, this.userview.salt,
              1000, 64, `sha512`).toString(`hex`)
            await UserService.put(this.userview);
            this.$router.push({ name: "dashboard" });
          } else {
            this.error = "You entered the wrong password"
          }
        } catch (err) {
          console.log(err);
        }
      }
    },
    async undoSettings() {
      try {
        this.error = null
        if (!this.admin) {
          this.userview = (await UserService.get(this.user.id)).data;
          this.oldPassword = null
          this.newPassword = null
          this.confirmPassword = null
        } else {
          this.adminview = (await UserService.get(this.user.id)).data;
        }
      } catch (err) {
        console.log(err);
      }
    },
    async cancelSettings() {
      this.$router.push({name: "dashboard"})
    }
  },
  async mounted() {
    try {
      if (!this.admin) {
        this.userview = (await UserService.get(this.user.id)).data;
      } else {
        this.adminview = (await UserService.get(this.user.id)).data;
      }
    } catch (err) {
      console.log(err);
    }
  }
};
</script>

<style scoped>

</style>
