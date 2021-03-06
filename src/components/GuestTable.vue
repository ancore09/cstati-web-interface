<template>
  <div>
    <v-data-table
        :headers="headers"
        :items="$store.state.guests"
        class="elevation-1 pt-2"
        :loading="$store.state.guests.length == 0"
        loading-text="Загрузка.."
        :items-per-page="15"
        :search="search"
        :header-props="headerProps"
        :footer-props="footerProps"
    >

      <template v-slot:top>
        <v-toolbar
            flat
        >
          <v-toolbar-title>Таблица гостей</v-toolbar-title>
          <v-divider
              class="mx-4"
              inset
              vertical
          ></v-divider>
          <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Поиск"
              class="mt-5"
          ></v-text-field>
          <v-spacer></v-spacer>
          <v-dialog
              v-model="dialog"
              max-width="650px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  v-bind="attrs"
                  v-on="on"
                  icon
              >
                <v-icon>mdi-plus</v-icon>
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.tgId"
                          label="Id"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.name"
                          label="Имя"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.phone"
                          label="Номер телефона"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-text-field
                          v-model="editedItem.sex"
                          label="Пол"
                      ></v-text-field>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-select
                          :items="[1, 2, 3]"
                          v-model="editedItem.room_type"
                          label="Тип билета"
                      ></v-select>
                    </v-col>
                    <v-col
                        cols="12"
                        sm="6"
                        md="4"
                    >
                      <v-checkbox
                        v-model="editedItem.payment"
                        label="Факт оплаты"
                      >
                      </v-checkbox>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                    color="blue darken-1"
                    text
                    @click="close"
                >
                  Отменить
                </v-btn>
                <v-btn
                    color="blue darken-1"
                    text
                    @click="save"
                >
                  Сохранить
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">Вы уверены, что хотите удалить этот <br> объект?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Отменить</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm">Удалить</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>

        </v-toolbar>
      </template>

      <template v-slot:item.payment="{ item }">
        <v-simple-checkbox
            v-model="item.payment"
            :ripple="false"
            v-on:input="updateItem(item)"
        ></v-simple-checkbox>
      </template>

      <template v-slot:item.room_type="{ item }">
        {{ getPrice(item.room_type) }}
      </template>

      <template v-slot:item.actions="{ item }">
        <v-icon
            small
            class="mr-2"
            @click="editItem(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
            small
            @click="deleteItem(item)"
        >
          mdi-delete
        </v-icon>
      </template>

    </v-data-table>
  </div>
</template>

<script>
import axios from "axios";

const COLORS = {
  info: 'info',
  warning: 'warning',
  error: 'error',
  success: 'success',
}
const ICONS = {
  info: 'mdi-information',
  warning: 'mdi-alert',
  error: 'mdi-alert-circle',
  success: 'mdi-check-circle',
}

export default {
  data () {
    return {
      items: [],
      search: '',
      loading: true,
      headers: [
        { text: 'Id', value: 'tgId', align: 'start', width: "20%"},
        { text: 'Имя', value: 'name', width: "20%"},
        { text: 'Номер телефона', value: 'phone' },
        { text: 'Пол', value: 'sex' },
        { text: 'Цена билета', value: 'room_type' },
        { text: 'Факт оплаты', value: 'payment' },
        { text: 'Действия', value: 'actions', sortable: false },
      ],
      headerProps: {
        sortByText: 'Сортировать'
      },
      footerProps: {
        itemsPerPageAllText: 'Все',
        itemsPerPageText: 'Объектов на странице'
      },
      editedIndex: -1,
      dialogDelete: false,
      dialog: false,
      editedItem: {
        tgId: 0,
        name: '',
        phone: '',
        room_type: 0,
        sex: '',
        payment: false
      },
      defaultItem: {
        tgId: 0,
        name: '',
        phone: '',
        room_type: 0,
        sex: '',
        payment: false
      }
    }
  },
  async mounted() {
    if (this.$store.getters.getGuests.length == 0) {
      this.$store.dispatch('loadGuests')
    }
  },
  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'Добавление' : 'Изменение'
    },
  },
  methods: {
    getPrice(type) {
      switch (type) {
        case 1:
          return 3500
        case 2:
          return 4500
        case 3:
          return 6000
      }
    },
    editItem (item) {
      this.editedIndex = this.$store.state.guests.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    deleteItem (item) {
      this.editedIndex = this.$store.state.guests.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },
    async deleteItemConfirm () {
      await this.$store.dispatch('deleteItem', this.editedIndex)
      await this.$store.dispatch('postLog', {
        text: 'Объект был удален',
        icon: ICONS.warning,
        color: COLORS.warning
      })
      this.closeDelete()
    },
    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    closeDelete () {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    async save () {
      if (this.editedIndex > -1) {
        await this.$store.dispatch('putItem', this.editedItem)
        await this.$store.dispatch('postLog', {
          text: 'Объект был обновлен',
          icon: ICONS.success,
          color: COLORS.success
        })
      } else {
        await this.$store.dispatch('postItem', this.editedItem)
        await this.$store.dispatch('postLog', {
          text: 'Объект был добавлен',
          icon: ICONS.success,
          color: COLORS.success
        })
      }
      this.close()
    },
    async updateItem(item) {
      await this.$store.dispatch('putItem', item)
      await this.$store.dispatch('postLog', {
        text: 'Объект был обновлен',
        icon: ICONS.success,
        color: COLORS.success
      })
    },
  },
  watch: {
    dialog (val) {
      val || this.close()
    },
    dialogDelete (val) {
      val || this.closeDelete()
    },
  },
}
</script>

<style scoped>

</style>
