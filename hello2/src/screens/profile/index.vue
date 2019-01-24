<template>
  <nb-container class="root">
    <nb-header hasTabs>
      <nb-left>
        <nb-button transparent :onPress="back">
          <nb-icon name="arrow-back" />
        </nb-button>
      </nb-left>
      <nb-body>
        <nb-title>Profile</nb-title>
      </nb-body>
      <nb-right/>
    </nb-header>
    <nb-content>
    	<view class="container">
    		<image
	      		class="background-img"
	      		:source="require('../../../images/default-background-avatar.jpg')" />
  			<nb-grid class="profile-content">
		      <nb-col :size="25">
		      	<image
	      		class="avatar-img"
	      		:source="require('../../../images/default-avatar.png')" />
		      </nb-col>
		      <nb-col :size="75">
		      	<nb-text class="text" v-once v-if="isUpdate">{{info.fullName}}</nb-text>
		      	<nb-text class="text" v-if="!isUpdate">{{info.fullName}}</nb-text>
		      </nb-col>
		    </nb-grid>
		    <nb-button v-if="!isMe" bordered warning class="message-btn" :onPress="sendMessage">
	        <nb-text>Message</nb-text>
	      </nb-button>
		    <!-- <view class="white-egde"></view> -->
	    </view>
	    <view class="info-detail" v-if="!isUpdate">
		    <nb-list>
		    	<nb-list-item noBorder>
	        	<nb-icon active name="data.icon" />
	          <view class="info-item">
	          	<nb-text class="info">Full Name: {{info.fullName}}</nb-text>
	          </view>
	        </nb-list-item>
	        <nb-list-item noBorder>
	        	<nb-icon active name="data.icon" />
	          <view class="info-item">
	          	<nb-text class="info">Gender: {{info.gender}}</nb-text>
	          </view>
	        </nb-list-item>
	        <nb-list-item noBorder>
	        	<nb-icon active name="data.icon" />
	          <view class="info-item">
	          	<nb-text class="info">Birthday: {{info.birthDay}}</nb-text>
	          </view>
	        </nb-list-item>
	        <nb-list-item noBorder>
	        	<nb-icon active name="data.icon" />
	          <view class="info-item">
	          	<nb-text class="info">Phone: {{info.phone}}</nb-text>
	          </view>
	        </nb-list-item>
	      </nb-list>
	      <nb-button primary block :onPress="routingToUpdateScreen" v-if="isMe">
	      	<nb-text>Change Information</nb-text>
	      </nb-button>
      </view>
      <view class="info-detail" v-if="isUpdate">
      	<nb-form>
		      <nb-item inlineLabel>
		        <nb-label>Full Name:</nb-label>
		        <nb-input v-model="info.fullName" />
		      </nb-item>
		      <nb-item inlineLabel last>
		        <nb-label>Gender:</nb-label>
		        <nb-input v-model="info.gender" />
		      </nb-item>
		      <nb-item inlineLabel>
		        <nb-label>Birthday:</nb-label>
		        <nb-input v-model="info.birthDay" />
		      </nb-item>
		      <nb-item inlineLabel last>
		        <nb-label>Phone:</nb-label>
		        <nb-input v-model="info.phone" />
		      </nb-item>
		    </nb-form>
		    <nb-button primary block :onPress="onSave">
					<nb-text>Save</nb-text>
				</nb-button>
      </view>
    </nb-content>
  </nb-container>
</template>

