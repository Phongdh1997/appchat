<template>
  <nb-container class="container">
    <image class="background-img"
          :source="require('../../../images/login-bg.jpg')" />
    <view class="background"></view>
    <view class="form-container">
      <nb-h1 class="header">CREATE A NEW ACCOUNT</nb-h1>
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
      <nb-button block info :onPress="signup" class="login-btn">
        <nb-text>SIGN UP</nb-text>
      </nb-button>
      <nb-button transparent info :onPress="toSignIn" >
        <nb-text class="orange-text">Already have an account?</nb-text>
      </nb-button>
    </view>
    <!-- <progress-dialog 
      :visible="loading" 
      title="Logining" 
      message="Please, wait..." /> -->
  </nb-container>
</template>

<script>
  import firebase from 'firebase'
	export default {
		name: 'Signup',
    props: {
      navigation: {
        type: Object
      }
    },
    data() {
      return {
        email: '',
        password: '',
        signupError: false,
        loading: false,
        firebase
      }
    },
    methods: {
      toSignIn () {
        this.navigation.navigate('Login')
      },
      signup() {
        loading = true
        self = this
        firebase.auth().createUserWithEmailAndPassword(this.email, this.password)
        .then(()=>{
          self.loading = false
          self.navigation.navigate('Login')
        }).catch(function(error) {
          self.loading = false
          var errorCode = error.code
          switch(errorCode) {
            case 'auth/invalid-email':
              alert("Email chưa chính xác")
              break;
            case 'auth/email-already-in-use':
              alert("Email đã được sử dụng")
              break;
            case 'auth/weak-password':
              alert("Password quá yếu")
            default:
              alert("Kết nối thất bại!!")
          }
        });
      }
    }
  };

</script>

<style>
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
    margin-bottom: 5px;
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