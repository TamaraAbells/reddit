<template>
  <div class='text-center'>
    <v-dialog
      v-model='dialog'
      width='500'
    >
      <v-card>
        <v-card-title class='modal-title headline grey lighten-2' primary-title>
          Log In
        </v-card-title>

        <v-card-text>
          Log in to a Reddit account to join, vote, and comment on all your favourite Reddit content.
        </v-card-text>

        <v-form ref='form' v-model='valid' lazy-validation @submit.prevent='submit'>
          <v-text-field
            class='login-username'
            label='Username'
            name='username'
            v-model='username'
            :rules='usernameRules'
            color='grey'
            @keydown='error = null'
          >
          </v-text-field>
          <v-text-field
            class='login-password'
            label='Password'
            name='password'
            v-model='password'
            :rules='passwordRules'
            color='grey'
            @keydown='error = null'
            type='password'
            >
          </v-text-field>
          <input type='submit' style='display: none;' />
        </v-form>

        <v-card-text class='error-text' v-if='error'>
          {{ error }}
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <!-- <v-btn color='blue' small @click='dialog = false'>
            Close
          </v-btn> -->
          <v-btn color='primary' @click='submit' min-width='80' :loading='loading'>
            <template v-slot:loader>
              <span class='custom-loader'>
                <v-icon dark>mdi-cached</v-icon>
              </span>
            </template>
            Log In
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      valid: true,
      username: '',
      usernameRules: [
        v => !!v || 'Username is required',
        v => (v && v.length >= 3 && v.length <= 15) || 'Username must be between 3 and 15 characters'
      ],
      password: '',
      passwordRules: [
        v => !!v || 'Password is required',
        v => (v && v.length >= 6 && v.length <= 30) || 'Password must be between 6 and 30 characters'
      ],
      error: null,
      loading: false
    }
  },
  methods: {
    submit () {
      if (!this.$refs.form.validate()) {
        return
      }

      this.loading = true

      axios.post('/api/users/login', {
        username: this.username,
        password: this.password
      })
        .then(res => {
          if (!res.data.success) {
            this.error = res.data.message
            this.loading = false
            return
          }

          window.location.href = '/'
        })
    }
  },
  computed: {
    dialog: {
      get () {
        return this.$store.state.modal === 'log-in'
      },
      set () {
        this.$store.commit('setModal', null)
      }
    }
  }
}
</script>

<style scoped lang="scss">
  .v-card__text {
    padding: 20px 24px 10px !important;
  }
  form {
    padding: 0 24px 20px;
  }
  .error-text {
    padding-top: 0 !important;
    color: red !important;
    position: relative;
    top: -10px;
  }
  .log-in-button {
    margin-right: 20px;
    color: black !important;
  }
</style>
