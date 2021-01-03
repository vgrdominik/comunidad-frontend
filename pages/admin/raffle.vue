<template>
  <CtCard title="Sorteos" width="100%" class="mx-auto">
    <template v-slot:rightTitleContent>
      <CtBtn color="white" :icon="['fas', 'plus']" @click="addRaffle">
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
        :items="raffles"
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
              v-for="raffleItem in props.items"
              :key=" raffleItem.id"
              cols="12"
              class="mt-2">
              <v-row dense>
                <v-col
                  v-for="(key, index) in filteredKeys"
                  :key="index"
                  class="text-center"
                >
                    <CtTextField v-if="raffleEdition[raffleItem.index] && raffleEdition[raffleItem.index].isEditing" @input="updateRaffleFromList(raffleItem.index)" :label="key.description" v-model="raffleItem[key.key]"/>
                    <span v-else :class="{ 'primary--text': sortBy === key }" v-html="raffleItem[key.key]" />
                </v-col>
                <v-col
                  class="text-center"
                >
                  <v-file-input
                    accept="image/*"
                    placeholder="Escoje una foto"
                    label="Foto"
                    :id="'photoUploadInput' + raffleItem.index"
                    @change="updatePhoto('photoUploadInput' + raffleItem.index, 'photoUpload' + raffleItem.index, false, raffleItem.index)"
                    v-model="raffleItem.photo"
                    v-if="raffleEdition[raffleItem.index] && raffleEdition[raffleItem.index].isEditing" />
                </v-col>
                <v-col
                  class="text-center"
                >
                  <img
                    :src="$axios.defaults.baseURL + raffleItem.photo"
                    width="180px"
                    title="Foto"
                    alt="Foto"
                    :id="'photoUpload' + raffleItem.index"
                  />
                </v-col>
                <v-col cols="1" class="text-right">
                  <CtBtn type="icon" color="primary" :icon="['fas', 'edit']" @click="raffleEdition[raffleItem.index].isEditing = ! raffleEdition[raffleItem.index].isEditing" />
                </v-col>
                <v-col cols="1">
                  <CtBtn type="icon" color="error" :icon="['fas', 'trash']" @click="removeRaffle(raffleItem.id)" />
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
                  <CtTextField :label="key.description" v-model="raffle[key.key]" @keyup.enter="save(null, raffle)" />
                </v-col>
                <v-col
                  class="text-center"
                >
                  <v-file-input
                    accept="image/*"
                    placeholder="Escoje una foto"
                    label="Foto"
                    id="photoUploadInput"
                    @change="updatePhoto('photoUploadInput', 'photoUpload', true)"
                    v-model="raffle['photo']" />
                </v-col>
                <v-col
                  class="text-center"
                >
                  <img
                    src="/img/payment_status_logo_180.png"
                    width="180px"
                    title="Foto"
                    alt="Foto"
                    id="photoUpload"
                  />
                </v-col>
                <v-col cols="2" class="text-right">
                  <CtBtn type="icon" color="primary" :icon="['fas', 'save']" @click="save(null, raffle)" />
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
import Compressor from 'compressorjs'

