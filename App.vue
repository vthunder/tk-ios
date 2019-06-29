<template>
  <app-navigation :screen-props="screenProps"></app-navigation>
</template>

<script>
import Vue from 'vue-native-core';
import Vuex from 'vuex';
import { StackNavigator } from 'vue-native-router';
import ApolloClient from 'apollo-boost';

import HomeScreen from './screens/home';
import FormScreen from './screens/form';
import LegalScreen from './screens/legal';
import DoneScreen from './screens/done';

Vue.use(Vuex);

const store = new Vuex.Store({
  state: {
    name: null,
    email: null,
    userType: null,
    childName: null,
    printerUri: null,
  },
  mutations: {
    setName(state, value) {
      state.name = value;
    },
    setEmail(state, value) {
      state.email = value;
    },
    setUserType(state, value) {
      state.userType = value;
    },
    setChildName(state, value) {
      state.childName = value;
    },
    setPrinterUri(state, value) {
      state.printerUri = value;
    },
    reset(state) {
      state.name = null;
      state.email = null;
      state.userType = null;
      state.childName = null;
      // note: we leave printerUri as is
    },
  }
});

const client = new ApolloClient({ uri: 'https://tinkerkitchen.org/api' });

const AppNavigation = StackNavigator(
  {
    Home: HomeScreen,
    Form: FormScreen,
    Legal: LegalScreen,
    Done: DoneScreen,
  },
  { initialRouteName: 'Home' }
);

export default {
  components: {
    AppNavigation,
  },
  data() {
    return {
      screenProps: { client, store },
    }
  },
};
</script>