<script>
	import firebase from 'firebase'

	export default {
		name: "ProfileScreen",
		data () {
			return {
				currUser: null,
				isUpdate: false,
				noData: true,
				toMessage: false,
				loading: false,
				isMe: false,
				isReady: false,
				me: null,
				newGroupKey: '-',
				info: {
					fullName: '-',
					gender: '-',
					birthDay: '-',
					phone: '-',
					avatarUrl: '../../../images/default-avatar.png',
					bgAvatarUrl: '../../../images/default-background-avatar.jpg',
					userId: '-',
					groups: [],
					groupChats: []
				}
			}
		},
		props: {
			navigation: {
				type: Object
			}
		},
		methods: {
			back() {
				this.navigation.goBack()
			},
			routingToUpdateScreen () {
				this.isUpdate = true
			},
			onSave () {
				this.loading = true
				self = this
				if (self.noData) {
					//insert data
					firebase.database().ref('users/' + self.currUser.uid).set(self.info)
					.then(()=>{
						self.loading = false
						self.isUpdate = false
						alert("success")
					}).catch(function(error) {
						self.loading = false
						alert("fail")
					});
				} else {
					// update data
					firebase.database().ref("users/" + self.currUser.uid)
					.update(self.info, function(error){
						self.loading = false
						if (error) {
							alert("update fail")
						} else {
							self.isUpdate = false
							alert("ok")
						};
					});
				}
			},
			sendMessage () {
				self = this
				var groupId = 'No Group'
				firebase.database().ref('users/' + this.currUser.uid)
				.once('value', function(snapshot) {
				  if (snapshot.exists()) {
				  	self.actionSend();
				  } else {
				  	alert("You haven't updated Info yet")
				  };
				});
			},
			actionSend () {
				self = this
				let url = 'users/' + this.currUser.uid + '/groups/' + this.info.userId
		        firebase.database().ref(url).once('value', function(snapshot) {
		        	if (snapshot.exists()) {
		        		var group = snapshot.val()
		        		self.navigation.navigate('Message', {group: group})
		        		return
		        	}
		        	// create group
		        	let url = 'users/' + self.currUser.uid + '/groups'
		        	let ref = firebase.database().ref(url)
		        	var newGroupKey = ref.push().key
		        	var newGroup = { [self.info.userId]:
				        				{
			     							groupId: newGroupKey,
			       							friendName: self.info.fullName,
			       							friendId: self.info.userId
			        					}
			        				}
		        	ref.set(newGroup).then(()=>{
			        	let url = 'users/' + self.info.userId + '/groups'
			        	console.log(self.me)
			        	firebase.database().ref(url)
			        	.set({[self.currUser.uid]:
			        			{
			        				groupId: newGroupKey,
	       							friendName: self.me.fullName,
	       							friendId: self.me.userId
			        			}
			        	}).then(()=> {
			        		let url = 'group/' + newGroupKey
				        	let ref = firebase.database().ref(url)
				        	let key1 = ref.push().key
				        	let key2 = ref.push().key
				        	var data = {
				        		members: {
				        			[key1]: self.currUser.uid,
				        			[key2]: self.info.userId
				        		},
				        		name: '-'
				        	};
				        	ref.set(data).then(()=>{
				        		self.navigation.navigate('Message', 
				        			{group: newGroup})
				        	});
						});
					});
		        });
			}
		},	// bug
		created () {
			var uid = this.navigation.getParam('uid', 'No Id')
			this.info.userId = uid
			this.currUser = firebase.auth().currentUser
			if (this.currUser == null) {
		        this.navigation.navigate("Login")
		    } else {
		    	this.isMe = uid == this.currUser.uid
				var starCountRef = firebase.database().ref('users/' + uid);
				self = this
				starCountRef.on('value', function(snapshot) {
				  if (snapshot.exists()) {
				  	self.info = snapshot.val()
				  	self.noData = false
				  };
				  	firebase.database().ref('users/' + self.currUser.uid)
					.once('value', function(snapshot) {
					  if (snapshot.exists()) {
					  	self.me = snapshot.val()
					  }
					  self.isReady = true
					});
				})
		    }
		}
  };

</script>

<style scoped>
	.root {
		background-color: #EEEEF6;
	}
	.container {
		height: 200px;
		width: 100%;
		position: relative;
		overflow: hidden;
	}
	.background-img {
		position: absolute;
		width: 100%;
		height: 200px;
		z-index: 1;
	}
	.profile-content {
		position: absolute;
		top: 50px;
		left: 30px;
		width: 330px;
		height: 60px;
		overflow: hidden;
		z-index: 3;
	}
	.message-btn {
		position: absolute;
		top: 100px;
		left: 120px;
		z-index: 10;
	}
	.avatar-img {
		width: 60px;
		height: 60px;
		border-radius: 30px;
	}
	.text {
		margin-top: auto;
		margin-bottom: auto;
		color: white;
		font-weight: 100;
		font-size: 22px;
		font-style: italic;
	}
	.white-egde {
		height: 60px;
		width: 180%;
		position: absolute;
		background-color: white;
		bottom: -30;
		left: -45%;
		z-index: 2;
		transform: rotate(10deg);
	}
	.info-detail {
		padding: 10px 20px;
	}
	.info-item {
		margin-left: 20px;
	}
	.info {
		color: #8D8D8D;
	}
</style>