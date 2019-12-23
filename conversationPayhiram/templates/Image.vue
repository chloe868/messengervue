<template>
  <div class="template" v-if="message.files !== null">
    <div :class="'header' + template">
      <div class="profile">
        <img :src="config.BACKEND_URL + message.account.profile.url" v-if="message.account.profile !== null">
        <i class="fa fa-user-circle-o text-green" v-else></i>
      </div>
      <span class="details" v-if="message.account !== null">
        <label><b>{{message.account.username}}</b></label>
      </span>
    </div>
    <div :class="'content' + template" v-if="message.validations !== null">
      <label>{{message.validations.payload}}</label>
    </div>
    <div :class="'content' + template">
      <img :src="config.BACKEND_URL + item.url" v-for="(item, index) in message.files" :key="index" class="message-image" @click="showImageModal(config.BACKEND_URL + item.url)" />
    </div>
    <div :class="'content' + template" style="margin-top: 10px;" v-if="group.account_id === user.userID">
      <button class="btn btn-danger">Decline</button>
      <button class="btn btn-primary">Approve</button>
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
  border: solid 1px #007bff !important;
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
  text-align: justify;
}
.template .content-right{
  min-height: 10px;
  float: left;
  width: 100%;
  overflow-y: hidden;
  text-align: right;
}
.template .content label, .template .content-righ label{
  line-height: 18px;
}

.message-image{
  width: 24%;
  max-height: auto;
  max-width: 24%;
  border-radius: 10px;
}

.message-image:hover{
  cursor: pointer;
  border: solid 1px $secondary;
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
      config: CONFIG
    }
  },
  components: {
  },
  props: ['message', 'template', 'group'],
  methods: {
    showImageModal(url){
      this.$emit('showImage', {url: url})
    }
  }
}
</script>
