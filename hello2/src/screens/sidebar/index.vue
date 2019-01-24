<template>
  <nb-container class="container">
    <nb-content>
    	<view>
	      <image
	      	class="background-avatar"
	      	:source="require('../../../images/default-background-avatar.jpg')" />
	      <view class="avatar-container">
	      	<image
	      		class="avatar-img"
	      		:source="require('../../../images/default-avatar.png')" />
	      	<nb-text class="username">Họ và tên</nb-text>
	      </view>
	      <view class="shadow"></view>
	    </view>
      <view>
	      <nb-list>
	      	<nb-list-item icon v-for="route in datas" button noBorder :onPress="()=> handleRouting(route)">
		        <nb-left>
		          <nb-button :style="{ backgroundColor: 'blue' }">
		            <nb-icon active :name="route.icon" />
		          </nb-button>
		        </nb-left>
		        <nb-body>
		          <nb-text>{{route.name}}</nb-Text>
		        </nb-body>
		        <nb-right/>
		    	</nb-list-item>
	    	

		    	<nb-list-item icon button noBorder :onPress="logout">
		        <nb-left>
		          <nb-button :style="{ backgroundColor: 'blue' }">
		            <nb-icon active name="md-code-working" />
		          </nb-button>
		        </nb-left>
		        <nb-body>
		          <nb-text>Log Out</nb-Text>
		        </nb-body>
		        <nb-right/>
		    	</nb-list-item>
	    	</nb-list>
      </view>
    </nb-content>
  </nb-container>
</template>

<script>
	import firebase from 'firebase'
	export default {
		name: 'SideBarScreen',
		props: {
	    navigation: {
	      type: Object
	    }
	  },
	  data () {
	  	return {
	  		datas: [
	  			{
	  				route: 'Home',
	  				name: 'Home',
	  				icon: 'md-code-working'
	  			},
	  			{
	  				route: 'MProfile',
	  				name: 'Profile',
	  				icon: 'md-code-working'
	  			}
	  		]
	  	}
	  },
		methods: {
			handleRouting(route) {
				var uid = firebase.auth().currentUser.uid
				this.navigation.navigate(route.route, {uid})
			},
			logout() {
				self = this
				firebase.auth().signOut().then(function() {
					self.navigation.navigate('Login')
				}).catch(function(error) {
				  alert("logout fail")
				});
			}
		}
  };

</script>

<style scoped>
	.background-avatar {
		width: 100%;
		height: 180px;
		position: relative;
	}
	.avatar-container {
		position: absolute;
		top: 20px;
		left: 20px;
		z-index: 2
	}
	.shadow {
		position: absolute;
		background-color: rgba(0,0,0,0.3);
		top: 0;
		left: 0;
		bottom: 0;
		right: 0;
		z-index: 1;
	}
	.avatar-img {
		width: 80px;
		height: 80px;
		border-radius: 40px;
	}
	.username {
		margin-top: 20px;
		color: white;
		font-weight: 100;
	}
	.left {
		align-self: center;
	}
</style>