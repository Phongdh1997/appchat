<template>
  <view>
    <view class="add-message">
      <nb-thumbnail square :source="require('../../../images/group-bg.png')" class="img" />
      <view>
        <nb-h3 class="px-16">Group chat</nb-h3>
        <nb-text class="px-16 gray-text">Send message to everyone</nb-text>
        <nb-button transparent primary class="btn" :onPress="showCreateForm">
          <nb-text class="btn-text">New group</nb-text>
        </nb-button>
      </view>
    </view>
    <nb-list>
      <nb-list-item
        avatar
        v-for="groupChat in groupChats"
        :onPress="()=>viewGroup(groupChat)">
        <nb-left>
          <nb-thumbnail small :source="require('../../../images/group-avatar.png')" />
        </nb-left>
        <nb-body>
          <nb-text>{{groupChat.name}}</nb-text>
          <nb-text note :numberOfLines="1">Send message everywhere</nb-Text>
        </nb-body>
        <nb-right>
          <nb-text note>3:43 pm</nb-text>
        </nb-right>
      </nb-list-item>
    </nb-list>
    <dialog 
        :visible="isCreate"
        title="Create a new Group"
        :onTouchOutside="cancel" >
      <view class="create-form">
        <nb-form>
          <nb-item>
            <nb-input placeholder="Enter group name" v-model="groupName" />
          </nb-item>
        </nb-form>
        <nb-text>{{error}}</nb-text>
        <view class="group-btn">
          <nb-button bordered success :onPress="createGroup" class="inline-btn">
            <nb-text>Create</nb-text>
          </nb-button>
          <nb-button block danger :onPress="cancel">
            <nb-text>Cancel</nb-text>
          </nb-button>
        </view>
      </view>
    </dialog>
  </view>
</template>

<script>
  import firebase from 'firebase'
  import { ProgressDialog, Dialog  } from 'react-native-simple-dialogs';

	export default {
		name: 'GroupChat',
    props: {
      navigation: {
        type: Object
      }
    },
    components: {
      Dialog
    },
    data() {
      return {
        groupChats: [],
        currUser: null,
        groupName: '',
        error: '',
        creating: false,
        isCreate: false
      }
    },
    watch: {
    },
    methods: {
      showCreateForm() {
        this.isCreate = true
      },
      cancel() {
        this.isCreate = false
      },
      createGroup () {
        if (this.groupName == '') {
          this.error = "Let you enter group name"
          return
        }
        this.creating = true
        self = this
        firebase.database().ref('users/' + this.currUser.uid)
        .once('value', function(snapshot) {
          if (snapshot.exists()) {
            self.actionCreate(snapshot.val());
            self.isCreate = false
          } else {
            alert("You haven't updated Info yet")
          };
        });
      },
      actionCreate(userInfo) {
        self = this
        let child = firebase.database().ref('groupChats').push()
        var key = child.key
        let data = {
          host: {
            user: userInfo
          },
          name: self.groupName,
          id: key
        }
        child.set(data)
        .then(()=>{
          let url = 'users/' + self.currUser.uid + '/groupChats/' + key
          let child = firebase.database().ref(url)
          let data = {
            id: key,
            name: self.groupName
          }
          child.set(data)
          .then(()=>{
            alert("success")
            self.creating = false
            self.isCreate = false
          })
          .catch((error)=>{
            alert("fail")
            self.creating = false
            self.isCreate = false
          })
        });
      },
      viewGroup(groupChat) {
        this.navigation.navigate('GroupChat', {groupChat})
      },
      getGroups(userId) {
        self = this
        firebase.database().ref('users/' + userId + '/groupChats')
        .once('value',function(snapshot) {
          if (snapshot.exists()) {
            var datas = snapshot.val()
            for (data in datas) {
              self.groupChats.push(datas[data])
            }
          };
        });
      },
      listenGroupChats() {
        let url = 'users/' + this.currUser.uid + '/groupChats'
        var starCountRef = firebase.database().ref(url);
        starCountRef.on('value', function(snapshot) {
          this.groupChats = []
          if (snapshot.exists()) {
            var groups = snapshot.val()
            for (idx in groups) {
              this.groupChats.push(groups[idx])
            }
          }
        }.bind(this));
      }
    },
    created() {
      this.currUser = firebase.auth().currentUser;
      if (this.currUser == null) {
        this.navigation.navigate("Login")
      } else {
        this.listenGroupChats()
      }

    }
  };

</script>

<style>
  .add-message {
    flex-direction: row;
    padding: 15px;
  }
  .btn {
    color: #60C2E8;
  }
  .px-16 {
    padding-left: 16px;
  }
  .img {
    width: 60px;
  }
  .gray-text {
    color: #B0B4B4;
  }
  .blue-text {
    color: #60C2E8;
  }
  .create-form {
    width: 100%;
    background-color: white;
  }
  .group-btn {
    flex-direction: row;
    justify-content: center;
  }
  .inline-btn {

  }
</style>