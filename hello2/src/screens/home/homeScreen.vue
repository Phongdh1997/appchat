<template>
  <nb-container>
  	<nb-header hasTabs>
      <nb-left>
        <nb-button transparent :onPress="showNav">
          <nb-icon name="menu" />
        </nb-button>
      </nb-left>
      <nb-body>
        <nb-title>Home</nb-title>
      </nb-body>
      <nb-right/>
    </nb-header>
    <nb-tabs tabBarPosition="bottom">
      <nb-tab heading="Message">
        <message-box :navigation="navigation"></message-box>
      </nb-tab>
      <nb-tab heading="Group">
        <group-chat :navigation="navigation"></group-chat>
      </nb-tab>
      <nb-tab heading="Invitation">
        <invitation :navigation="navigation"></invitation>
      </nb-tab>
      <nb-tab heading="Search">
      	<search-screen :navigation="navigation"></search-screen>
      </nb-tab>
    </nb-tabs>
  </nb-container>
</template>

<script>

	import React from 'react'
	import MessageBox from '../message_box/index.vue'
	import LoginForm from '../login/loginForm.vue'
	import AboutMe from '../about_me/aboutMe.vue'
	import firebase from 'firebase'
	import SearchScreen from '../search/index.vue'
	import GroupChat from '../group/index.vue'
	import Invitation from '../invitation/index.vue'

	export default {
		name: 'HomeScreen',
		props: {
			navigation: {
				type: Object
			}
		},
		data () {
			return {
				isLogin : false,
				currUser: {
					type: Object
				}
			}
		},
		components: {
			MessageBox,
			SearchScreen,
			GroupChat,
			Invitation
		},
		created() {
			console.log("created")
			this.currUser = firebase.auth().currentUser
			if (this.currUser == null) {
		        this.navigation.navigate("Login")
		    }
		},
		methods: {
			showNav() {
				this.navigation.navigate('DrawerOpen')
			}
		}
  	};

</script>

<style>
</style>