<template>
  <view v-if="loading" class="loading"><text>Loading...</text></view>
  <scroll-view v-else :content-container-style="contentContainerStyle">
    <text class="h2">Thanks for checking in!</text>
    <text v-if="printerUri" class="h3 my-2">Printing badge...</text>
    <view v-if="subscribed === false" class="horizontal mt-4">
      <awesome-button class="mr-4" type="primary" :on-press="subscribe">
        Subscribe to Our Email List
      </awesome-button>
      <awesome-button v-if="printerUri" type="secondary" :on-press="reset">No Thanks</awesome-button>
      <awesome-button v-else type="secondary" :on-press="printAndDone">No Thanks - Print Badge</awesome-button>
    </view>
    <view v-else-if="subscribed" class="mt-4">
      <awesome-button v-if="printerUri" type="primary" :on-press="reset">Done</awesome-button>
      <awesome-button v-else type="primary" :on-press="printAndDone">Print Badge</awesome-button>
    </view>
  </scroll-view>
</template>

<script>
import AwesomeButton from 'react-native-really-awesome-button/src/themes/blue';
import { gql } from 'apollo-boost';
import moment from 'moment';
import * as Print from 'expo-print';
import { Platform } from 'react-native';

export default {
  props: {
    navigation: { type: Object },
    screenProps: { type: Object },
  },
  components: {
    AwesomeButton,
  },
  data: function() {
    return {
      loading: true,
      subscribed: null,
      subSuccess: null,
      subFailure: false,
      successMsg: '',
    };
  },
  async mounted() {
    if (this.printerUri) this.print();
    this.checkSubscription();
    this._loadingTimer = setTimeout(() => {
      if (this.loading) {
        this.subscribed = true; // don't show subscribe button if something is broken
        this.loading = false;
      }
    }, 5 * 1000); // timeout in 5 seconds
    this._resetTimer = setTimeout(() => { this.reset(); }, 30 * 1000); // reset in 30 seconds
  },
  computed: {
    contentContainerStyle() {
      return { alignItems: 'center', justifyContent: 'center', flex: 1 };
    },
    printerUri() {
      return this.screenProps.store.state.printerUri;
    },
    userName() {
      if (this.screenProps.store.state.name === 'Ashley Qian')
        return `Ashley Qian &#9734;&#9734;&#9734;`
      return this.screenProps.store.state.name;
    },
    userType() {
      const state = this.screenProps.store.state;
      if (state.userType === 'member') return 'Member';
      if (state.userType === 'founding-member') return 'Founding Member';
      if (state.userType === 'staff') return 'STAFF';
      if (state.userType === 'guest') return 'Guest';
      if (state.userType === 'daypass-ticket') return 'Day-pass / Ticket';
      if (state.userType === 'child') return '';
    },
  },
  methods: {
    async printAndDone() {
      await this.print();
      this.reset();
    },
    async print() {
      if (Platform.OS !== 'ios') return;
      const store = this.screenProps.store;
      if (!store.state.printerUri) {
        const printer = await Print.selectPrinterAsync();
        store.commit('setPrinterUri', printer.url);
      }
      await Print.printAsync({
        html: `
          <style>
            @page { margin: 0px; }
            * { font-family: sans-serif; margin: 0; }
            .content {
              height: 100%;
              display: flex;
              flex-direction: column;
            }
            .name { font-size: 92; }
            .type { font-size: 42; flex-grow: 1; }
            .date { font-size: 32; }
          </style>
          <div class="content">
            <h1 class="name">${this.userName}</h1>
            <h2 class="type">${this.userType}</h2>
            <p class="date">Checked in on: ${moment().format('MMMM Do YYYY')}</p>
          </div>`,
        orientation: Print.Orientation.portrait,
        width: 612,
        height: 410,
        printerUrl: store.state.printerUri,
      });
    },
    async checkSubscription() {
      const state = this.screenProps.store.state;
      const client = this.screenProps.client;

      if (!state.email) return;

      const ret = await client.query({
        query: gql`
          query mailingListCheck($email: String!, $list: String) {
            mailing_list_check(email: $email, list: $list)
          }
        `,
        variables: { email: state.email }
      });
      if (ret && ret.data) {
        this.subscribed = ret.data.mailing_list_check;
      } else {
        this.subscribed = true; // don't show subscribe button if something went wrong
      }
      if (this._loadingTimer) clearTimeout(this._timer);
      this.loading = false;
    },

    async subscribe() {
      this.subSuccess = null;
      this.subFailure = false;

      try {
        const state = this.screenProps.store.state;
        const client = this.screenProps.client;
        const ret = await client.mutate({
          mutation: gql`
            mutation MailingListSignup($name: String, $email: String!) {
              mailing_list_signup(name: $name, email: $email)
            }
          `,
          variables: {
            name: state.name,
            email: state.email,
            list: 'Tinker Kitchen Newsletter',
          },
        });
        this.reset();
      } catch(e) {
        // (({ graphQLErrors: [{ message }] }) => {
        // switch (message) {
        //   case 'Member Exists':
        //     this.successMsg = 'Success! But... it seems you were already subscribed.';
        //     this.subSuccess = true;
        //     break;
        //   default:
        //     this.failure = true;
        // }
        // fixme: should we show an error?
        this.reset();
      }
    },

    reset() {
      if (this._resetTimer) clearTimeout(this._resetTimer);
      this.screenProps.store.commit('reset');
      this.navigation.popToTop();
    },
  },
};
</script>

<style>
.loading, .container {
    align-items: center;
    justify-content: center;
    flex: 1;
}
.horizontal {
    flex-direction: row;
    align-items: center;
    justify-content: center;
}
.h2 { font-size: 24; }
.h3 { font-size: 18; }
.my-2 { margin-top: 10; margin-bottom: 10; }
.mt-4 { margin-top: 20; }
.mr-4 { margin-right: 20; }
</style>