export default {
  middleware: 'authenticated',

  data() {
    return {
      raffles: [],
      raffleEdition: [],
      raffle: {
        description: '',
        creator_id: '1',
        photo: '',
        photoFile: '',
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
      ],
    }
  },

  computed: {
    serverMessage () {
      return this.$store.state.serverMessage.serverMessage
    },

    // DataIterator
    numberOfPages () {
      return Math.ceil(this.raffles.length / this.itemsPerPage)
    },
    filteredKeys () {
      return this.keys.filter(key => key.key !== `to_views`) // TODO Filtered by views
    },
  },

  mounted() {
    this.fetch()
  },

  destroyed() {
    for (let i = 0; i < this.raffleEdition.length; i++) {
      this.raffleEdition[i].raffleNextEditionTimeFunction = null
    }
  },

  methods: {
    addRaffle() {
      this.form = true
    },

    removeRaffle(raffleId) {
      if (raffleId !== null) {
        this.$axios.delete('/api/raffle/' + raffleId)
          .then((response) => this.fetch())
          .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))
        return
      }

      return
    },

    updateRaffleFromList(raffleKey) {
      if (this.raffleEdition[raffleKey].raffleNextEditionTime) {
        this.$nextTick(() => {
           this.save(raffleKey, this.raffles[raffleKey])
        })

        this.raffleEdition[raffleKey].raffleNextEditionTime = false
        this.raffleEdition[raffleKey].raffleNextEditionTimeFunction = setTimeout(() => {
          this.raffleEdition[raffleKey].raffleNextEditionTime = true
          if (this.raffleEdition[raffleKey].raffleNextEditionTimeQueued) {
            this.raffleEdition[raffleKey].raffleNextEditionTimeQueued = false
            this.updateRaffleFromList(raffleKey)
          }
        }, 2000)
      } else {
        this.raffleEdition[raffleKey].raffleNextEditionTimeQueued = true
      }
    },

    save(raffleItemIndex, raffle) {
      if (raffleItemIndex !== null) {
        this.$axios.put('/api/raffle/' + this.raffles[raffleItemIndex].id, raffle)
          .then((raffleResult) => {
            if (raffle.photo && raffle.photoFile) {
              new Compressor(raffle.photoFile, {
                quality: 0.7,
                maxWidth: 650,
                success: (result) => {
                  const formData = new FormData()

                  formData.append('select_file', result, result.name)
                  formData.append('description', raffle.description)
                  formData.append('creator_id', raffle.creator_id)

                  this.$axios.post('/api/raffleUploadPhoto/' + raffleResult.data.id, formData)
                    .then(() => {})
                    .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))

                  return
                },
                error(err) {
                  console.log(err.message);
                },
              });
            }
          })
          .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))

      } else {
        this.$axios.post('/api/raffle', raffle)
          .then((raffleResult) => {
            if (raffle.photo && raffle.photoFile) {
              new Compressor(raffle.photoFile, {
                quality: 0.7,
                maxWidth: 650,
                success: (result) => {
                  const formData = new FormData()

                  formData.append('select_file', result, result.name)
                  formData.append('description', raffle.description)
                  formData.append('creator_id', raffle.creator_id)

                  this.$axios.post('/api/raffleUploadPhoto/' + raffleResult.data.id, formData)
                    .then(() => {
                      this.fetch()
                    })
                    .catch((error) => {
                      (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.')
                      this.fetch()
                    })

                  return
                },
                error(err) {
                  this.fetch()
                  console.log(err.message);
                },
              });
            }

            this.raffle.description = ''
            this.raffle.creator_id = '1'

            this.form = false
          })
          .catch((error) => (error.response.data.message) ? (error.response.data.message === 'The given data was invalid.' && error.response.data.errors) ? this.setServerMessage(error.response.data.errors) : this.setServerMessage(error.response.data.message) : this.setServerMessage('Error.'))
      }

      return
    },

    fetch() {
      this.$axios.get('/api/raffle')
        .then((response) => {
          this.raffles = []
          this.raffleEdition = []

          for (let i = 0; i < response.data.length; i++) {
            this.raffles.push(response.data[i])
            this.raffles[i].creator_id = this.raffles[i].creator.id
            this.raffles[i].photo = this.raffles[i].photo
            this.raffles[i].photoFile = ''
            this.raffles[i].index = i

            this.raffleEdition.push({
              id: this.raffles[i].id,
              index: i,
              isEditing: false,
              raffleNextEditionTime: true,
              raffleNextEditionTimeQueued: false,
              raffleNextEditionTimeFunction: null,
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

    readURL(input, id, isNew, raffleItemIndex) {
      if (input.files && input.files[0]) {
        if (isNew) {
          this.raffle.photoFile = input.files[0]
        } else {
          this.raffles[raffleItemIndex].photoFile = input.files[0]
          this.updateRaffleFromList(raffleItemIndex)
        }

        let reader = new FileReader()

        reader.onload = function(e) {
          document.getElementById(id).src = e.target.result
        }

        reader.readAsDataURL(input.files[0]) // convert to base64 string
      }
    },

    updatePhoto(idInput, idImage, isNew = false, raffleItemIndex = null) {
      this.readURL(document.getElementById(idInput), idImage, isNew, raffleItemIndex)
    },

    ...mapActions({
      setServerMessage: 'serverMessage/setServerMessage',
    }),
  }
}
</script>
