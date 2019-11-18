<template>
  <scroll-view v-else ref="topview" :content-container-style="contentContainerStyle">
    <view class="top">
      <bar-code-scanner :type="type" :on-bar-code-scanned="onScanned" />
      <image class="logo" :source="logoImg" />
      <text class="welcome">Welcome! Hold your pass/ticket to the camera to scan</text>
      <view class="form">
        <text class="h3">No pass? No worries! Fill out your name & email:</text>
        <text class="h4 mb-4">(no spam, we promise!)</text>

        <text-input v-model="name" placeholder="Full Name" class="text-box mb-4" :on-change-text="readyNext" />
        <text-input v-model="email" placeholder="Email" class="text-box mb-4"
                    auto-capitalize="none"
                    :auto-correct="false"
                    keyboard-type="email-address"
                    auto-complete-type="email"
                    text-content-type="emailAddress"
                    :on-change-text="readyNext" />

        <button title="Next" :disabled="!ready" :on-press="next" />
      </view>
    </view>
  </scroll-view>
</template>

<script>
 import React from 'react';
 import RadioItem from '../components/radioItem';
 import AwesomeButton from 'react-native-really-awesome-button/src/themes/blue';
 import { BarCodeScanner } from 'expo-barcode-scanner';
 import * as Permissions from 'expo-permissions';
 import { gql } from 'apollo-boost';

 import logoImg from "../assets/logo.png";

 export default {
   props: {
     navigation: { type: Object },
     screenProps: { type: Object },
   },
   components: {
     AwesomeButton,
     BarCodeScanner,
     RadioItem,
   },
   data() {
     const state = this.screenProps.store.state;
     return {
       name: state.name,
       childName: state.childName,
       email: state.email,
       ready: null,
       hasCameraPermission: null,
       type: BarCodeScanner.Constants.Type.front,
       qrStop: false,
       logoImg,
     };
   },
   async mounted() {
     const { status } = await Permissions.askAsync(Permissions.CAMERA);
     this.hasCameraPermission = status;
     this.navigation.addListener('willBlur', () => this.qrStop = true);
     this.navigation.addListener('didFocus', () => this.qrStop = false);
     setTimeout(() => { this.readyNext(); }, 500);
   },
   destroyed() {
     this.qrStop = true;
   },
   computed: {
     contentContainerStyle() {
       return {
         alignItems: 'center',
         backgroundColor: 'white',
         flex: 1,
       };
     },
   },
   methods: {
     readyNext() {
       this.ready = false;
       if (!this.name || !this.email) return;
       if (this.name.split(' ').length < 1) return;
       this.ready = true;
     },

     next() {
       const store = this.screenProps.store;
       store.commit('setName', this.name);
       store.commit('setEmail', this.email);
       store.commit('setUserType', 'unknown');
       store.commit('setChildName', this.childName);
       this.navigation.navigate('Legal');
     },
     async onScanned({ type, data }) {
       if (this.qrStop) return;
       this.qrStop = true;

       try {
         const ret = await this.screenProps.client.mutate({
           mutation: gql`
             mutation checkInQrScan($qr_data: String!) {
               check_in_qr_scan(qr_data: $qr_data) {
                 type
                 name
                 email
                 user_status
                 purchase_name
                 purchase_email
                 consumable_status
               }
             }
           `,
           variables: { qr_data: data },
         });
         if (ret.data.check_in_qr_scan) {
           const store = this.screenProps.store;
           const qr = ret.data.check_in_qr_scan;
           if (qr.type === 'user') {
             store.commit('setName', qr.name);
             store.commit('setEmail', qr.email);
             store.commit('setUserType', qr.user_status);
             store.commit('setChildName', '');
             this.navigation.navigate('Legal');
           } else if (qr.type === 'daypass') {
           } else if (qr.type === 'ticket') {
           } else {
             // unknown qr code type
           }
         }
       } catch(e) {
         // do nothing on invalid/unknown qr codes, but resume scanning
         this.qrStop = false;
       }
     },

     goToForm() {
       this.reset();
       this.navigation.navigate('Form');
     },

     reset() {
       const store = this.screenProps.store;
       store.commit('reset');
     },
   },
 };
</script>

<style>
.top {
    flex: 0.8;
    align-items: center;
    justify-content: center;
}
.logo {
    width: 200;
    height: 200;
}
.welcome {
    font-size: 24;
    margin: 60;
}
.nopass-callout {
    padding: 30;
    border-color: lightgray;
    border-width: 1;
    border-radius: 5;
    align-items: center;
}
.nopass-text {
    font-size: 18;
    margin-bottom: 20;
}
.container {
    align-items: center;
    background-color: #fff;
    justify-content: flex-start;
    flex: 1;
}
.form {
    margin-top: 50;
    margin-bottom: 20;
    width: 600;
}

.h2 {
    font-size: 24;
}
.h3 {
    font-size: 20;
}
.h4 {
    font-size: 16;
}

.mb-1 { margin-bottom: 5; }
.mb-2 { margin-bottom: 10; }
.mb-3 { margin-bottom: 15; }
.mb-4 { margin-bottom: 20; }

.mt-1 { margin-top: 5; }
.mt-2 { margin-top: 10; }
.mt-3 { margin-top: 15; }
.mt-4 { margin-top: 20; }

.text-box {
    height: 60;
    border-color: gray;
    border-width: 1;
    padding: 10;
    border-radius: 3;
    font-size: 22;
}
.item-wrapper {
    height: 40;
    padding-left: 16;
}
.item-inner {
    align-items: center;
    border-bottom-color: lightgray;
    border-bottom-width: 1;
    flex: 1;
    flex-direction: row;
    justify-content: space-between;
}
.left {
    font-size: 16;
}
.right {
}
</style>
