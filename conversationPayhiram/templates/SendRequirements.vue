<template>
  <div class="conversations" v-if="parseInt(group.account_id) !== user.userID && parseInt(group.request.status) < 2 && parseInt(group.request.type) === 1">
    <div class="message-row">
      <div class="template">
        <div class="incre-row text-center">
          <label class="text-primary">Hi <b>{{user.username}}!</b> Send the requirements below. Just click the button.</label>
          <span class="incre-row">
            <button class="btn btn-white text-primary" @click="complyRequirements(item)" v-for="(item, index) in group.validations.requirements" :key="index" v-if="item.validations !== null" style="margin-right: 10px;">{{item.title}}</button>
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
      config: CONFIG,
      validation: null
    }
  },
  props: ['group'],
  methods: {
    complyRequirements(item){
      this.validation = item
      $('#browseImagesModal').modal('show')
    },
    manageImageUrl(url){
      if(this.validation !== null){
        this.sendMessage(url)
      }
    },
    sendMessage(url){
      if(url !== null){
        console.log(this.validation)
        let parameter = {
          messenger_group_id: this.group.id,
          message: this.newMessageInput,
          account_id: this.user.userID,
          status: 0,
          payload: 'image',
          payload_value: this.validation.validations.id,
          url: url
        }
        this.APIRequest('messenger_messages/create_with_images', parameter).then(response => {
          if(response.data !== null){
            this.newMessageInput = null
            AUTH.messenger.messages.push(response.data)
            this.$parent.retrieve()
          }
        })
      }
    }
  }
}
</script>
