<template>
  <view v-if="loading" class="loading"><text>Loading...</text></view>
  <scroll-view v-else :content-container-style="contentContainerStyle">
    <text class="h2">Thanks for checking in!</text>
    <view v-if="subscribed === false" class="horizontal mt-4">
      <awesome-button class="mr-4" type="primary" :on-press="subscribe">
        Subscribe to Our Email List
      </awesome-button>
      <awesome-button type="secondary" :on-press="reset">No Thanks</awesome-button>
    </view>
    <view v-else-if="subscribed" class="mt-4">
      <awesome-button type="primary" :on-press="reset">Done</awesome-button>
    </view>
  </scroll-view>
</template>

<script>
import AwesomeButton from 'react-native-really-awesome-button/src/themes/blue';
import { gql } from 'apollo-boost';

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
    this.checkSubscription();
    this._timer = setTimeout(() => {
      if (loading) {
        this.subscribed = true; // don't show subscribe button if something is broken
        this.loading = false;
      }
    }, 5 * 1000); // timeout in 5 seconds
    setTimeout(() => { this.reset(); }, 30 * 1000); // reset in 30 seconds
  },
  computed: {
    contentContainerStyle() {
      return { alignItems: 'center', justifyContent: 'center', flex: 1 };
    },
  },
  methods: {
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
      if (this._timer) clearTimeout(this._timer);
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
.mt-4 { margin-top: 20; }
.mr-4 { margin-right: 20; }
</style>
