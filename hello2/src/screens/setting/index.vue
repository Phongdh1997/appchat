<template>
	<view>
		<text>setting</text>
		<text>setting</text>
		<text>data:</text>
		<text v-for="data in datas">{{data}}</text>
	</view>
</template>

<script>
	import firebase from 'firebase'
	export default {
		name: "SettingScreen",
		props: {
			i: Number
		},
		data () {
			return {
				datas: []
			}
		},
		created() {
			
		}
  };

</script>

<style>

</style>