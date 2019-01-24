<template>
	<root>
  		<app-navigation></app-navigation>
 	</root>
</template>

<script>
	import React from 'react'
	import { Root } from "native-base";
	import { Text } from 'react-native'
	import SideBarScreen from "./screens/sidebar/index.vue";
	import { StackNavigator, DrawerNavigator } from "vue-native-router";
	import LoginForm from './screens/login/loginForm.vue'
	import Signup from './screens/signup/signup.vue'
	import HomeScreen from './screens/home/homeScreen.vue'
	import ProfileScreen from './screens/profile/index.vue'
	import SettingScreen from './screens/setting/index.vue'
	import MessageScreen from './screens/message_box/message.vue'
	import GroupChatScreen from './screens/group/groupChat.vue'
	import ForgotPassword from './screens/forgot_password/index.vue'
	import MessageItem from './screens/message_box/messageItem.vue'

	const Drawer = DrawerNavigator(
	  {
	    Home: { screen: HomeScreen},
	    MSetting: { screen: SettingScreen },
	    MProfile: { screen: ProfileScreen }
	  },
	  {
	    initialRouteName: "Home",
    	contentComponent: props => {
      	return <SideBarScreen {...props} />;
    	}
	  }
	);

	const AppNavigation = StackNavigator(
	  {
	    Drawer: Drawer,
	    Login: { screen: LoginForm },
	    Signup: { screen: Signup },
	    Message: { screen: MessageScreen },
	    MessageItem: { screen: MessageItem },
	    GroupChat: { screen: GroupChatScreen },
	    ForgotPassword: { screen: ForgotPassword }
	  },
	  {
	    initialRouteName: 'Drawer',
	   	headerMode: "none"
	  }
	);

	export default {
		name: 'MainApp',
		components: {
			AppNavigation,
			Root
		},
		props: {
			isLogin: Boolean
		}
  };

</script>

<style>
</style>