<template>
  <div class="holder">
    <div class="header text-primary">
      <ul>
        <li class="right" @click="active = 'primary'" :class="{'active': active === 'primary'}">View Details</li>
        <li @click="active = 'secondary'" :class="{'active': active === 'secondary'}">Other transactions</li>
      </ul>
    </div>
    <div class="item-holder">
      <ul v-if="active === 'primary'" class="primary-menus">
        <li v-for="(item, index) in menus" @click="setMenu(item)">{{item}}</li>
      </ul>
      <div class="content-holder" v-if="active === 'Product Details'">
        <product :code="data.product.code" :startDate="data.start" :endDate="data.end" v-if="auth.messenger.group.payload !== 'request' && data !== null"></product>
      </div>

      <div class="content-holder" v-if="active === 'Requirements'">
        <ul class="requirements-menus">
          <li v-for="(item, index) in common.requirementOptions" :key="index">
            {{item.title}}
            <button class="btn btn-primary pull-right">Submit</button>
            <button class="btn btn-success pull-right" style="margin-right: 5px;">Add</button>
          </li>
        </ul>
      </div>
      <div class="content-holder" v-if="active === 'Tracking'">
      </div>
      <div class="content-holder" v-if="active === 'Fund Transfer'">
      </div>
      <div v-if="active === 'secondary'" class="primary-menus">
        <m-card v-for="(group, index) in groups" :key="'A' + group.id" :group="group" :index="index" :moduleText="'groups'"></m-card>
      </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.holder{
  width: 100%;
  float: right;
}
.holder .header{
  width: 100%;
  float: left;
  height: 50px;
  border-bottom: solid 1px #eee;
}

.item-holder{
}

ul{
  width: 100%;
  list-style: none;
  margin: 0;
  padding: 0;
}

.header ul .right{
  border-right: 1px #eee solid;
}

.header ul li{
  width: 50%;
  float: left;
  line-height: 50px;
  text-align: center;
}

.active{
  text-decoration: underline;
}

.primary-menus{
  position: relative;
}
.primary-menus li{
  width: 100%;
  float: left;
  line-height: 50px;
  padding-left: 10px;
  padding-right: 10px;
}

.requirements-menus li{
  width: 100%;
  float: left;
  line-height: 50px;
  padding-left: 10px;
  padding-right: 10px;
}

.primary-menus li:hover{
  cursor: pointer;
  background: #eee;
}

.header ul li:hover{
  cursor: pointer;
  text-decoration: underline;
}
.create-new-group{
  line-height: 50px; 
}
.create-new-group:hover{
  cursor: pointer;
  color: $white;
}

.content-holder{
  width: 100%;
  min-height: 50px;
  overflow-y: hidden;
  margin: 0px;
  padding-left: 10px;
  padding-right: 10px;
}

.btn{
  margin-top: 7px;
}
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import COMMON from 'src/common.js'
export default {
  mounted(){
  },
  data(){
    return {
      user: AUTH.user,
      auth: AUTH,
      config: CONFIG,
      common: COMMON,
      active: 'primary',
      menus: ['Product Details', 'Requirements', 'Tracking', 'Fund Transfer'],
      data: null
    }
  },
  props: ['groups', 'partners'],
  components: {
    'm-card': require('components/increment/messengervue/userlistPayhiram/Card.vue'),
    'm-options': require('components/increment/messengervue/userlistPayhiram/OtherOptions.vue'),
    'product': require('components/increment/imarketvue/rental/MessengerProduct.vue')
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    setMenu(item){
      if(item === 'Product Details'){
        this.retrieveProduct()
      }
      this.active = item
    },
    makeActive(selectedIndex, moduleText){
      this.$parent.makeActiveCard(selectedIndex, moduleText)
    },
    updateModalViewFlag(flag){
      this.$parent.updateMobileViewFlag(flag)
    },
    retrieveProduct(){
      let group = AUTH.messenger.group
      if(group.payload === 'rental'){
        let parameter = {
          condition: [{
            value: group.thread,
            clause: '=',
            column: 'code'
          }]
        }
        this.APIRequest('rentals/retrieve', parameter).then(response => {
          if(response.data.length > 0){
            this.data = response.data[0]
          }else{
            this.data = null
          }
        })
      }else if(group.payload === 'installment'){
        let parameter = {
          condition: [{
            value: group.thread,
            clause: '=',
            column: 'code'
          }]
        }
        this.APIRequest('installment_requests/retrieve', parameter).then(response => {
          if(response.data.length > 0){
            this.data = response.data[0]
          }else{
            this.data = null
          }
        })
      }
    }
  }
}
</script>

