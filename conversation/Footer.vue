<template>
  <div id="footer" class="holder">
    <i id="attach-file" class="fa fa-paperclip" title="upload file" aria-hidden="true" @click="showImages()"></i>
    <input type="text" class="form-control" placeholder="Type your message here..." 
      v-model="newMessageInput" @keypress="sendMessageOnEnter" @input="manageInput"/>
    <small class="instruction">Type @P_ to show/search products</small>
    <i id="send-btn" class="fa fa-paper-plane" title="send message" aria-hidden="true" @click="sendMessage()"></i>
    <div class="products" v-if="products.showProducts === true">
      <messenger-products 
        :messageInput="newMessageInput"
        :searchProduct="products.searchedProducts" 
        @searchProductEvent="products.searchedProducts = $event.searchValue, newMessageInput = $event.updatedValue">
      </messenger-products>
    </div>
    <browse-images-modal :object="user.profile" v-if="user.profile !== null"></browse-images-modal>
    <browse-images-modal :object="newProfile" v-if="user.profile === null"></browse-images-modal>
    <!-- <img src="" alt=""> -->
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.products{
  position: absolute;
  height: calc(100vh - 300px);
  width: 300px;
  z-index: 10;
  bottom: 12vh;
  right: 20px;
  background: white;
  border: 1px solid $primary;
  padding: 10px;
  border-radius: 5px;
  // overflow: auto;
}
small.instruction {
    position: absolute;
    bottom: 2px;
    font-size: 8pt;
    color: white;
    left: 59px;
}
.holder{
  display: inline-flex;
  align-items: center;
  width: 100%;
  float: left;
  height: 10vh;
  border-top: 1px solid $primary;
  background-color: $primary;
  position: relative;
}
.profile{
  width: 50px;
  height: 50px;
  border-radius: 50%;
  float: left;
}
.btn{
  width: 10%;
  float: right;
  height: 45px;
  position: relative;
  bottom: 60px;
  right: 13px;
  text-align: center;
}
.form-control{
  width: 100% !important;
  float: right !important;
  height: 5vh !important;
}
span{
  width: 10%;
  float: left;
  height: 45px;
  line-height: 8vh;
  text-align: center;
}
span i{
  font-size: 24px;
}
span i:hover{
  cursor: pointer;
  color: #3f0050;
}
#attach-file {
  font-size: 3vh;
  padding: 0 20px 0 10px;
  color: $white;
}
#send-btn {
  font-size: 3vh;
  padding: 0 20px 0 10px;
  color: $white;
}
#send-btn:hover, #attach-file:hover {
  cursor: pointer;
  color: $hover;
}
</style>
<script>
import ROUTER from '../../../../router'
import AUTH from '../../../../services/auth'
import CONFIG from '../../../../config.js'
export default {
  mounted(){
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      newProfile: {
        account_id: null,
        url: null
      },
      errorMessage: null,
      newMessageInput: '',
      prevNewMessageIndex: null,
      payload: null,
      products: {
        showProducts: false,
        searchedProducts: ''
      },
      updatedMessage: ''
    }
  },
  props: ['group'],
  components: {
    'browse-images-modal': require('components/increment/generic/image/BrowseModal.vue'),
    'messenger-products': require('components/increment/messengervue/products/MessengerProducts.vue')
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    manageInput(event){
      this.newMessageInput = event.target.value
      if(this.newMessageInput && this.user.type === 'PARTNER'){
        let str = this.newMessageInput
        let lowStr = str.toLowerCase()
        let trigger = lowStr.endsWith('@p') // triggers messenger-products
        let n = lowStr.lastIndexOf('@p_')
        let temp = n > -1 ? str.substring(n) : ''  // true if '@p_' is found
        let searchProduct = temp.slice(3) // removing @P_ in searching
        if(trigger && str !== ' ' || n > -1){
          this.products = {
            showProducts: true,
            searchedProducts: searchProduct
          }
          $('#messenger-prodcut-holder:first').addClass('hovered')
        }else{
          this.products = {
            showProducts: false,
            searchedProducts: ''
          }
        }
        if(searchProduct[0] === ' '){
          this.products = {
            ...this.prodcuts,
            showProducts: false
          }
        }
      }
    },
    sendMessageOnEnter(event){
      if(event.charCode === 13){
        this.sendMessage()
      }
    },
    sendMessage(){
      if((this.newMessageInput !== '' || this.newMessageInput !== null) && this.group.new === false){
        let parameter = {
          messenger_group_id: this.group.id,
          message: this.newMessageInput,
          account_id: this.user.userID,
          status: 'messenger',
          payload: 'messenger',
          type: 'messenger'
        }
        this.APIRequest('messenger_messages/create', parameter).then(response => {
          if(response.data !== null){
            this.newMessageInput = null
            AUTH.messenger.messages.push(response.data)
          }
        })
      }else if((this.newMessageInput !== '' || this.newMessageInput !== null) && this.group.new === true){
        let parameter = {
          creator: this.user.userID,
          message: this.newMessageInput,
          member: this.group.id,
          status: 'messenger',
          payload: 'messenger',
          type: 'messenger'
        }
        this.APIRequest('custom_messenger_groups/create', parameter).then(response => {
          if(response.data !== null){
            this.newMessageInput = null
            this.$parent.group = response.data
            this.$parent.newFlag = false
            AUTH.messenger.messengerGroupId = parseInt(response.data)
            this.$parent.retrieve()
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
