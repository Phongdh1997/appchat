<template>
	<view>
		<view class="search-container">
	    <nb-input class="search-input" regular placeholder="Nhập tên" v-model="searchInput" />
	    <nb-button block info class="search-btn" :onPress="search">
	     	<nb-text>Tìm Bạn</nb-text>
	    </nb-button>
	  </view>
    <view>
    	<scroll-view>
    		<user-item v-for="user in results" :user="user" :navigation="navigation"></user-item>
    	</scroll-view>
    </view>
	</view>
</template>

<script>

	import React from 'react'
	import firebase from 'firebase'
	import UserItem from './userItem.vue'

	export default {
		name: 'SearchScreen',
		props: {
			navigation: {
				type: Object
			}
		},
		data () {
			return {
				searchInput: '',
				results: []
			}
		},
		components: {
			UserItem
		},
		created() {
			
		},
		methods: {
			search() {
				self = this
				var ref = firebase.database().ref('users')
				ref.orderByChild('fullName').equalTo(this.searchInput).once("value", function(snapshot) {
				    var temp = snapshot.val()
					for (idx in temp) {
						self.results.push(temp[idx])
					}
				});
			}
		}
  };

</script>

<style>
	.search-container {
		flex-direction: row;
		justify-content: center;
	}
	.search-input {
		width: 70%;
	}
	.search-btn {
		width: 30%;
	}
</style>