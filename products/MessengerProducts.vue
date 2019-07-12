<template>
  <div class="marketplace-holder">
    <div class="product-holder">
      <div class="listing">
        <div class="filter" v-if="data !== null">
          <div class="input-group">
            <input type="text" class="form-control" :value="searchProduct" @keyup="searchProductHandler" placeholder="Search here...">
          </div>
        </div>
        <div class="results">
          <products v-if="data !== null" :data="sortedData" :selectedId="selectedProductId" @selectedIdEvent="selectedIdEventHandler($event)"></products>
          <dynamic-empty v-if="data === null" :title="'No products yet!'" :action="'Please be back soon.'" :icon="'far fa-smile'" :iconColor="'text-primary'"></dynamic-empty>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.marketplace-holder{
  width: 100%;
  float: left;
  min-height: 10px;
  overflow-y: hidden;
  margin-bottom: 50px;
  position: relative;
}
.product-holder{
  width: 100%;
  float: left;
  min-height: 10px;
  overflow-y: hidden;
  position: relative;
}
.listing{
  width: 100%;
  float: left;
  min-height: 10px;
  overflow-y: hidden;
  position: relative;
}
.listing .filter{
  width: 100%;
  float: left;
  height: 50px;
}
.form-control{
  height: 45px !important;
}
.input-group{
  margin-bottom: 10px !important;
}
.input-group-addon{
  width: 100px !important;
  background: #22b173 !important;
  color: #fff !important;
}
.input-group-title{
  width: 100px !important;
  background: #028170 !important;
  color: #fff !important;
}

.listing .results{
  width: 100%;
  font-size: left;
  min-height: 10px;
  overflow-y: hidden;
}

.results {
  overflow-y: auto !important;
  /* height: 438px; */
  height: calc(100vh - 362px) !important;
}

</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import axios from 'axios'
export default {
  mounted(){
    this.retrieve()
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      errorMessage: null,
      data: null,
      searchValue: '',
      newValue: ''
    }
  },
  components: {
    'products': require('components/increment/messengervue/products/Products.vue'),
    'dynamic-empty': require('components/increment/generic/empty/EmptyDynamicIcon.vue')
  },
  props: ['searchProduct', 'messageInput', 'selectedProductId'],
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    retrieve(){
      let parameter = {
        condition: [{
          value: 'published',
          column: 'status',
          clause: '='
        },
        {
          value: this.user.userID,
          column: 'account_id',
          clause: '='
        }],
        account_id: this.user.userID
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('products/retrieve', parameter).then(response => {
        $('#loading').css({display: 'none'})
        if(response.data.length > 0){
          this.data = response.data
        }
      })
    },
    searchProductHandler(event){
      this.searchValue = event.target.value
      let updatedValue = this.searchValue.slice()
      let oldMessage = this.messageInput.slice()
      let index = oldMessage.lastIndexOf('@p_')
      oldMessage = oldMessage.slice(0, index + 3)
      let updatedSearchValue = oldMessage + updatedValue
      this.$emit('searchProductEvent', {searchValue: this.searchValue, updatedValue: updatedSearchValue})
    },
    selectedIdEventHandler(event){
      this.$emit('selectedIdEvent', event)
    }
  },
  computed: {
    sortedData(){
      let sorted = this.data.filter(product => {
        return (
          product.title.toLowerCase().includes(this.searchProduct.toLowerCase()) ||
          product.tags.toLowerCase().includes(this.searchProduct.toLowerCase()) ||
          product.sku.toLowerCase().includes(this.searchProduct.toLowerCase())
        )
      })
      if(sorted !== null && sorted.length > 0){
        this.selectedId = sorted[0].id
      }
      return sorted
    }
  }
}
</script>

