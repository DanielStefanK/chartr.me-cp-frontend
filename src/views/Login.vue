<template>
  <v-container fluid fill-height>
    <v-layout align-center justify-center>
      <v-flex xs12 sm8 md4>
        <v-card class="elevation-12">
          <v-toolbar dark color="primary">
            <v-toolbar-title>Login</v-toolbar-title>
          </v-toolbar>
          <form @submit.prevent="login" :disabled="isLoading">
            <v-card-text>
              <v-text-field
                :disabled="isLoading"
                :error-messages="validationErrors('email',
                  {
                    required: 'Please enter an email',
                    email: 'Please enter a valid email',
                  })"
                v-model="email"
                prepend-icon="alternate_email"
                name="email"
                label="E-Mail"
                type="email"
              ></v-text-field>
              <v-text-field
                :disabled="isLoading"
                :error-messages="validationErrors('password',
                  {
                    required: 'Please enter a password',
                  })"
                v-model="password"
                id="password"
                prepend-icon="lock"
                name="password"
                label="Password"
                type="password"
              ></v-text-field>
            </v-card-text>
            <v-card-actions>New to chartr? Sign up&nbsp;
              <router-link to="signup">here</router-link>
              <v-spacer></v-spacer>
              <v-btn color="primary" :loading="isLoading" type="submit">Login</v-btn>
            </v-card-actions>
          </form>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import gql from 'graphql-tag';
import { required, email } from 'vuelidate/lib/validators';

import { onLogin } from '../plugins/vue-apollo.js';

import { EventBus } from '../utils/eventBus';
import validationMixins from '@/mixins/validationHelper';

export default {
  data() {
    return {
      email: '',
      password: '',
      isLoading: false,
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
    login() {
      if (!this.$v.$invalid) {
        this.isLoading = true;
        this.$apollo
          .mutate({
            mutation: gql`
              mutation login($email: String!, $password: String!) {
                login(email: $email, password: $password) {
                  token
                  me {
                    name
                    email
                    id
                    permissions
                    company {
                      id
                    }
                  }
                }
              }
            `,

            variables: {
              email: this.email,
              password: this.password,
            },
          })
          .then(async ({ data }) => {
            await onLogin(
              this.$apollo.provider.defaultClient,
              data.login.token,
            );
            EventBus.$emit('snackbar', {
              text: 'Successfully logged in',
              color: 'success',
            });
            await this.$apollo.provider.defaultClient.writeQuery({
              query: require('@/graphql/MeQuery.gql'),
              data: {
                me: data.login.me,
              },
            });
            this.$router.push({ name: 'dashboard' });
          })
          .catch(error => {
            EventBus.$emit('snackbar', {
              text: error.message,
              color: 'error',
            });
          })
          .then(() => {
            this.isLoading = false;
          });
      } else {
        this.$v.$touch();
      }
    },
  },
};
</script>
