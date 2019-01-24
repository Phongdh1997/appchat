<template>
  <view>
    <nb-text v-if="!isGroupsEmty">Chat Box</nb-text>
    <view class="add-message" v-if="isGroupsEmty">
      <nb-thumbnail square :source="require('../../../images/message-bg.jpg')" class="img" />
      <view>
        <nb-h3 class="px-16">Search your friend</nb-h3>
        <nb-text class="px-16 gray-text">Easy to get message from your friend</nb-text>
        <nb-button transparent primary class="btn">
          <nb-text class="btn-text">Start</nb-text>
        </nb-button>
      </view>
    </view>
    <nb-list>
      <!-- group = {groupId: '', friendName: '', friendId: ''} -->
      <nb-list-item avatar v-for="group in groups" :onPress="()=> readMessage(group)">
        <nb-left>
          <nb-thumbnail small :source="require('../../../images/message-bg.jpg')" />
        </nb-left>
        <nb-body>
          <nb-text>{{group.friendName}}</nb-text>
          <nb-text note :numberOfLines="1">Doing what you like will always keep you happy . .</nb-Text>
        </nb-body>
        <nb-right>
          <nb-text note>3:43 pm</nb-text>
        </nb-right>
      </nb-list-item>
    </nb-list>
  </view>
</template>

<script>
  import firebase from 'firebase'
  import { ProgressDialog } from 'react-native-simple-dialogs';
	export default {

		name: 'MessageBox',
    props: {
      navigation: {
        type: Object
      }
    },
    data() {
      return {
        currUser: null,
        isGroupsEmty: true,
        groups: []
      }
    },
    watch: {
      groups() {
        if (this.groups == []) {
          this.isGroupsEmty = true
        } else {
          this.isGroupsEmty = false
        }
      }
    },
    methods: {
      readMessage(group) {
        this.navigation.navigate('Message', {group: group})
      },
      listenMessageBox(userId) {
        firebase.database().ref('users/' + userId + '/groups')
        .on('value', function(snapshot) {
          this.groups = []
          if (snapshot.exists()) {
            var datas = snapshot.val()
            for (data in datas) {
              this.groups.push(datas[data])
            }
          };
        }.bind(this));
      }
    },
    created() {
      this.currUser = firebase.auth().currentUser;
      if (this.currUser == null) {
        this.navigation.navigate("Login")
      } else {
        this.listenMessageBox(this.currUser.uid)
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
</style>