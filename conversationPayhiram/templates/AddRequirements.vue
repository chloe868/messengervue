<template>
  <div class="conversations" v-if="parseInt(group.account_id) === user.userID && parseInt(group.request.type) === 1">
    <div class="message-row" v-if="group.validations.complete_status === false">
      <div class="template">
        <div class="incre-row text-center">
          <label class="text-primary">Hi <b>{{user.username}}!</b> Allow validation by clicking the options below.</label>
          <span class="incre-row">
            <button class="btn btn-white text-primary" @click="addValidation(item.payload)" v-for="(item, index) in group.validations.requirements" :key="index" v-if="item.validations === null" style="margin-right: 10px;">{{item.title}}</button>
          </span>
        </div>
      </div>
    </div>
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
export default {
  mounted(){
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG
    }
  },
  props: ['group'],
  methods: {
    addValidation(payload){
      let parameter = {
        account_id: this.user.userID,
        payload: payload,
        request_id: this.group.payload,
        status: 'pending'
      }
      this.APIRequest('request_validations/create', parameter).then(response => {
        if(response.data > 0){
          this.$parent.retrieve()
        }
      })
    }
  }
}
</script>
