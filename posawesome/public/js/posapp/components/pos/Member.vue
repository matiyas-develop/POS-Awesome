<template>
  <v-row justify="center">
   <v-dialog
      v-model="membershipcardDialog"
      max-width="600px"
      @click:outside="clear_customer"
    >
      <v-card>
        <v-card-title>
          <span v-if="membershipcard_id" class="headline primary--text">{{
            __('Update Customer')
          }}</span>
          <span v-else class="headline primary--text">{{
            __('Create Membership Card')
          }}</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('Valid From') + ' *'"
                  background-color="white"
                  hide-details
                  v-model="valid_from"
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._(' Valid Upto') + ' *'"
                  background-color="white"
                  hide-details
                  v-model="valid_upto "
                ></v-text-field>
              </v-col>
               <v-col cols="12">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('Customer') + ' *'"
                  background-color="white"
                  hide-details
                  v-model="customer"
                ></v-text-field>
              </v-col>
              </v-col>
               <v-col cols="12">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._('Description')+ ' *'"
                  background-color="white"
                  hide-details
                  v-model="description"
                ></v-text-field>
              </v-col>
               <v-col cols="12">
                <v-text-field
                  dense
                  color="primary"
                  :label="frappe._(' Max Use ')+ ' *'"
                  background-color="white"
                  hide-details
                  v-model="max_use"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="error" dark @click="close_dialog">{{
            __('Close')
          }}</v-btn>
          <v-btn color="success" dark @click="submit_dialog">{{
            __('Submit')
          }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    membershipcardDialog: false,
    pos_profile: '',
    membershipcard_id: '',
   
  }),
  watch: {},
  methods: {
    close_dialog() {
      this.membershipcardDialog = false;
      this.clear_customer();
    },
    clear_customer() {
      this.name = '';
      
    },

    submit_dialog() {
      // validate if all required fields are filled
      if (!this.valid_from) {
        evntBus.$emit('show_mesage', {
          text: __('Valid from required.'),
          color: 'error',
        });
        return;
      }
     
      if (this.valid_from) {
        const vm = this;
        const args = {
          valid_from: this.valid_from,
          valid_upto :this.valid_upto,
          customer:this.customer,
          description:this.description,
          max_use:this.max_use,
          pos_profile_doc: this.pos_profile,
        };
        frappe.call({
          method: 'posawesome.posawesome.api.posapp.create_membership_card',
          args: args,
          callback: (r) => {
            if (!r.exc && r.message.name) {
              let text = __('Membership Card created successfully.');
              if (vm.membershipcard_id) {
                text = __('Customer updated successfully.');
              }
              evntBus.$emit('show_mesage', {
                text: text,
                color: 'success',
              });
              args.name = r.message.name;
              frappe.utils.play_sound('submit');
              evntBus.$emit('set_membershipcard', r.message.name);
              this.close_dialog();
            } else {
              frappe.utils.play_sound('error');
              evntBus.$emit('show_mesage', {
                text: __('Customer creation failed.'),
                color: 'error',
              });
            }
          },
        });
        this.membershipcardDialog = false;
      }
    },
  },
  created: function () {
    evntBus.$on('open_member', (data) => {
      this.membershipcardDialog = true;
      if (data) {
        this.name = data.name;
        
      }
    });
    evntBus.$on('open_member', (data) => {
      this.membershipcardDialog = true;
      if (data) {
        this.name = data.name;
        this.unique_code = data.unique_code;
      }
    });
    evntBus.$on('register_pos_profile', (data) => {
      this.pos_profile = data.pos_profile;
    });
    evntBus.$on('payments_register_pos_profile', (data) => {
      this.pos_profile = data.pos_profile;
    });
    this.getCustomerGroups();
    this.getCustomerTerritorys();
    this.getGenders();
    // set default values for customer group and territory from user defaults
    this.group = frappe.defaults.get_user_default('Customer Group');
    this.territory = frappe.defaults.get_user_default('Territory');
  },
};
</script>
