<template>
  <view v-if="loading" class="loading"><text>Loading...</text></view>
  <scroll-view v-else ref="topview" :content-container-style="contentContainerStyle">
    <image class="terms" :source="currentTermsSource" />
    <awesome-button type="primary" :on-press="agree">I Agree</awesome-button>
  </scroll-view>
  
</template>

<script>
import AwesomeButton from 'react-native-really-awesome-button/src/themes/blue';
import { gql } from 'apollo-boost';
const moment = require('moment');

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
      agreedTerms: {},
      ready: null,
      loading: true,
      current: 0,
      terms: [
        'Liability_Waiver_and_Media_Release_2018_11_10',
      ],
    };
  },
  async mounted() {
    setTimeout(() => { this.loadTerms(); }, 500); // XXX UGH
  },
  computed: {
    contentContainerStyle() {
      return { alignItems: 'center', justifyContent: 'space-around', paddingTop: 20, overflow: 'scroll' };
    },
    currentTerms() {
      return this.terms[this.current];
    },
    currentTermsUrl() {
      return `https://tinkerkitchen.org/files/${this.currentTerms}.png`
    },
    currentTermsSource() {
      return { uri: this.currentTermsUrl };
    },
  },
  methods: {
    async loadTerms() {
      try {
        const state = this.screenProps.store.state;
        const client = this.screenProps.client;
        const ret = await client.query({
          query: gql`
            query getLegalTerms($name: String!, $email: String!) {
              get_legal_terms(name: $name, email: $email)
            }
          `,
          variables: { name: state.name, email: state.email },
        });
        if (ret.data.get_legal_terms) this.terms = ret.data.get_legal_terms;
      } catch (e) {}

      if (!this.terms.length) {
        await this.checkin();
        this.navigation.navigate('Done');
      } else {
        this.loading = false;
      }
    },

    async checkin() {
      const state = this.screenProps.store.state;
      const client = this.screenProps.client;
      const ret = await client.mutate({
        mutation: gql`
          mutation Checkin($data: CheckinInput!) {
            checkin(data: $data)
          }
        `,
        variables: {
          data: {
            name: state.name,
            child_name: state.childName,
            email: state.email,
            user_type: state.userType,
            agreed_terms: Object.entries(this.agreedTerms)
                                .map(e => ({ terms_name: e[0], agreed_timestamp: e[1] })),
          },
        },
      });
    },
    async agree() {
      const currentTerms = this.terms[this.current];
      this.agreedTerms[currentTerms] = moment().utc().format("YYYY-MM-DD HH:mm:ss");
      if (this.terms[this.current+1]) {
        this.current = this.current + 1;
        this.refs.topview.scrollTo(0, 0);
      } else {
        await this.checkin();
        this.navigation.navigate('Done');
      }
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
.terms {
    align-content: stretch;
    width: 840;
    height: 1000;
    border-width: 1;
    border-color: black;
    margin-bottom: 5;
}
</style>
