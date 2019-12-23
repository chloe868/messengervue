<template>
  <div>
    <div class="messenger-holder" v-if="groups !== null || partners !== null">
      <div class="conversation" v-bind:class="{'active-conversation': mobileViewFlag === false}">
        <conversation :groupId.sync="groupId" :group="selectedGroupData" @changeGroupEvent="changeGroupHandler($event)"></conversation>   
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
    this.retrieve(this.$route.params.username ? this.$route.params.username : '')
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      newTitle: null,
      data: null,
      groups: null,
      partners: null,
      selectedIndex: 0,
      selectedGroupData: null,
      prevModuleSelected: null,
      groupId: null,
      mobileViewFlag: false
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
          this.prevModuleSelected = 'groups'
          this.makeActiveCard(indexGroup, 'groups')
        }else{
          let indexPartner = this.checkIfExistUsername(this.$route.params.username, this.partners)
          if(indexPartner !== null){
            this.prevModuleSelected = 'partners'
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
    retrieve(username){
      let parameter = {
        account_id: this.user.userID,
        code: this.$route.params.code
      }
      this.APIRequest('custom_messenger_groups/retrieve', parameter).done(response => {
        this.groups = response.data
        this.partners = response.accounts
        this.prevModuleSelected = 'groups'
        setTimeout(() => {
          this.makeActiveCard(response.active, 'groups')
        }, 100)
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
    selectedGroup(index, moduleText){
      this.selectedIndex = index
      var i = 0
      if(moduleText === 'groups'){
        this.groupId = this.groups[index].id
        this.selectedGroupData = this.groups[index]
        AUTH.messenger.messengerGroupId = parseInt(this.groups[index].id)
        for (i = 0; i < this.$children.length; i++) {
          if(this.$children[i].$el.id === 'groupConversation'){
            this.$children[i].newFlag = false
            this.$children[i].conversations = null
          }
        }
      }
    },
    makeActiveCard(index, moduleText){
      if(this.selectedIndex !== index){
        this.groups[this.selectedIndex].flag = false
        this.groups[index].flag = true
      }
      this.prevModuleSelected = moduleText
      this.selectedGroup(index, moduleText)
    },
    updateMobileViewFlag(flag){
      this.mobileViewFlag = flag
    },
    changeGroupHandler(data){
      this.selectedGroupData = data
      this.retrieve(this.user.username)
    }
  }
}
</script>
