<template>
  <view class="wrapper">
    <view class="background">
      <touchable-opacity class="item" :on-press="press">
        <text v-if="selected" class="left selected">{{ text }}</text>
        <text v-else class="left">{{ text }}</text>
        <text v-if="selected" class="right">x</text>
      </touchable-opacity>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    name: {
      Type: String,
    },
    text: {
      Type: String,
    },
    selected: {
      Type: Boolean,
    },
    onselect: {
      Type: Function,
    },
  },
  components: {
  },
  data: function() {
    return {
    };
  },
  methods: {
    press() {
      this.onselect(this.name);
    }
  },
};
</script>

<style>
.wrapper {
    height: 50;
    padding-left: 16;
}
.background {
    background-color: gray;
    height: 100%;
}
.item {
    align-items: center;
    background-color: #fff;
    border-bottom-color: lightgray;
    border-bottom-width: 1;
    flex: 1;
    flex-direction: row;
    justify-content: space-between;
}
.left {
    font-size: 16;
}
.selected {
    color: blue;
}
.right {
}
</style>
