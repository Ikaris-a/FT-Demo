<template>
  <div class="hello">hello {{ test }}</div>
</template>

<script>
import { addressesByNetwork, SupportedChainId } from '@looksrare/sdk';
export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  data() {
    return {
      test: 1,
    };
  },
  created() {
    this.testFun();
  },
  methods: {
    async testFun() {
      const addresses = await addressesByNetwork[SupportedChainId.MAINNET];
      console.log(addresses);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
