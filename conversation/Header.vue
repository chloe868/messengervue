<template>
  <div class="messenger-header-holder" v-if="group !== null">
    <img :src="config.BACKEND_URL + group.title.profile.url" class="profile" v-if="group.title.profile !== null">
    <i class="fa fa-user-circle-o" v-else></i>
    <label>{{group.title.username}}
      <!-- <span class="badge badge-primary">{{group.total_members}}</span> -->
    </label> 
    <i class="fa fa-chevron-right" @click="setMobileView()"></i>
    <i id="call-icon" class="fa fa-phone-square" aria-hidden="true" @click="callHandler"></i>
  </div>
</template>
<style scoped>
.messenger-header-holder{
  width: 100%;
  float: left;
  height: 8vh;
  padding-left: 5px;
  border-bottom: solid 1px #eee;
  padding-left: 10px;
}
.profile{
  width: 40px;
  height: 40px;
  border-radius: 50%;
  float: left;
  margin-top: 5px;
}


label{
  line-height: 8vh;
  padding-left: 10px;
  float: left;
  align: center;
}

i{
  font-size: 30px;
  line-height: 8vh;
  float: left;
  color: #22b173;
}
.fa-chevron-right{
  display: none;
  padding-right: 10px;
  float: right;
}

#call-icon {
    float: right;
    margin-right: 25px;
    font-size: 45px;
}

#call-icon:hover {
    cursor: pointer;
}

@media (max-width: 991px){
  .fa-chevron-right{
    display: block;
  }
  #call-icon {
    margin-right: 10px;
  }
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
      config: CONFIG
    }
  },
  props: ['group'],
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    setMobileView(){
      this.$parent.updateMobileViewFlag(true)
    },
    callHandler(){
      AUTH.triggerAudioCall()
    }
  }
}
</script>
