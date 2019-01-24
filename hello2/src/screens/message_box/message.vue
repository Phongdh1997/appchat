<template>
  <nb-container class="container">
    <nb-header>
      <nb-left>
        <nb-button transparent :onPress="back">
          <nb-icon name="arrow-back" />
        </nb-button>
      </nb-left>
      <nb-body>
        <nb-title>{{group.friendName}}</nb-title>
      </nb-body>
      <nb-right>
        <nb-button transparent>
          <nb-icon name="menu" />
        </nb-button>
      </nb-right>
    </nb-header>
    <flat-list :data="messages">
      <view render-prop-fn="renderItem" >
        <message-item 
          :navigation="navigation" 
          :message="args.item" 
          :isMe="currUser.uid == args.item.userId">    
        </message-item>
      </view>
    </flat-list>
    <nb-form class="input-container">
      <nb-input placeholder='Nhập tin nhắn' class="message-input" v-model="content" />
      <nb-button icon class="message-btn" :onPress="sendMessage">
        <nb-icon active name="home" />
      </nb-button>
    </nb-form>
  </nb-container>
</template>

<script>
  import firebase from 'firebase'
  import React from 'react'
  import {Text} from 'react-native';
  import { ProgressDialog } from 'react-native-simple-dialogs';
  import MessageItem from './messageItem.vue'

	export default {
		name: 'MessageScreen',
    props: {
      navigation: {
        type: Object
      }
    },
    data () {
      return {
        group: '',
        messages: [],
        content: '',
        currUser: null,
        loading: false,
        sending: false,
        sended: false,
        received: false,
        sendError: false
      }
    },
    components: {
      MessageItem
    },
    methods: {
      back () {
        this.navigation.goBack()
      },
      sendMessage () {
        if (this.content == '') {
          return
        }
        self = this
        this.sending = true
        var newChild = firebase.database()
        .ref('group/' + self.group.groupId + '/messages').push()
        let d = new Date()
        let sendDate = d.toString()
        let key = newChild.key
        var data = {
          key: key,
          content: self.content,
          sendDate: sendDate,
          userId: self.currUser.uid
        };
        this.content = ''
        newChild.set(data)
        .then(()=>{
          self.sending = false
          self.sended = true
        }).catch(function(error) {
          self.sending = false
          self.sendError = true
        });
      },
      listenMessage() {
        let url = 'group/' + this.group.groupId + '/messages'
        var starCountRef = firebase.database().ref(url);
        starCountRef.on('child_added', function(childSnapshot, prevChildKey) {
          if (childSnapshot.exists()) {
            this.messages.push(childSnapshot.val())
          }
        }.bind(this));
      }
    },
    created() {
      this.group = this.navigation.getParam('group', 'No Id')
      this.currUser = firebase.auth().currentUser;
      if (this.currUser == null) {
        this.navigation.navigate("Login")
      } else {
        this.listenMessage()
      }
    }
  };

</script>

<style scoped>
  .container {
    background-color: #D3DAE0;
  }
  .message-input {
    height: 40px;
  }
  .input-container {
    flex-direction: row;
    justify-content: center;
    background-color: white;
  }
  .message-input {
    width: 280px;
  }
  .messgae-btn {
    width: 50px;
    color: blue;
    background-color: white;
  }
</style>