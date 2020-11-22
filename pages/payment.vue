<template>
  <CtCard title="Pagos" width="100%" class="mx-auto">
    <template v-slot:rightTitleContent>
      <CtBtn color="white" :icon="['fas', 'plus']" @click="addPayment">
        Añadir
      </CtBtn>
    </template>
    <v-container fluid>
      <v-row dense>
        <v-col cols="12" v-if="serverMessage && serverMessage instanceof Object" class="error--text">
          <v-row v-for="(serverError, index) in serverMessage" :key="index">
            <v-col cols="12" v-html="serverError" />
          </v-row>
        </v-col>
        <v-col cols="12" v-else-if="serverMessage" v-html="serverMessage" class="error--text" />
      </v-row>
      <v-data-iterator
        :items="payments"
        :items-per-page.sync="itemsPerPage"
        :page="page"
        :search="search"
        :sort-by="sortBy.toLowerCase()"
        :sort-desc="sortDesc"
        hide-default-footer
      >
        <template v-slot:header>
          <v-row class="d-flex">
            <v-text-field
              v-model="search"
              flat
              outlined
              dense
              hide-details
              label="Search"
            ></v-text-field>
            <template v-if="$vuetify.breakpoint.mdAndUp">
              <v-spacer></v-spacer>
              <v-select
                v-model="sortBy"
                flat
                hide-details
                outlined
                dense
                :items="keys"
                item-text="description"
                item-value="key"
                label="Ordenar por"
              ></v-select>
              <v-spacer></v-spacer>
              <v-btn-toggle
                v-model="sortDesc"
                mandatory
              >
                <v-btn
                  large
                  depressed
                  color="primary"
                  :value="false"
                >
                  <v-icon>mdi-arrow-down</v-icon>
                </v-btn>
                <v-btn
                  large
                  depressed
                  color="primary"
                  :value="true"
                >
                  <v-icon>mdi-arrow-up</v-icon>
                </v-btn>
              </v-btn-toggle>
            </template>
          </v-row>
          <v-row dense>
            <v-col
              cols="12"
              class="mt-2">
              <v-row dense>
                <v-col
                  v-for="(key, index) in filteredKeys"
                  :key="index"
                  class="text-center"
                >
                  <template v-if="key.hint">
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on }">
                        <CtBtn type="icon" color="primary" :icon="['fas', 'question-circle']" v-on="on">
                          <span class="primary--text" v-html="key.description" />
                        </CtBtn>
                      </template>
                      <span v-html="key.hint" />
                    </v-tooltip>
                  </template>
                  <template v-else>
                    <CtBtn type="text" color="primary">
                      <span class="primary--text" v-html="key.description" />
                    </CtBtn>
                  </template>
                </v-col>
                <v-col cols="1" />
                <v-col cols="1" />
              </v-row>
            </v-col>
          </v-row>
        </template>

        <template v-slot:default="props">
          <v-row dense>
            <v-col
              v-for="paymentItem in props.items"
              :key=" paymentItem.id"
              cols="12"
              class="mt-2">
              <v-row dense>
                <v-col
                  v-for="(key, index) in filteredKeys"
                  :key="index"
                  class="text-center"
                >
                    <CtTextField v-if="paymentEdition[paymentItem.index] && paymentEdition[paymentItem.index].isEditing" @input="updatePaymentFromList(paymentItem.index)" :label="key.description" v-model="paymentItem[key.key]"/>
                    <span v-else :class="{ 'primary--text': sortBy === key }" v-html="paymentItem[key.key]" />
                </v-col>
                <v-col cols="1" class="text-right">
                  <CtBtn type="icon" color="primary" :icon="['fas', 'edit']" @click="paymentEdition[paymentItem.index].isEditing = ! paymentEdition[paymentItem.index].isEditing" />
                </v-col>
                <v-col cols="1">
                  <CtBtn type="icon" color="error" :icon="['fas', 'trash']" @click="removePayment(paymentItem.id)" />
                </v-col>
              </v-row>
            </v-col>
          </v-row>
        </template>

        <template v-slot:footer>
          <v-row dense>
            <v-col
              v-if="form"
              cols="12"
              class="mt-2">
              <v-row dense>
                <v-col
                  v-for="(key, index) in filteredKeys"
                  :key="index"
                  class="text-center"
                >
                  <CtTextField :label="key.description" v-model="payment[key.key]" @keyup.enter="save(null, payment)" />
                </v-col>
                <v-col cols="2" class="text-right">
                  <CtBtn type="icon" color="primary" :icon="['fas', 'save']" @click="save(null, payment)" />
                </v-col>
              </v-row>
            </v-col>
          </v-row>
          <v-row class="mt-2" align="center" justify="center">
            <span class="grey--text">Items por página</span>
            <v-menu offset-y>
              <template v-slot:activator="{ on }">
                <v-btn
                  dark
                  text
                  color="primary"
                  class="ml-2"
                  v-on="on"
                >
                  {{ itemsPerPage }}
                  <v-icon>mdi-chevron-down</v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-item
                  v-for="(number, index) in itemsPerPageArray"
                  :key="index"
                  @click="updateItemsPerPage(number)"
                >
                  <v-list-item-title>{{ number }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>

            <v-spacer></v-spacer>

            <span
              class="mr-4
            grey--text"
            >
            Página {{ page }} of {{ numberOfPages }}
          </span>
            <v-btn
              fab
              dark
              color="primary darken-3"
              class="mr-1"
              @click="formerPage"
            >
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>
            <v-btn
              fab
              dark
              color="primary darken-3"
              class="ml-1"
              @click="nextPage"
            >
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
          </v-row>
        </template>
      </v-data-iterator>
    </v-container>
  </CtCard>
</template>

<script>
import { mapActions } from 'vuex'

export default {
  middleware: 'authenticated',

  data() {
    return {
      payments: [],
      paymentEdition: [],
      payment: {
        description: '',
        creator_id: '1',
        status: '',
      },
      form: false,

      // DataIterator
      itemsPerPageArray: [4, 8, 12],
      search: '',
      filter: {},
      sortDesc: false,
      page: 1,
      itemsPerPage: 8,
      sortBy: 'description',
      keys: [
        {
          description: 'Descripción',
          hint: null,
          key: 'description',
        },
        {
          description: 'Status',
          hint: null,
          key: 'status',
        },
      ],
    }
  },

  computed: {
    serverMessage () {
      return this.$store.state.serverMessage.serverMessage
    },

    // DataIterator
    numberOfPages () {
      return Math.ceil(this.payments.length / this.itemsPerPage)
    },
    filteredKeys () {
      return this.keys.filter(key => key.key !== `to_views`) // TODO Filtered by views
    },
  },

  mounted() {
    this.fetch()
  },

  destroyed() {
    for (let i = 0; i < this.paymentEdition.length; i++) {
      this.paymentEdition[i].paymentNextEditionTimeFunction = null
    }
  },

  methods: {
    addPayment() {
      this.form = true
    },

    removePayment(paymentId) {
      if (paymentId !== null) {
        this.$axios.delete('/api/payment/' + paymentId)
          .then((response) => this.fetch())
          .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))
        return
      }

      return
    },

    updatePaymentFromList(paymentKey) {
      if (this.paymentEdition[paymentKey].paymentNextEditionTime) {
        this.$nextTick(() => {
           this.save(paymentKey, this.payments[paymentKey])
        })

        this.paymentEdition[paymentKey].paymentNextEditionTime = false
        this.paymentEdition[paymentKey].paymentNextEditionTimeFunction = setTimeout(() => {
          this.paymentEdition[paymentKey].paymentNextEditionTime = true
          if (this.paymentEdition[paymentKey].paymentNextEditionTimeQueued) {
            this.paymentEdition[paymentKey].paymentNextEditionTimeQueued = false
            this.updatePaymentFromList(paymentKey)
          }
        }, 2000)
      } else {
        this.paymentEdition[paymentKey].paymentNextEditionTimeQueued = true
      }
    },

    save(paymentItemIndex, payment) {
      if (paymentItemIndex !== null) {
        this.$axios.put('/api/payment/' + this.payments[paymentItemIndex].id, payment)
          .then(() => {})
          .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))
        return
      }

      this.$axios.post('/api/payment', payment)
        .then(() => {
          this.fetch()

          this.payment.description = ''
          this.payment.creator_id = '1'
          this.payment.status = ''

          this.form = false
        })
        .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))

      return
    },

    fetch() {
      this.$axios.get('/api/payment')
        .then((response) => {
          this.payments = []
          this.paymentEdition = []

          for (let i = 0; i < response.data.length; i++) {
            this.payments.push(response.data[i])
            this.payments[i].creator_id = this.payments[i].creator.id
            this.payments[i].index = i

            this.paymentEdition.push({
              id: this.payments[i].id,
              index: i,
              isEditing: false,
              paymentNextEditionTime: true,
              paymentNextEditionTimeQueued: false,
              paymentNextEditionTimeFunction: null,
            })
          }

        })
        .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))
    },

    // DataIterator
    nextPage () {
      if (this.page + 1 <= this.numberOfPages) this.page += 1
    },
    formerPage () {
      if (this.page - 1 >= 1) this.page -= 1
    },
    updateItemsPerPage (number) {
      this.itemsPerPage = number
    },

    ...mapActions({
      setServerMessage: 'serverMessage/setServerMessage',
    }),
  }
}
</script>
