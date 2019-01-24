<template>
  <nb-container class="container">
    <nb-header>
      <nb-left>
        <nb-button transparent :onPress="back">
          <nb-icon name="arrow-back" />
        </nb-button>
      </nb-left>
      <nb-body>
        <nb-title>{{groupChat.name}}</nb-title>
      </nb-body>
      <nb-right>
        <nb-button transparent :onPress="showOption">
          <nb-icon name="ios-people" />
        </nb-button>
      </nb-right>
    </nb-header>
    <flat-list :data="messages">
      <view render-prop-fn="renderItem" >
        <message-item 
          :message="args.item"
          :isMe="currUser.uid == args.item.userId"
          :navigation="navigation">
        </message-item>
      </view>
    </flat-list>
    <view class="input-container">
      <nb-form >
        <nb-input placeholder='Nhập tin nhắn' class="message-input" v-model="content" />
      </nb-form>
      <nb-button icon class="message-btn" :onPress="sendMessage">
        <nb-icon active name="home" />
      </nb-button>
    </view>

    <!-- add member dialog -->
    <dialog 
        :visible="isAddMember"
        title="Add New Member"
        :onTouchOutside="()=> { isAddMember = false; }" >
      <view>
        <nb-form class="input-container">
          <nb-item class="add-input">
            <nb-input block placeholder="Search User" v-model="searchInput" />
          </nb-item>
          <nb-button class="search-btn" primary :onPress="searchUser">
            <nb-text>Search</nb-text>
          </nb-button>
        </nb-form>

        <view class="search-result">
          <scroll-view>
            <nb-list-item icon v-for="user in searchResults">
              <nb-left>
                <nb-button :style="{ backgroundColor: 'blue' }">
                  <nb-icon active name="md-person" />
                </nb-button>
              </nb-left>
              <nb-body>
                <nb-text>{{user.fullName}}</nb-Text>
              </nb-body>
              <nb-right>
                <nb-button transparent :onPress="()=> inviteMember(user)">
                  <nb-icon name="person-add" />
                </nb-button>
              </nb-right>
            </nb-list-item>
          </scroll-view>
        </view>
        <nb-text>{{errorMessage}}</nb-text>
        <view class="right-content">
          <nb-button info :onPress="()=> isAddMember = false">
            <nb-text>Close</nb-text>
          </nb-button>
        </view>
      </view>
    </dialog>

    <!-- view member dialog -->
    <dialog 
        :visible="isViewMember"
        title="Member List"
        :onTouchOutside="()=> {isViewMember = false;}" >
      <view>
        <view class="member-list">
          <scroll-view>
            <nb-list-item icon v-for="member in members">
              <nb-left>
                <nb-button transparent
                  :onPress="()=> viewInfo(member.user.userId)">
                  <image
                      class="avatar-img"
                      :source="require('../../../images/default-avatar.png')" />
                </nb-button>
              </nb-left>
              <nb-body>
                <nb-text>{{member.user.fullName}}</nb-Text>
              </nb-body>
              <nb-right>
                <view v-if="host.userId == member.user.userId">
                  <nb-text>Admin</nb-text>
                </view>
                <nb-button v-else-if="isHost"
                  transparent :onPress="()=>removeMember(member.user)"
                >
                  <nb-icon name="md-remove" />
                </nb-button>
                <nb-button transparent primary :onPress="()=>unJoin(member.user)"
                  v-else-if="currUser.uid == member.user.userId"
                >
                  <nb-text>Unjoin</nb-text>
                </nb-button>
                <view v-else>
                  <nb-text>Member</nb-text>
                </view>
              </nb-right>
            </nb-list-item>
          </scroll-view>
        </view>
        <nb-text>{{errorMessage}}</nb-text>
        <view class="right-content">
          <nb-button info :onPress="()=> isViewMember = false">
            <nb-text>Close</nb-text>
          </nb-button>
        </view>
      </view>
    </dialog>
  </nb-container>
</template>

