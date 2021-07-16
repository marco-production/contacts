<template>
  <v-container class="lighten-5 mt-5">
    <v-col class="text-right">
      <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="#388E3C" dark class="mb-2" v-bind="attrs" v-on="on">
                <v-icon left dark>mdi-plus-thick</v-icon> 
                New Contact
              </v-btn>
            </template>
            <v-card>
              
              <v-card-title>
                <span class="text-h5">{{ title }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <validation-observer ref="observer">
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <validation-provider name="First Name" rules="required" v-slot="{errors}">
                        <v-text-field v-model="editedContact.firstname" :error-messages="errors" label="First Name"></v-text-field>
                      </validation-provider>
                    </v-col>
                  <v-col cols="12" sm="6" md="6">
                    <validation-provider name="Last Name" rules="required" v-slot="{errors}">
                        <v-text-field v-model="editedContact.lastname" :error-messages="errors" label="Last Name" ></v-text-field>
                      </validation-provider>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <validation-provider name="Email" rules="required|email" v-slot="{errors}">
                      <v-text-field v-model="editedContact.email" :error-messages="errors" label="Email"></v-text-field>
                      </validation-provider>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <validation-provider name="Phone" rules="required" v-slot="{errors}">
                      <v-text-field v-mask="'(###) ###-####'" v-model="editedContact.phone" :error-messages="errors" label="Phone"></v-text-field>
                      </validation-provider>
                    </v-col>
                  </v-row>
                  </validation-observer>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="save">Save</v-btn>
              </v-card-actions>

            </v-card>
          </v-dialog>
    </v-col>
    <table>
      <thead>
        <tr>
          <th scope="col">Fullname</th>
          <th scope="col">Email</th>
          <th scope="col">Telephone</th>
          <th scope="col">Created at</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="contact in contacts" :key=contact.firstname>
          <td data-label="Account">{{ contact.firstname }} {{ contact.lastname }}</td>
          <td data-label="Due Date">{{ contact.email }}</td>
          <td data-label="Amount">{{ contact.phone }}</td>
          <td data-label="Period">{{ contact.createdAt }}</td>
          <td data-label="Actions">
            
            <v-tooltip left>
              <template v-slot:activator="{ on, attrs }">
                <v-btn :href="'tel:'+contact.phone" color="blue" class="ma-2 white--text" v-bind="attrs" v-on="on"> 
                  <v-icon centered dark>mdi-phone</v-icon>
                </v-btn>
              </template>
              <span>Call</span>
            </v-tooltip>

            <v-btn color="warning" @click="editContact(contact)" class="ma-2 white--text"> 
              Edit
              <v-icon right dark>mdi-pencil-outline</v-icon>
            </v-btn>
            <v-btn color="red" @click="deleteContact(contact)" class="ma-2 white--text"> 
              Delete
              <v-icon right dark>mdi-trash-can-outline</v-icon>
            </v-btn>

          </td>
        </tr>
      </tbody>
    </table>

    <v-dialog v-model="dialogDelete" max-width="550px">
      <v-card>
        <v-card-title class="text-h5">Are you sure you want to delete this contact?</v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
          <v-btn color="blue darken-1" text @click="deleteContactConfirm">Delete</v-btn>
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>
    
  </v-container>
</template>

<script>
import { ValidationProvider, ValidationObserver, extend } from 'vee-validate';
import * as rules from 'vee-validate/dist/rules';
import { messages } from 'vee-validate/dist/locale/en.json';

Object.keys(rules).forEach(rule => {
    extend(rule, {
    ...rules[rule],
    message: messages[rule]
  });
});

for (let [rule, validation] of Object.entries(rules)) {
  extend(rule, {
    ...validation
  });
}

 export default {
   name: 'Index',
   
    data: () => ({
      dialog: false,
      dialogDelete: false,

      contacts: [],
      editedIndex: -1,
      editedContact: {
        firstname: '',
        lastname: '',
        email: '',
        phone: '',
      },
      defaultContact: {
        firstname: '',
        lastname: '',
        email: '',
        phone: '',
      },
    }),

    components: {
      ValidationProvider,
      ValidationObserver,
    },

    computed: {
      title () {
        return this.editedIndex === -1 ? 'New contact' : 'Edit contact'
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

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
        this.contacts = [
          {
            firstname: 'Marco Antonio',
            lastname: 'De la cruz Santos',
            email: 'marco@hotmail.com',
            phone: '(829) 585-0900',
            createdAt: this.currentDate(),
          },
        ]
      },

      currentDate() {
        const current = new Date();
        const date = `${current.getDate()}/${current.getMonth()+1}/${current.getFullYear()}`;
        return date;
      },

      editContact (item) {
        this.editedIndex = this.contacts.indexOf(item)
        this.editedContact = Object.assign({}, item)
        this.dialog = true
      },

      deleteContact (item) {
        this.editedIndex = this.contacts.indexOf(item)
        this.editedContact = Object.assign({}, item)
        this.dialogDelete = true
      },

      deleteContactConfirm () {
        this.contacts.splice(this.editedIndex, 1)
        this.closeDelete()
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedContact = Object.assign({}, this.defaultContact)
          this.editedIndex = -1
        })
      },

      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedContact = Object.assign({}, this.defaultContact)
          this.editedIndex = -1
        })
      },

      async save () {

        const isValid = await this.$refs.observer.validate()

        if(isValid){

          if (this.editedIndex > -1) {

            this.editedContact.createdAt = this.currentDate()
            Object.assign(this.contacts[this.editedIndex], this.editedContact)

          } else {

            this.editedContact.createdAt = this.currentDate()
            this.contacts.push(this.editedContact)
          }

          this.close()
        }
      },
    },
  }
</script>
<style scoped>

  table {
    border: 1px solid #ccc;
    border-collapse: collapse;
    margin: 0;
    padding: 0;
    width: 100%;
    table-layout: fixed;
    margin-top: 20px;
  }

  table tr {
    background-color: #f8f8f8;
    border: 1px solid #ddd;
    padding: .35em;
  }

  table th,
  table td {
    padding: .625em;
    text-align: center;
  }

  table th {
    font-size: .85em;
    letter-spacing: .1em;
    text-transform: uppercase;
  }

  @media screen and (max-width: 600px) {
    table {
      border: 0;
    }

    table thead {
      border: none;
      clip: rect(0 0 0 0);
      height: 1px;
      margin: -1px;
      overflow: hidden;
      padding: 0;
      position: absolute;
      width: 1px;
    }
    
    table tr {
      border-bottom: 3px solid #ddd;
      display: block;
      margin-bottom: .625em;
    }
    
    table td {
      border-bottom: 1px solid #ddd;
      display: block;
      font-size: .8em;
      text-align: right;
    }
    
    table td::before {
      content: attr(data-label);
      float: left;
      font-weight: bold;
      text-transform: uppercase;
    }
    
    table td:last-child {
      border-bottom: 0;
    }
  }
</style>