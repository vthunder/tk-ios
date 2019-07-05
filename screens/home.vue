<template>
  <scroll-view v-else ref="topview" :content-container-style="contentContainerStyle">
    <view class="top">
      <bar-code-scanner :type="type" :on-bar-code-scanned="onScanned" />
      <view>
        <image class="logo" :source="logoImg" />
      </view>
      <text class="welcome">Welcome! Hold your pass/ticket to the camera to scan</text>
      <view class="nopass-callout">
        <text class="nopass-text">Don't have a pass?</text>
        <awesome-button type="primary" :on-press="goToForm">Tap to Check In</awesome-button>
      </view>
    </view>
  </scroll-view>
</template>

<script>
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
  },
  data() {
    return {
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
.qr-code {
    position: absolute;
    height: 170;
    width: 120;
    left: 50;
    bottom: 50;
    resizeMode: stretch;
}
</style>
