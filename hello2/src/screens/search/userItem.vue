<template>
	<view>
		<nb-list-item thumbnail :onPress="viewInfo">
      <nb-left>
        <nb-thumbnail square :source="require('../../../images/default-avatar.png')" />
      </nb-left>
      <nb-body>
        <nb-text>{{user.fullName}}</nb-Text>
       	<nb-text note :numberOfLines="1">Its time to build a difference . .</nb-Text>
      </nb-body>
      <nb-right>
      </nb-right>
    </nb-list-item>
	</view>
</template>

<script>

	import React from 'react'
	import firebase from 'firebase'

	export default {
		name: 'UserItem',
		props: {
			navigation: {
				type: Object
			},
			user: {
				type: Object
			},
			avatarUrl: String
		},
		data () {
			return {
			}
		},
		components: {
			
		},
		created() {
			
		},
		methods: {
			viewInfo() {
				this.navigation.navigate("MProfile", {uid: this.user.userId})
			}
		}
  };

</script>

<style>

</style>