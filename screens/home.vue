<template>
  <scroll-view v-else ref="topview" :content-container-style="contentContainerStyle">
    <view class="top">
      <bar-code-scanner :type="type" :on-bar-code-scanned="onScanned" />
      <view>
        <image class="logo"
               :source="{uri: 'https://tinkerkitchen.org/images/Logo - orange - url.png'}" />
      </view>
      <text class="welcome">Welcome! Hold your pass/ticket to the camera to scan</text>
      <view class="nopass-callout">
        <text class="nopass-text">Don't have a pass?</text>
        <awesome-button type="primary" :on-press="goToForm">Tap to Check In</awesome-button>
      </view>
    </view>
    <image class="qr-code"
           :source="{uri: 'https://tinkerkitchen.org/images/Check in QR code.png'}"/>
  </scroll-view>
</template>

<script>
import AwesomeButton from 'react-native-really-awesome-button/src/themes/blue';
import { BarCodeScanner } from 'expo-barcode-scanner';
import * as Permissions from 'expo-permissions';

export default {
  props: {
    navigation: {
      type: Object
    }
  },
  components: {
    AwesomeButton,
    BarCodeScanner,
  },
  data() {
    return {
      hasCameraPermission: null,
      type: BarCodeScanner.Constants.Type.front,
      data: null,
    };
  },
  async mounted() {
    const { status } = await Permissions.askAsync(Permissions.CAMERA);
    this.hasCameraPermission = status;
  },
  computed: {
    contentContainerStyle() {
      return {
        alignItems: 'center',
        backgroundColor: 'white',
        flex: 1,
      };
      // justify-content: space-evenly;

    },
  },
  methods: {
    onScanned({ type, data }) {
      if (this.data) return;

      this.data = data;
      alert(`Scanned type: ${type}\nValue: ${data}`);
    },

    goToForm() {
      this.navigation.navigate('Form');
    },

    clearData() {
      this.data = null;
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
