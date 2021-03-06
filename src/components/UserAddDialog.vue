<template>
  <v-dialog v-model="dialog" width="500">
    <div slot="activator" v-if="this.$slots.default">
      <slot></slot>
    </div>
    <div slot="activator" v-else>
      <v-btn flat icon>
        <v-icon>add</v-icon>
      </v-btn>Add a user to your company
    </div>

    <v-card>
      <v-card-title class="headline grey lighten-2" primary-title>Add User</v-card-title>
      <form @submit.prevent="add">
        <v-card-text>
          <v-text-field v-model="name" prepend-icon="person" name="name" label="Name" type="text"></v-text-field>
          <v-text-field v-model="email" prepend-icon="alternate_email" label="E-Mail" type="email"></v-text-field>
          <v-text-field
            v-model="password"
            id="password"
            prepend-icon="lock"
            label="Password"
            type="password"
          ></v-text-field>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <span v-if="error" class="red--text">A user with this email already exists</span>
          <v-spacer></v-spacer>
          <v-btn color="primary" :loading="isLoading" flat type="submit">add</v-btn>
        </v-card-actions>
      </form>
    </v-card>
  </v-dialog>
</template>

<script>
import gql from 'graphql-tag';
import { required, email } from 'vuelidate/lib/validators';

import { EventBus } from '../utils/eventBus';
import validationMixins from '@/mixins/validationHelper';

export default {
  props: {
    users: {
      type: Array,
      required: false,
    },
  },

  data() {
    return {
      dialog: false,
      name: '',
      email: '',
      password: '',
      isLoading: false,
      error: false,
    };
  },

  mixins: [validationMixins],

  validations: {
    email: {
      required,
      email,
    },
    password: {
      required,
    },
  },

  methods: {
    async add() {
      this.isLoading = true;
      this.error = false;
      if (this.users) {
        const user = this.users.find(user => {
          return user.email === this.email;
        });

        if (user) {
          this.error = true;
          this.isLoading = false;
          return;
        }
      }

      const result = await this.$apollo.query({
        query: gql`
          query checkUser($email: String!) {
            checkUser(email: $email)
          }
        `,
        fetchPolicy: 'network-only',
        variables: {
          email: this.email,
        },
      });

      if (result.data.checkUser) {
        this.error = true;
        this.isLoading = false;
        EventBus.$emit('snackbar', {
          text: 'A user with that name already exists',
          color: 'error',
        });
        return;
      }

      this.$emit('add', {
        name: this.name,
        password: this.password,
        email: this.email,
      });

      this.name = '';
      this.email = '';
      this.password = '';
      this.dialog = false;
      this.error = false;
      this.isLoading = false;
      EventBus.$emit('snackbar', {
        text: 'The user was added',
        color: 'success',
      });
    },
  },
};
</script>
