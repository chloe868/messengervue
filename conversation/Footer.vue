<template>
  <div id="footer" class="holder">
    <div class="product-selected" v-if="selectedProduct !== null"> 
    <!-- <div class="product-selected">  -->
      <!-- <i class="fa fa-image"></i> -->
      <i class="fa fa-times close-icon" aria-hidden="true" @click="deleteSelectedProduct"></i>
      <!-- <img width="auto" height="100"
        src="https://dimg.dillards.com/is/image/DillardsZoom/zoom/perry-ellis-big--tall-herringbone-flat-front-pants/04137298_zi_light_beige.jpg" alt=""> -->
      <img width="auto" height="100" v-if="selectedProductImage !== null" :src="config.BACKEND_URL + selectedProductImage">
      <i class="fa fa-image alt-image" v-else></i> 
      <div class="arrow-down"></div>
    </div>
    <div class="tools-container">
      <i id="attach-file" class="fa fa-paperclip" title="upload file" aria-hidden="true" @click="showImages()"></i>
      <input type="text" class="form-control" placeholder="Type your message here..." 
        v-model="newMessageInput" @keyup="keyupHandler" @keypress="keypressHandler" @input="manageInput"/>
      <i id="send-btn" class="fa fa-paper-plane" title="send message" aria-hidden="true" @click="sendMessage(selectedProduct)"></i>
      <small class="instruction" v-if="user.type === 'PARTNER'">Type @P_ to show/search products</small>
    </div>
    <div class="products" v-if="products.showProducts === true">
      <messenger-products 
        :messageInput="newMessageInput"
        :searchProduct="products.searchedProducts" 
        :selectedProduct="selectedProduct"
        @searchProductEvent="searchProductEventHandler($event)"
        @selectedIdEvent="selectedIdEventHandler($event)">
      </messenger-products>
    </div>
    <browse-images-modal :object="user.profile" v-if="user.profile !== null"></browse-images-modal>
    <browse-images-modal :object="newProfile" v-if="user.profile === null"></browse-images-modal>
    <!-- <img src="" alt=""> -->
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.arrow-down {
  position: absolute;
  width: 0; 
  height: 0; 
  left: 25%;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent; 
  border-top: 20px solid $secondary;
}
.close-icon {
  position: absolute;
  top: 0;
  right: 0;
}
.close-icon:hover {
  cursor: pointer;
}
.alt-image {
  font-size: 90px;
  padding-top: 10px;
}
.product-selected {
  position: absolute;
  top: -110px;
  left: 50px;
  border: 1px solid $secondary;
}
// product-selected img {
//   width: 50px;
//   height: 50px;
// }
.tools-container {
  display: flex;
  align-items: center;
  width: 100%;
}
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
  align-items: flex-end;
  padding-bottom: 20px;
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
      updatedMessage: '',
      selectedProduct: null,
      selectedProductImage: null
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
        let str = this.newMessageInput.slice()
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
        }else{
          this.products = {
            showProducts: false,
            searchedProducts: ''
          }
        }
        if(searchProduct[0] === ' '){
          this.products = {
            ...this.products,
            showProducts: false
          }
        }
      }
    },
    keyupHandler(event){ // hide messenger-products when text-input is cleared
      if(event.target.value === ''){
        this.products = {
          showProducts: false,
          searchedProducts: ''
        }
      }
    },
    keypressHandler(event){ // send message using enter key
      if(event.charCode === 13){
        this.sendMessage()
      }
    },
    searchProductEventHandler(event){
      this.products.searchedProducts = event.searchValue
      this.newMessageInput = event.updatedValue
    },
    sendMessage(){
      let messageType = 'text'
      let messageValue = null
      if(this.selectedProduct !== null){
        messageType = 'product'
        messageValue = this.selectedProduct.id
      }
      if((this.newMessageInput !== '' || this.newMessageInput !== null) && this.group.new === false){
        let parameter = {
          messenger_group_id: this.group.id,
          message: this.newMessageInput,
          account_id: this.user.userID,
          status: 0,
          payload: messageType,
          payload_value: messageValue
        }
        if(this.selectedProduct !== null){
          parameter['product'] = this.selectedProduct
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
            this.$parent.newFlag = false
            AUTH.messenger.messengerGroupId = parseInt(response.data)
            this.$emit('changeGroupEvent', response.data)
          }
        })
      }
      this.selectedProduct = null
    },
    selectedIdEventHandler(product){
      this.selectedProduct = product
      this.selectedProductImage = product.featured[0].url
      // remove @p_...
      let message = this.newMessageInput.slice().toLowerCase()
      let n = message.lastIndexOf('@p')
      let newMessage = message.slice(0, n)
      this.products.showProducts = {
        ...this.products,
        showProducts: false
      }
      this.newMessageInput = newMessage.slice()
    },
    deleteSelectedProduct(){
      this.selectedProduct = null
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
