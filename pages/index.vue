<template>
  <v-row dense>
    <v-col cols="12">
      <v-row dense>
        <v-col cols="12" md="8">
          <br><br><br><br>
          <v-row>
            <v-col>
              <h1 :class="{ 'display-2': true, 'font-weight-bold': true, 'mt-12': $vuetify.breakpoint.mdAndUp }">
                <v-row>MI COMUNIDAD</v-row>
              </h1>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <p>Disfruta de sorteos solo con interactuar conmigo.</p>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <CtBtn type="empty" color="primary" class="pt-7 pb-7 px-12" to="/registro">
                Empezar
              </CtBtn>
            </v-col>
          </v-row>
        </v-col>
        <v-col cols="4" v-if="$vuetify.breakpoint.mdAndUp">
          <br><br><br><br>
          <v-img src="/img/payment_status.png" :width="$vuetify.breakpoint.lgAndUp ? '80%' : '100%'" class="float-right" title="Payment status" alt="Payment status" />
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12">
          <br><br><br><br><br>
        </v-col>
      </v-row>
      <v-row dense>
        <v-col
          v-for="(raffleItem, index) in raffles"
          :key="index"
          cols="4">
          <v-row dense>
            <v-col cols="12" class="text-center">
              <v-card>
                <img
                  :src="$axios.defaults.baseURL + raffleItem.photo"
                  width="100%"
                  title="Foto"
                  alt="Foto"
                  :id="'photoUpload' + raffleItem.id"
                />
                <v-card-title class="title" v-html="raffleItem.description" />
              </v-card>

            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row dense>
        <v-col cols="12">
          <v-row dense>
            <v-spacer />
            <v-col>
              <br><br><br><br>
              <CtBtn block type="empty" color="primary" class="pt-7 pb-7 px-12" to="/registro">
                Empezar
              </CtBtn>
              <br><br>
            </v-col>
            <v-spacer />
          </v-row>
        </v-col>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
import { mapGetters } from 'vuex'
export default {

  data() {
    return {
      raffles: [],
    }
  },

  mounted() {
    // Try to get token cookie if token isn't set
    if (!this.$store.state.user.token || !this.getUser()) {
      let token = document.cookie.match(new RegExp('(^| )token=([^;]+)'))
      if (token) {
        this.$store.dispatch('user/setToken', token[2])

        let user = document.cookie.match(new RegExp('(^| )user=([^;]+)'))
        if (user) {
          this.$store.commit('user/updateUser', JSON.parse(decodeURI(user[2])))
        } else {
          console.log('test')
          try {
            this.$store.dispatch('user/fetchUser')
          } catch (error) {}
        }
      }
    }

    this.fetch();
  },

  methods: {

    fetch() {
      this.$axios.get('/api/raffleSummary')
        .then((response) => {
          this.raffles = response.data
        })
        .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))
    },

    ...mapGetters({
      getUser: 'user/getUser',
    }),
  },
}
</script>
