<template>
  <div id="messenger">
    <div class="messenger-holder" v-if="conversations !== null">
      <div class="message-row" v-for="(item, index) in conversations" :key="index">
        
        <div v-if="parseInt(item.account_id) !== user.userID">
          <image-message :message="item" :template="''" :group="group" v-if="item.payload === 'image'" @showImage="showImage($event)"></image-message>
          <div class="template" v-if="item.payload === 'text'">
            <div class="header">
              <div class="profile">
                <img :src="config.BACKEND_URL + item.account.profile.url" v-if="item.account.profile !== null">
                <i class="fa fa-user-circle-o text-green" v-else></i>
              </div>
              <span class="details" v-if="item.account !== null">
                <label><b>{{item.account.username}}</b></label>
              </span>
            </div>
            <div class="content">
              <label>
                <label v-html="item.message"></label>
              </label>
            </div>
          </div>
        </div>

        <div v-else>
          <image-message :message="item" :template="'-right'" :group="group" v-if="item.payload === 'image'" @showImage="showImage($event)"></image-message>
          <div class="template" v-if="item.payload === 'text'">
            <div class="header-right">
              <div class="profile">
                <img :src="config.BACKEND_URL + item.account.profile.url" v-if="item.account.profile !== null">
                <i class="fa fa-user-circle-o text-green" v-else></i>
              </div>
              <span class="details" v-if="item.account !== null">
                <label><b>{{item.account.username}}</b></label>
              </span>
            </div>
            <div class="content-right">
              <label id="labelRight">
                <bdi>
                  <label v-html="item.message"></label>
                </bdi>
              </label>
            </div>
          </div>
        </div>

      </div>
    </div>
    <div class="conversations" v-if="parseInt(group.account_id) === user.userID">
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
      <div class="message-row" v-if="group.validations.transfer_status === 'approved'">
        <div class="template">
          <div class="incre-row text-center">
            <label class="text-primary">Hi <b>{{user.username}}!</b> You've compled the validation, click transfer to proceed:</label>
            <span class="incre-row">
              <button class="btn btn-white text-primary" @click="transfer()">Transfer</button>
            </span>
          </div>
        </div>
      </div>
    </div>
    <div class="conversations" v-if="parseInt(group.account_id) !== user.userID">
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
    <div class="conversations" v-if="parseInt(group.request.status) === 2">
      <div class="message-row" v-if="group.rating === null">
        <div class="template">
          <div class="incre-row text-center">
            <label class="text-primary">Hi <b>{{user.username}}!</b> Please help <b>{{group.title.username}}</b> by giving a review.</label>
            <span class="incre-row">
              <button class="btn btn-white text-primary" @click="review()">Submit review and rating</button>
            </span>
          </div>
        </div>
      </div>
      <div class="incre-row text-center">
        <label><i style="color: #aaa">Transaction complete!</i></label>
      </div>
    </div>
    <add-ratings ref="addRatings"></add-ratings>
    <authenticate-otp ref="authenticateOTP"></authenticate-otp>
    <browse-images-modal></browse-images-modal>
    <show-image-modal ref="showImage"></show-image-modal>
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
.content-product {
  border: 1px solid $primary;
  float: right !important;
  width: auto !important;
  height: auto !important;
}
.messenger-holder{
  width: 100%;
  float: left;
  min-height: 50px;
  overflow-y: hidden;
}
.messenger-messages{
  width: 100%;
  float: left;
  min-height: 50px;
  overflow-y: hidden;
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
.template .header{
  height: 40px;
  width: 100%;
  float: left;
}

.header .profile{
  float: left;
  width: 40px;
  height: 40px;
  margin-right: 2%;
}
.header .profile img{
  height: 30px;
  width: 30px;
  z-index: 0;
  border-radius: 50%;
}

.header .profile i{
  line-height: 30px;
  font-size: 30px;
  color: $secondary !important;
}
.header .details{
  float: left;
  height: 40px;
}
.header .details label{
  width: 100%;
  float: left;
  color: #555;
  line-height: 12px;
  line-height: 30px;
}


.header-right .profile{
  float: right;
  width: 40px;
  height: 40px;
  margin-left: 2%;
  text-align: right;
}
.header-right .profile img{
  height: 30px;
  width: 30px;
  z-index: 0;
  border-radius: 50%;
}

.header-right .profile i{
  line-height: 30px;
  font-size: 30px;
  color: $primary !important;
}
.header-right .details{
  float: right;
  height: 40px;
}
.header-right .details label{
  width: 100%;
  float: right;
  color: #555;
  line-height: 12px;
  line-height: 30px;
}
.template .content{
  min-height: 10px;
  float: left;
  width: 100%;
  overflow-y: hidden;
  text-align: left;
  max-width: 60%;
}
.template .content-right{
  min-height: 10px;
  float: left;
  width: 100%;
  overflow-y: hidden;
  text-align: right;
}
.template .content label{
  line-height: 18px;
  background: $textBlue;
  color: $white;
  padding-top: 7px;
  padding-bottom: 7px;
  padding-right: 5px;
  padding-left: 10px;
  margin-bottom: 0px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
}

.template .content-right label{
  line-height: 18px;
  background: $textBlue;
  color: $white;
  padding-top: 5px;
  padding-bottom: 5px;
  padding-right: 10px;
  padding-left: 5px;
  margin-bottom: 0px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  border-bottom-left-radius: 10px;
}

.template .content-right #labelRight{
  max-width: 60%;
}


</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
export default {
  mounted(){
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      validation: null,
      requirements: {
        url: null
      }
    }
  },
  components: {
    'image-message': require('components/increment/messengervue/conversationPayhiram/templates/Image.vue'),
    'add-ratings': require('components/increment/generic/rating/Create.vue'),
    'authenticate-otp': require('modules/transfer/Otp.vue'),
    'browse-images-modal': require('components/increment/generic/image/BrowseModal.vue'),
    'show-image-modal': require('components/increment/generic/modal/Image.vue')
  },
  props: ['conversations', 'group'],
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    review(){
      let payload = 'profile'
      let payloadValue = this.group.title.id
      if(payloadValue !== null){
        this.$refs.addRatings.show(payload, payloadValue, 'request', this.group.payload)
      }
    },
    transfer(){
      this.$refs.authenticateOTP.show()
    },
    addValidation(payload){
      let parameter = {
        account_id: this.user.userID,
        payload: payload,
        request_id: this.group.payload,
        status: 'pending'
      }
      this.APIRequest('request_validations/create', parameter).then(response => {
        if(response.data > 0){
          this.$parent.retrieveParent()
        }
      })
    },
    retrieve(){
      this.$parent.retrieveParent()
    },
    complyRequirements(item){
      this.validation = item
      $('#browseImagesModal').modal('show')
    },
    manageImageUrl(url){
      if(this.validation !== null){
        this.sendMessage(url)
      }
    },
    showImage(event){
      this.$refs.showImage.setImage(event.url)
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
            this.$parent.retrieveParent()
          }
        })
      }
    },
    successOTP(){
      console.log('process thread')
    }
  }
}
</script>
