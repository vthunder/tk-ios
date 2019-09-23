<template>
  <scroll-view v-else ref="topview" :content-container-style="contentContainerStyle">
    <view class="form">
      <text class="h2">Full Name</text>
      <text-input v-model="name" class="text-box mb-4" :on-change-text="readyNext" />

      <text class="h2">Email</text>
      <text-input v-model="email" class="text-box mb-4"
                  auto-capitalize="none"
                  :auto-correct="false"
                  keyboard-type="email-address"
                  auto-complete-type="email"
                  text-content-type="emailAddress"
                  :on-change-text="readyNext" />

      <text class="h2">I am... (select one)</text>
      <radio-item name="member"
                  text="A member"
                  :selected="userTypeIs('member')" :onselect="selectUser" />
      <radio-item name="guest"
                  text="A guest of a member"
                  :selected="userTypeIs('guest')" :onselect="selectUser" />
      <radio-item name="daypass-ticket"
                  text="Using a day pass or class/event ticket"
                  :selected="userTypeIs('daypass-ticket')" :onselect="selectUser" />
      <radio-item name="child" class="mb-4"
                  text="Checking in for my child"
                  :selected="userTypeIs('child')" :onselect="selectUser" />

      <view v-if="userType === 'child'">
        <text class="h2">Your child's name</text>
        <text-input v-model="childName" class="text-box mb-4" :on-change-text="readyNext" />
      </view>

      <button title="Next" :disabled="!ready" :on-press="next" />
    </view>
  </scroll-view>
</template>

<script>
import React from 'react';
import RadioItem from '../components/radioItem';
import { gql } from 'apollo-boost';

export default {
  props: {
    navigation: { type: Object },
    screenProps: { type: Object },
  },
  components: {
    RadioItem,
  },
  data: function() {
    const state = this.screenProps.store.state;
    return {
      name: state.name,
      childName: state.childName,
      email: state.email,
      userType: state.userType,
      ready: null,
    };
  },
  async mounted() {
    setTimeout(() => { this.readyNext(); }, 500);
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
      if (!this.name || !this.email || !this.userType) return;
      if (this.name.split(' ').length < 1) return;
      if (this.userType === 'child' && !this.childName) return;
      this.ready = true;
    },

    userTypeIs(type) {
      return type === this.userType;
    },

    selectUser(type) {
      this.userType = type;
      this.readyNext();
    },

    next() {
      const store = this.screenProps.store;
      store.commit('setName', this.name);
      store.commit('setEmail', this.email);
      store.commit('setUserType', this.userType);
      store.commit('setChildName', this.childName);
      this.navigation.navigate('Legal');
    },
  },
};
</script>

<style>
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

.mb-1 { margin-bottom: 5; }
.mb-2 { margin-bottom: 10; }
.mb-3 { margin-bottom: 15; }
.mb-4 { margin-bottom: 20; }

.mt-1 { margin-top: 5; }
.mt-2 { margin-top: 10; }
.mt-3 { margin-top: 15; }
.mt-4 { margin-top: 20; }

.text-box {
    height: 40;
    border-color: gray;
    border-width: 1;
    padding: 10;
    border-radius: 3;
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
