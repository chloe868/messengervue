<template>
  <div id="footer" class="holder">
    <div class="tools-container">
      <i class="fa fa-image" title="Add photo" aria-hidden="true" @click="showImages()"></i>
      <input type="text" placeholder="Type your message here..." v-model="newMessageInput" @keypress="keypressHandler" class="message-input">
<!--       <i class="fa fa-paper-plane" title="Send message" aria-hidden="true" @click="sendMessage()"></i> -->
      <button class="btn send-btn" @click="sendMessage()">Send</button>
    </div>
    <browse-images-modal></browse-images-modal>
    <!-- <img src="" alt=""> -->
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.tools-container {
  display: flex;
  align-items: center;
  width: 100%;
  line-height: 45px;
}

.holder{
  display: inline-flex;
  align-items: center;
  width: 100%;
  float: left;
  height: 50px;
  border-top: 1px solid #eee;
  background-color: $white;
  align-items: flex-end;
  position: relative;
}

.tools-container i{
  font-size: 20px;
  padding-left: 10px;
  padding-right: 10px;
}

.tools-container i:hover{
  cursor: pointer;
  color: $secondary;
}

.message-input{
  border: 0px;
  width: 100%;
  line-height: 45px;
}

.send-btn{
  background: $white;
  color: #007bff;
  border: solid 1px #007bff;
  height: 40px;
  margin-top: 2px;
  margin-right: 10px;
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
      errorMessage: null,
      newMessageInput: ''
    }
  },
  props: ['group'],
  components: {
    'browse-images-modal': require('components/increment/generic/image/BrowseModal.vue'),
    'messenger-modal': require('components/increment/messengervue/conversationPayhiram/modal/Modal.vue')
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    keypressHandler(event){
      if(event.charCode === 13){
        this.sendMessage()
      }
    },
    sendMessage(){
      if((this.newMessageInput !== '' || this.newMessageInput !== null) && AUTH.messenger.group.new === false){
        let parameter = {
          messenger_group_id: AUTH.messenger.group.id,
          message: this.newMessageInput,
          account_id: this.user.userID,
          status: 0,
          payload: 'text',
          payload_value: null,
          code: AUTH.messenger.messages.length + 1
        }
        let newMessageTemp = {
          ...parameter,
          account: this.user,
          created_at_human: null,
          sending_flag: true,
          error: null
        }
        AUTH.messenger.messages.push(newMessageTemp)
        this.newMessageInput = null
        this.APIRequest('messenger_messages/create', parameter).then(response => {
          if(response.data !== null){
            this.newMessageInput = null
            // update previous message
            for (var i = AUTH.messenger.messages.length - 1; i > 0; i--) {
              let item = AUTH.messenger.messages[i]
              if(typeof item.code === 'undefined' || item.code === undefined){
                break
              }
              if(parseInt(item.code) === parseInt(response.data.code)){
                AUTH.messenger.messages[i] = response.data
                break
              }
            }
          }
        })
      }
    },
    showImages(){
      $('#browseImagesModal').modal('show')
    },
    hideImages(){
      $('#browseImagesModal').modal('hide')
    }
  }
}
</script>
