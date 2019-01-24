<template>
  <nb-container class="container">
    <image class="background-img"
          :source="require('../../../images/login-bg.jpg')" />
    <view class="background"></view>
    <view class="form-container">
      <nb-h1 class="header">WELCOME</nb-h1>
      <nb-form class="form">
        <nb-item floatingLabel class="input-item">
          <nb-label class="float-text">Email</nb-label>
          <nb-input class="input" v-model="email" />
        </nb-item>
        <nb-item floatingLabel class="input-item">
          <nb-label class="float-text">Password</nb-label>
          <nb-input class="input" v-model="password" :secureTextEntry="true" />
        </nb-item>
      </nb-form>
      <nb-button block info :onPress="login" class="login-btn">
        <nb-text>SIGN IN</nb-text>
      </nb-button>
      <view class="text-btn-container">
        <nb-button transparent info :onPress="forgotPassword" >
          <nb-text class="white-text">Forgot Password?</nb-text>
        </nb-button>
        <nb-button transparent info :onPress="redirectToSignup" >
          <nb-text class="orange-text">Don't have an account?</nb-text>
        </nb-button>
      </view>
    </view>
    <progress-dialog 
      :visible="loading" 
      title="Logining" 
      message="Please, wait..." />
  </nb-container>
</template>

<script>
  import firebase from 'firebase'
  import { ProgressDialog } from 'react-native-simple-dialogs';
	export default {
		name: 'LoginForm',
    props: {
      navigation: {
        type: Object
      }
    },
    components: {
      ProgressDialog
    },
    data() {
      return {
        email: '',
        password: '',
        loading: false,
        firebase
      }
    },
    methods: {
      login() {
        this.loading = true
        self = this
        firebase.auth().signInWithEmailAndPassword(this.email, this.password)
        .then(()=>{
          self.loading = false
          self.navigation.navigate('Home')
        }).catch(function(error) {
          self.loading = false
          alert("Email hoặc password không đúng")
        });
      },
      redirectToSignup () {
        this.navigation.navigate('Signup')
      },
      forgotPassword () {
        this.navigation.navigate('ForgotPassword')
      }
    }
  };

</script>

<style>
  .container {
    flex: 1;
  }
  .content {
    flex: 1;
  }
  .background {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: rgba(36,89,109,0.9);
    z-index: 2;
  }
  .background-img {
    width: 100%;
    z-index: 1;
  }
  .form-container {
    position: absolute;
    width: 80%;
    top: 100px;
    left: 10%;
    z-index: 100;
  }
  .login-btn {
    border-radius: 5px;
    margin-top: 20px;
    background-color: rgba(26,172,228,0.7);
  }
  .input-item {
    margin-top: 15px;
    margin-left: 0px;
    margin-bottom: 0px;
    margin-right: 0px;
  }
  .input {
    color: white;
    font-weight: 100;
  }
  .float-text {
    color: white;
    font-weight: 100;
  }
  .header {
    text-align: center;
    font-size: 30px;
    color: white;
  }
  .white-text {
    color: #EEF0ED;
  }
  .orange-text {
    color: #E2793B;
  }
</style>