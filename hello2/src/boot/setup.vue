<template>
	<load-screen v-if="!isAppReady"></load-screen>
	<main-app v-else :isLogin="isLogin"></main-app>

</template>

<script>
	import Vue from 'vue-native-core'
	import { VueNativeBase } from 'native-base'
	import MainApp from '../MainApp.vue'
	import LoadScreen from '../screens/load/loadScreen.vue'
	import { user } from '../setup/profile.js'
	import firebase from 'firebase'

	import SettingScreen from '../screens/setting/index.vue'

	Vue.use(VueNativeBase);
	Vue.use(firebase);

	export default {
		name: 'Setup',
		components: {
			MainApp,
			LoadScreen,
			SettingScreen
		},
		data () {
			return {
				isAppReady: false,
				isLogin: false
			}
		},
		created: function() {
			this.setUpFirebase();
	    this.loadFonts();

  	},
  	methods: {
  		checkLogin() {
  			self = this
  			if (user.email != '-') {
  				firebase.auth().signInWithEmailAndPassword(user.email, user.password)
	        .then(()=>{
	          self.isLogin = true
	          self.isAppReady = true
	        }).catch(function(error) {
	          self.isAppReady = true;
	        });
  			} else {
  				this.isAppReady = true
  			}
  		},
	    loadFonts: async function() {
	      try {
	        this.isAppReady = false;
	        await Expo.Font.loadAsync({
	          Roboto: require("native-base/Fonts/Roboto.ttf"),
	          Roboto_medium: require("native-base/Fonts/Roboto_medium.ttf"),
	          Ionicons: require("@expo/vector-icons/fonts/Ionicons.ttf")
	        });
	        this.checkLogin()
	      } catch (error) {
	        console.log("some error occured", error);
	        this.checkLogin()
	      }
	    },
	    setUpFirebase() {
	    	// Initialize Firebase
			  var config = {
			    apiKey: "AIzaSyA3dOv6o3QQlm0noti9DV4IkgM2AZtzZkE",
			    authDomain: "chat-app-f7a68.firebaseapp.com",
			    databaseURL: "https://chat-app-f7a68.firebaseio.com",
			    projectId: "chat-app-f7a68",
			    storageBucket: "chat-app-f7a68.appspot.com",
			    messagingSenderId: "33255428934"
			  };
			  firebase.initializeApp(config);
	    },
	  }
	};

</script>
