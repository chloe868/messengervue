<template>
  <div>
    <div class="messenger-header">
      <label><b>****{{auth.messenger.title.substr(55, 43)}}</b><i class="fa fa-times pull-right" @click="close()"></i></label>
    </div>
    <div class="conversation-content">
      <div class="message-holder">
        <messages :data="auth.messenger.messages"  v-if="auth.messenger.messages !== null"></messages>
      </div>
      <div class="input-holder">
        <send :flag="flag" :groupId="groupId"></send>
      </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.messenger-header{
  width: 100%;
  float: left;
  height: 50px;
  border-top-right-radius: 10px;
  border-top-left-radius: 10px;
  line-height: 50px;
  padding-left: 10px;
  padding-right: 10px;
}

.messenger-header label{
  width: 100%;
}

.messenger-header i{
  font-size: 24px;
  margin-top: 13px;
}

.messenger-header i:hover{
  cursor: pointer;
  color: $primary;
}

.conversation-content{
  height: 275px;
  width: 100%;
  float: left;
  background: #fff;
}

.message-holder{
  height: 225px;
  overflow-y: auto;
  width: 100%;
  float: left;
  background: #fff;
  display: flex;
  flex-direction: column-reverse;
}

.input-holder{
  height: 50px;
  width: 100%;
  float: left;
  background: #fff;
  border-top: solid 1px #ddd;
}
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import COMMON from 'src/common.js'
import axios from 'axios'
export default {
  mounted(){
    this.retrieve()
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      common: COMMON,
      data: null,
      groupId: null,
      flag: true,
      auth: AUTH
    }
  },
  components: {
    'send': require('components/increment/messengervue/overlay/Send.vue'),
    'messages': require('components/increment/messengervue/overlay/Messages.vue')
  },
  watch: {
    groupId: function(newVal, oldVal) { // watch it
      this.groupId = newVal
      this.retrieve()
    }
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    close(){
      AUTH.messenger.messages = []
      AUTH.messenger.title = null
      AUTH.messenger.messengerGroupId = null
      AUTH.messenger.group = null
      AUTH.messenger.badge = 0
      AUTH.messenger.data = null
    },
    retrieve(){
      if(AUTH.messenger.group === null){
        let parameter = {
          condition: [{
            column: 'title',
            value: AUTH.messenger.data.code,
            clause: '='
          }]
        }
        this.APIRequest('messenger_groups/retrieve', parameter).done(response => {
          if(response.data.length > 0){
            // retrieve messages using id
            AUTH.messenger.messengerGroupId = response.data[0].id
            AUTH.messenger.group = response.data[0]
            this.retrieveMessages(response.data[0].id)
          }else{
            // create new message
            this.createNewMessage()
          }
        })
      }
    },
    createNewMessage(){
      let parameter = {
        member: AUTH.messenger.data.account_id,
        creator: this.user.userID,
        title: AUTH.messenger.data.code,
        payload: AUTH.messenger.data.id
      }
      this.APIRequest('custom_messenger_groups/create', parameter).done(response => {
        if(response.data > 0){
          AUTH.messenger.messengerGroupId = response.data
          this.retrieveMessages(response.data)
        }else{
          // Unable to create
        }
      })
    },
    retrieveMessages(id){
      let parameter = {
        condition: [{
          column: 'messenger_group_id',
          value: id,
          clause: '='
        }]
      }
      this.APIRequest('messenger_messages/retrieve', parameter).done(response => {
        if(response.data.length > 0){
          AUTH.messenger.messages = response.data
        }else{
          // create new message
          AUTH.messenger.messages = null
        }
      })
    }
  }
}
</script>
