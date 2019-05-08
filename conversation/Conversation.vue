<template>
  <div class="holder" id="groupConversation">
    <c-header :group="group" v-if="group !== null"></c-header>
    <div class="conversation-messages-holder">
      <c-body :conversations="auth.messenger.messages" v-if="group !== null"></c-body>
    </div>
    <c-footer :group="group" v-if="group !== null"></c-footer>
  </div>
</template>
<style scoped>
.holder{
  width: 100%;
  float: left;
}
.conversation-messages-holder{
  width: 100%;
  float: left;
  height: 72vh;
  overflow-y: auto;
  display: flex;
  flex-direction: column-reverse;
  position: relative;
}
</style>
<script>
import ROUTER from '../../../../router'
import AUTH from '../../../../services/auth'
import CONFIG from '../../../../config.js'
import axios from 'axios'
export default {
  mounted(){
    this.retrieve()
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      auth: AUTH,
      id: null,
      newFlag: false
    }
  },
  components: {
    'c-header': require('components/increment/messengervue/conversation/Header.vue'),
    'c-body': require('components/increment/messengervue/conversation/Body.vue'),
    'c-footer': require('components/increment/messengervue/conversation/Footer.vue')
  },
  props: ['groupId', 'group'],
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
    retrieve(){
      if(this.groupId && this.group.new === false){
        let parameter = {
          condition: [{
            value: this.groupId,
            column: 'messenger_group_id',
            clause: '='
          }],
          sort: {
            'created_at': 'ASC'
          }
        }
        $('#loading').css({display: 'block'})
        this.APIRequest('messenger_messages/retrieve', parameter).done(response => {
          $('#loading').css({display: 'none'})
          if(response.data.length > 0){
            AUTH.messenger.messages = response.data
            this.updateUnreadToRead()
          }else{
            AUTH.messenger.messages = null
          }
        })
      }else{
        AUTH.messenger.messages = null
      }
    },
    updateUnreadToRead(){
      if(AUTH.messenger.messages !== null){
        let lastMessage = AUTH.messenger.messages[AUTH.messenger.messages.length - 1]
        if(parseInt(lastMessage.account.id) !== this.user.userID){
          // update
          let parameter = {
            id: lastMessage.id,
            status: 1
          }
          this.APIRequest('messenger_messages/update', parameter).then(response => {
            AUTH.retrieveMessages(this.user.userID, this.user.type)
          })
        }
      }
    },
    updateMobileViewFlag(flag){
      this.$parent.updateMobileViewFlag(flag)
    }
  }
}
</script>
