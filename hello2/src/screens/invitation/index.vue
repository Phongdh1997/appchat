<template>
  <view>
    <view class="add-message">
      <nb-thumbnail square :source="require('../../../images/group-bg.png')" class="img" />
      <view>
        <nb-h3 class="px-16">Group chat</nb-h3>
        <nb-text class="px-16 gray-text">Send message to everyone</nb-text>
        <nb-button 
          transparent primary class="btn"
          :onPress="()=> isSearch = true"
        >
          <nb-text class="btn-text">Search group</nb-text>
        </nb-button>
      </view>
    </view>
    <view>
      <scroll-view>
        <nb-list-item icon v-for="invitation in invitations">
          <nb-left>
            <nb-button :style="{ backgroundColor: 'blue' }">
              <nb-icon active name="md-person-add" />
            </nb-button>
          </nb-left>
          <nb-body>
            <nb-text>{{invitation.groupChat.name}}</nb-Text>
            <nb-text v-if="invitation.type == 'invite'">
              Vào nhóm tui nè :)))</nb-text>
            <nb-text v-else>cho tui vào với, please!!!</nb-text>
          </nb-body>
          <nb-right>
            <nb-button transparent
              :onPress="()=>acceptInvitation(invitation)"
            >
              <nb-icon name="md-checkmark" />
            </nb-button>
            <nb-button transparent :onPress="()=>deleteInvitation(invitation.id)">
              <nb-icon name="md-trash" />
            </nb-button>
          </nb-right>
        </nb-list-item>
      </scroll-view>
    </view>

    <dialog 
        :visible="isSearch"
        title="Result"
        :onTouchOutside="()=> { isSearch = false; }" >
      <view>
        <nb-form class="input-container">
          <nb-item class="add-input">
            <nb-input block placeholder="Search User" v-model="searchInput" />
          </nb-item>
          <nb-button class="search-btn" primary :onPress="searchGroup">
            <nb-text>Search</nb-text>
          </nb-button>
        </nb-form>

        <view class="search-result">
          <scroll-view>
            <nb-list-item icon v-for="group in searchResults">
              <nb-left>
                <nb-button :style="{ backgroundColor: 'blue' }">
                  <nb-icon active name="md-people" />
                </nb-button>
              </nb-left>
              <nb-body>
                <nb-text>{{group.name}}</nb-Text>
              </nb-body>
              <nb-right>
                <nb-button transparent primary
                  v-if="isHost" transparent :onPress="()=> deleteGroup(group)"
                >
                  <nb-text>Remove</nb-text>
                </nb-button>
                <nb-button transparent primary
                  v-else-if="isInGroup(group) == 'IN_GROUP'"
                  :onPress="()=> unJoin(group)"
                >
                  <nb-text>Unjoin</nb-text>
                </nb-button>
                <nb-button transparent primary
                  v-else transparent :onPress="()=> join(group)"
                >
                  <nb-text>Join</nb-text>
                </nb-button>
              </nb-right>
            </nb-list-item>
          </scroll-view>
        </view>
        <nb-text>{{errorMessage}}</nb-text>
        <view class="right-content">
          <nb-button info :onPress="()=> isSearch = false">
            <nb-text>Close</nb-text>
          </nb-button>
        </view>
      </view>
    </dialog>
  </view>
</template>