<script>
  import firebase from 'firebase'
  import React from 'react'
  import {Text} from 'react-native';
  import { ProgressDialog, Dialog } from 'react-native-simple-dialogs';
  import MessageItem from '../message_box/messageItem.vue'
  import { ActionSheet } from "native-base";
  import { Toast } from "native-base";

	export default {
		name: 'GroupChatScreen',
    props: {
      navigation: {
        type: Object
      }
    },
    data () {
      return {
        groupChat: {},
        messages: [],
        content: '',
        currUser: null,
        loading: false,
        sending: false,
        sended: false,
        received: false,
        sendError: false,
        isReady: false,
        isAddMember: false,
        isViewMember: false,
        isHost: false,
        host: null,
        searchResults: [],
        members: [],
        searchInput: '',
        errorMessage: '',
        btnOptions: ["View Member", "Cancel"],
        optionCancelIndex: 4,
        optionDestructiveIndex: 3,
        clicked: 0
      }
    },
    components: {
      MessageItem,
      Dialog
    },
    methods: {
      back () {
        this.navigation.goBack() 
      },
      viewInfo (uid) {
        this.isViewMember = false
        this.navigation.navigate('MProfile', {uid})
      },
      getMessages() { // no use
        self = this
        this.loading = true
        var starCountRef = firebase.database().ref('groupChats/' + this.groupChat.id);
        starCountRef.once('value', function(snapshot) {
          var temp = snapshot.val()
          self.isHost = self.currUser.uid == temp.host.user.userId
          var messages = temp.messages
          for (idx in messages) {
            self.messages.push(temp[idx])
          }
        });
      },
      sendMessage () {
        if (this.content == '') {
          return
        }
        self = this
        this.sending = true
        var d = new Date()
        var sendDate = d.toString()
        let url = 'groupChats/' + self.groupChat.id + '/messages'
        var newChild = firebase.database().ref(url).push()
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
      cancelDialog () {
        this.isShow = false
      },
      searchUser () {
        self = this
        this.errorMessage = ''
        this.searchResults = []
        var ref = firebase.database().ref('users')
        ref.orderByChild('fullName').equalTo(this.searchInput).once("value", function(snapshot) {
          var temp = snapshot.val()
          for (idx in temp) {
            self.searchResults.push(temp[idx])
          }
        });
      },
      isExists (userId, members) {
        if (userId == this.currUser.uid) {
          return true
        }
        for (i = 0; i < members.length; i++) {
          if (members[i].user.userId == userId) {
            return true
          }
        }
        return false
      },
      inviteMember (user) {
        self = this
        if (self.isExists(user.userId, self.members)) {
          self.errorMessage = "This user had already been a member"
          return
        }
        self = this
        firebase.database().ref('users/' + user.userId)
        .once('value', function(snapshot) {
          if (snapshot.exists()) {
            let url = 'users/' + user.userId + '/invitations'
            firebase.database().ref(url)
            .once('value', function(snapshot) {
              if (snapshot.exists()) {
                var invitations = snapshot.val()
                for (idx in invitations) {
                  if (invitations[idx].groupChat.id == self.groupChat.id) {
                    self.errorMessage = "this user have already invited..."
                    return
                  }
                }
              }
              // if not invited
              let url = 'users/' + user.userId + '/invitations'
              var newChild = firebase.database().ref(url).push()
              let key = newChild.key
              newChild.set({
                type: 'invite',
                groupChat: self.groupChat,
                id: key,
                user: user
              })
              .then(()=>{
                self.errorMessage = "Invited ..."
              }).catch(function(error) {
                self.errorMessage = "fail ..."
              });
            });
          } else {
            self.errorMessage = "this user haven't updated Info yet"
          };
        });
      },
      deleteGroup () {
        self = this
        var members = this.members
        for (i = 0; i < members.length; i++) {
          this.deleteRegisterGroup(self.groupChat.id, members[i].user.userId)
        };
        this.deleteRegisterGroup(self.groupChat.id, self.host.userId)
        firebase.database().ref('groupChats/' + self.groupChat.id).remove()
        .then(()=>{
          self.navigation.navigate('Home')  // navigate
        });
      },
      deleteMemberInGroupChats(groupId, uid) {
        let url = 'groupChats/' + groupId + '/members/' + uid
        return firebase.database().ref(url).remove()
      },
      deleteRegisterGroup(groupId, uid) {
        url = 'users/' + uid + '/groupChats/' + groupId
        return firebase.database().ref(url).remove()
      },
      removeMember (user) {
        self = this
        return new Promise((s,f)=>{
          self.deleteMemberInGroupChats(self.groupChat.id, user.userId)
          .then(()=> {
            self.deleteRegisterGroup(self.groupChat.id, user.userId)
            .then(()=>{
              self.errorMessage = "removed"
              s()
            });
          });
        });
      },
      unJoin (user) {
        self = this
        this.removeMember(user).then(()=>{
          self.isViewMember = false
          self.navigation.navigate('Home')  // navigate
        });
      },
      handleFabIconPress () {
        this.isFabIconActive = !this.isFabIconActive;
      },
      showOption () {
        ActionSheet.show(
          {
            options: this.btnOptions,
            cancelButtonIndex: this.optionCancelIndex,
            destructiveButtonIndex: this.optionDestructiveIndex,
            title: "Select An Option"
          },
          buttonIndex => {
            this.handleOptionClick(this.btnOptions[buttonIndex])
          }
        );
      },
      handleOptionClick(option) {
        if (option == 'View Member') {
          this.errorMessage = ''
          this.isViewMember = true
        } else if (option == 'Add Member') {
          if (!this.isHost) {
            alert("You are not the admin")
          } else {
            this.errorMessage = ''
            this.isAddMember = true
          }
        } else if (option == 'Destroy Group') {
          this.deleteGroup()
        }
      },
      listenMessages() {
        let url = 'groupChats/' + this.groupChat.id + '/messages'
        var starCountRef = firebase.database().ref(url);
        starCountRef.on('child_added', function(childSnapshot, prevChildKey) {
          if (childSnapshot.exists()) {
              this.messages.push(childSnapshot.val())
          }
        }.bind(this));
      },
      listenMembers() {
        var url = 'groupChats/' + this.groupChat.id + '/members'
        firebase.database().ref(url)
        .on('value', function(snapshot) {
          var temp = [{user: this.host}]
          if (snapshot.exists()) {
            var members = snapshot.val()
            for (idx in members) {
              temp.push(members[idx])
            }
          }
          this.members = temp
        }.bind(this));
      },
      checkHost () {
        self = this
        let url = 'groupChats/' + this.groupChat.id + '/host/user'
        var starCountRef = firebase.database().ref(url);
        var promise = starCountRef.once('value', function(snapshot) {
          if (snapshot.exists()) {
            var user = snapshot.val()
            self.members.push({user})
            self.isHost = self.currUser.uid == user.userId
            self.host = user
            self.isReady = true
            if (self.isHost) {
              self.btnOptions.unshift("Add Member")
              self.btnOptions.unshift("Destroy Group")
            }
          }
        });
        return promise
      }
    },
    created() {
      this.groupChat = this.navigation.getParam('groupChat', 'No Id')
      this.currUser = firebase.auth().currentUser;
      if (this.currUser == null) {
        this.navigation.navigate("Login")
      } else {
        this.checkHost().then(()=>{
          this.listenMessages()
          this.listenMembers()
        });
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
  .add-input {
    flex: 2;
  }
  .search-btn {
    flex: 1;
  }
  .message-input {
    width: 280px;
  }
  .messgae-btn {
    width: 50px;
    color: blue;
    background-color: white;
  }
  .search-result {
    max-height: 200px;
  }
  .member-list {
    max-height: 250px;
  }
  .right-content {
    flex-direction: row;
    justify-content: flex-end;
  }
  .avatar-img {
    width: 40px;
    height: 40px;
    border-radius: 20px;
  }
</style>