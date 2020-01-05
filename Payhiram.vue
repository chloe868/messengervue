<template>
  <div>
    <div class="messenger-holder" v-if="groups !== null || partners !== null">
      <div class="conversation" v-bind:class="{'active-conversation': mobileViewFlag === false}">
        <conversation :groupId.sync="auth.messenger.messengerGroupId" :group="auth.messenger.group"></conversation>   
      </div>
      <div class="users" v-bind:class="{'active-users': mobileViewFlag === true}">
        <groups :groups="groups" :partners="partners" v-if="groups !== null || partners !== null"></groups>
      </div>
    </div>

    <empty-dynamic v-if="groups === null && partners === null" :icon="'fa fa-smile-o'" :iconColor="'text-primary'" :title="'Invalid transaction code.'" :action="'Wait for peers to contact you!'"></empty-dynamic>
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";

.messenger-holder{
  width: 100%;
  float: left;
  border: 1px solid #eee;
}
.conversation{
  width: 70%;
  float: left;
  min-height: 500px;
  overflow-y:hidden;
  height: 90vh;
}
.users{
  width: 30%;
  float: left;
  height: 90vh;
  overflow-y:scroll;
  border-left: solid 1px #eee;
}

.users::-webkit-scrollbar { width: 0; }
@media (max-width: 992px){
  .users{
    display: none;
    width: 100%;
    border-left: 0px;
    margin-left: 0%;
  }
  .conversation{
    width: 100%;
    display: none;
  }
  .active-conversation, .active-users{
    display: block;
  }
}
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import axios from 'axios'
export default {
  mounted(){
    AUTH.checkPlan()
    this.retrieve(true)
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      newTitle: null,
      groups: null,
      partners: null,
      selectedIndex: 0,
      mobileViewFlag: false,
      auth: AUTH
    }
  },
  props: ['params'],
  components: {
    'conversation': require('components/increment/messengervue/conversationPayhiram/Conversation.vue'),
    'groups': require('components/increment/messengervue/userlistPayhiram/Groups.vue'),
    'empty-dynamic': require('components/increment/generic/empty/EmptyDynamicIcon.vue')
  },
  watch: {
    '$route.params.username': function(){
      if(this.$route.params.username){
        let indexGroup = this.checkIfExistUsername(this.$route.params.username, this.groups)
        if(indexGroup !== null){
          this.makeActiveCard(indexGroup, 'groups')
        }else{
          let indexPartner = this.checkIfExistUsername(this.$route.params.username, this.partners)
          if(indexPartner !== null){
            this.makeActiveCard(indexPartner, 'partners')
          }
        }
      }
    }
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    create(){
      if(this.newTitle !== null || this.newTitle !== ''){
        let parameter = {
          'account_id': this.user.userID,
          'title': this.newTitle
        }
        this.APIRequest('custom_messenger_groups/create', parameter).then(response => {
          console.log(response)
        })
      }
    },
    retrieve(flag){
      let parameter = {
        account_id: this.user.userID,
        code: this.$route.params.code
      }
      this.APIRequest('custom_messenger_groups/retrieve', parameter).done(response => {
        this.groups = response.data
        this.partners = response.accounts
        if(flag === true){
          let active = 0
          if(AUTH.messenger.group !== null){
            for (var i = 0; i < response.data.length; i++) {
              let item = response.data[i]
              if(item.id === parseInt(AUTH.messenger.group.id)){
                active = i
                break
              }
            }
          }
          setTimeout(() => {
            this.makeActive(active, 'groups')
          }, 100)
        }
      })
    },
    checkIfExistUsername(username, list){
      if(list !== null){
        for (var i = 0; i < list.length; i++) {
          if(list[i].title.username === username){
            return i
          }
        }
      }
      return null
    },
    makeActiveCard(index, moduleText){
      AUTH.messenger.group = this.groups[index]
      AUTH.messenger.messengerGroupId = this.groups[index].id
      console.log('hi')
      this.retrieve(false)
    },
    makeActive(index, moduleText){
      AUTH.messenger.group = this.groups[index]
      AUTH.messenger.messengerGroupId = this.groups[index].id
    },
    updateMobileViewFlag(flag){
      this.mobileViewFlag = flag
    }
  }
}
</script>