<script>
  import firebase from 'firebase'
  import { ProgressDialog, Dialog } from 'react-native-simple-dialogs';

	export default {
		name: 'Invitation',
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
        currUser: null,
        error: '',
        invitations: [],
        searchInput: '',
        isSearch: false,
        isHost: false,
        searchResults: [],
        errorMessage: ''
      }
    },
    methods: {
      isInGroup(group) {
        var userId = this.currUser.uid
        if (userId == group.host.user.userId) {
          this.isHost = true
          return "IS_HOST"
        };
        var members = group.members
        for (member in members) {
          if (userId == members[member].user.userId) {
            return "IN_GROUP"
          };
        };
        return "NOT_IN_GROUP"
      },
      deleteGroup (group) {
        var members = group.members
        for (member in members) {
          this.deleteRegisterGroup(group.id, member)
        };
        this.deleteRegisterGroup(group.id, group.host.user.userId)
        firebase.database().ref('groupChats/' + group.id).remove()
        this.searchResults = []
      },
      deleteMemberInGroupChats(groupId, uid) {
        let url = 'groupChats/' + groupId + '/members/' + uid
        firebase.database().ref(url).remove()
      },
      deleteRegisterGroup(groupId, uid) {
        url = 'users/' + uid + '/groupChats/' + groupId
        firebase.database().ref(url).remove()
      },
      unJoin (group) {
        this.deleteMemberInGroupChats(group.id, this.currUser.uid)
        this.deleteRegisterGroup(group.id, this.currUser.uid)
        this.searchResults = []
      },
      join (group) {
        self = this
        firebase.database().ref('users/' + self.currUser.uid)
        .once('value', function(snapshot) {
          if (snapshot.exists()) {
            var user = snapshot.val()
            let url = 'users/' + group.host.user.userId + '/invitations'
            firebase.database().ref(url)
            .once('value', function(snapshot) {
              if (snapshot.exists()) {
                var invitations = snapshot.val()
                for (idx in invitations) {
                  if (invitations[idx].user.userId == self.currUser.uid) {
                    self.errorMessage = "sent/invited..."
                    return
                  }
                }
              }
              // if not invited
              let url = 'users/' + group.host.user.userId + '/invitations'
              var newChild = firebase.database().ref(url).push()
              let key = newChild.key
              newChild.set({
                type: 'join',
                groupChat: {
                  id: group.id,
                  name: group.name
                },
                id: key,
                user: user
              })
              .then(()=>{
                self.errorMessage = "sent ..."
              }).catch(function(error) {
                self.errorMessage = "fail ..."
              });
            });
          } else {
            self.errorMessage = "You haven't updated Info yet"
          };
        });
      },
      searchGroup () {
        self = this
        this.errorMessage = ''
        var ref = firebase.database().ref('groupChats')
        ref.orderByChild('name').equalTo(this.searchInput)
        .once("value", function(snapshot) {
          var list = []
          if (snapshot.exists()) {
            var temp = snapshot.val()
            for (idx in temp) {
              list.push(temp[idx])
            }
          }
          self.searchResults = list
          self.searchInput = ''
        });
      },
      deleteInvitation (id) {
        let url = 'users/' + self.currUser.uid + '/invitations/' + id
        firebase.database().ref(url).remove().then(()=>{
          alert("delete success")
        }).catch((error)=>{
          alert("delete fail")
        });
      },
      acceptInvitation(invitation) {
        self = this
        let url = 'groupChats/' + invitation.groupChat.id + '/members/' + invitation.user.userId
        var newChild = firebase.database().ref(url)
        newChild.set({user: invitation.user}).then(()=>{
          let url = 'users/' + invitation.user.userId + '/groupChats/' + invitation.groupChat.id
          let child = firebase.database().ref(url)
          let data = {
            id: invitation.groupChat.id,
            name: invitation.groupChat.name
          };
          child.set(data).then(()=>{
            alert("success")
            if (self.$parent.groupChats != null) {
              var check = true
              for (i = 0; i < self.$parent.groupChats.length; i++) {
                if (self.$parent.groupChats[i].id == invitation.groupChat.id) {
                  check = false
                }
              }
              if (check) {
                self.$parent.groupChats.push(data)
              }
            }
            let url = 'users/' + self.currUser.uid + '/invitations/' + invitation.id
            firebase.database().ref(url).remove()
          })
          .catch((error)=>{
            alert("fail")
          })
        }).catch(function(error) {
          alert(error)
        });
      },
      listenInvitations () {
        let url = 'users/' + this.currUser.uid + '/invitations'
        firebase.database().ref(url)
        .on('value', function(snapshot) {
          var temp = []
          if (snapshot.exists()) {
            var invitations = snapshot.val()
            for (idx in invitations) {
              temp.push(invitations[idx])
            }
          }
          this.invitations = temp
        }.bind(this));
      }
    },
    created() {
      this.currUser = firebase.auth().currentUser;
      if (this.currUser == null) {
        this.navigation.navigate("Login")
      } else {
        this.listenInvitations()
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
</style>