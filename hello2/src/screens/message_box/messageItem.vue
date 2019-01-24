<template>
  <nb-grid class="container">
    <nb-col :size="15">
      <view :style="{paddingLeft: 5}" v-if="!isMe">
        <nb-button transparent :onPress="()=> viewInfo(message.userId)">
          <image
              class="avatar-img"
              :source="require('../../../images/default-avatar.png')" />
        </nb-button>
      </view>
    </nb-col>
    <nb-col :size="70">
        <nb-text
          :class="{ message: true, 'blue-bg': isMe, 'white-bg': !isMe}">
            {{message.content}}
          </nb-text>
    </nb-col>
    <nb-col :size="15">
      <view :style="{paddingLeft: 5}" v-if="isMe">
        <nb-button transparent :onPress="()=> viewInfo(message.userId)">
          <image
              class="avatar-img"
              :source="require('../../../images/default-avatar.png')" />
        </nb-button>
      </view>
    </nb-col>
  </nb-grid>
</template>

<script>
  import firebase from 'firebase'
  import React from 'react'
  import {Text} from 'react-native'
  import { ProgressDialog } from 'react-native-simple-dialogs'
	export default {
		name: 'MessageItem',
    props: {
      navigation: {
        type: Object
      },
      message: {
        type: Object
      },
      isMe: Boolean
    },
    methods: {
      viewInfo (uid) {
        console.log("navigate")
        this.navigation.navigate("MProfile", {uid})
      }
    }
  };

</script>

<style scoped>
  .container {
    flex-direction: row;

  }
  .message {
    padding: 11px 6px;
    border-radius: 5px;
    font-size: 14px;
    margin-bottom: 5px;
  }
  .blue-bg {
    background-color: #B0E3FE;
  }
  .white-bg {
    background-color: white;
  }
  .avatar-img {
    width: 40px;
    height: 40px;
    border-radius: 20px;
  }
  .container {
    margin-bottom: 10px;
  }
</style>