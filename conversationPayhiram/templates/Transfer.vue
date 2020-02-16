<template>
  <div class="conversations">
    <div class="message-row" v-if="parseInt(auth.messenger.group.account_id) === user.userID && auth.messenger.group.validations.transfer_status === 'approved' && parseInt(auth.messenger.group.request.status) < 2 && jquery.inArray(parseInt(auth.messenger.group.request.type), common.fulfillmentTypesWithValidation)">
      <div class="template">
        <div class="incre-row text-center">
          <label class="text-primary">Hi <b>{{user.username}}!</b> You've completed the validation, click transfer to proceed:</label>
          <span class="incre-row">
            <button class="btn btn-white text-primary" @click="transfer()">Transfer</button>
          </span>
        </div>
      </div>
    </div>
    <div class="message-row" v-if="parseInt(auth.messenger.group.account_id) === user.userID && parseInt(auth.messenger.group.request.status) < 2 && parseInt(auth.messenger.group.request.type) === 2">
      <div class="template">
        <div class="incre-row text-center">
          <label class="text-primary">Hi <b>{{user.username}}!</b> If you receive the money from other peer already, then you can continue to transfer and complete the thread.</label>
          <span class="incre-row">
            <button class="btn btn-white text-primary" @click="transfer()">Transfer</button>
          </span>
        </div>
      </div>
    </div>
    <div class="message-row" v-if="parseInt(auth.messenger.group.account_id) !== user.userID && parseInt(auth.messenger.group.request.type) === 3 && parseInt(auth.messenger.group.request.status) < 2">
      <div class="template">
        <div class="incre-row text-center">
          <label class="text-primary">Hi <b>{{user.username}}!</b> If you receive the money from other peer already, then you can continue to transfer and complete the thread.</label>
          <span class="incre-row">
            <button class="btn btn-white text-primary" @click="transfer()">Transfer</button>
          </span>
        </div>
      </div>
    </div>
    <authenticate-otp ref="authenticateOTP" v-if="common.authorize === 'OTP'"></authenticate-otp>
    <authenticate-pin ref="authenticateOTP" v-if="common.authorize === 'PIN'"></authenticate-pin>
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.btn{
  height: 45px !important;
}
.btn-white{
  background: $white;
  border: solid 1px $textBlue !important;
}

.btn-white:hover{
  cursor: pointer;
  background: $primary !important;
  color: $white !important;
}

.message-row, .template{
  width: 98%;
  min-height: 10px;
  overflow-y: hidden;
  margin-bottom: 5px;
  margin-top: 5px;
  margin-left: 1%;
  margin-right: 1%;
}

</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import LEDGER from 'src/services/Ledger.js'
import COMMON from 'src/common.js'
export default {
  mounted(){
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      auth: AUTH,
      common: COMMON,
      jquery: $
    }
  },
  props: ['group'],
  components: {
    'authenticate-otp': require('modules/transfer/Otp.vue'),
    'authenticate-pin': require('modules/transfer/Pin.vue')
  },
  methods: {
    transfer(){
      this.$refs.authenticateOTP.show()
    },
    successOTP(){
      console.log('hi')
      LEDGER.processRequest(AUTH.messenger.group, this.user.userID, response => {
        if(response.data !== null && response.data === true){
          this.$parent.retrieve()
        }else{
          // error message here
        }
      })
    }
  }
}
</script>
