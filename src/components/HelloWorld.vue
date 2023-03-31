<template>
  <div class="hello">
    <ul v-if="addresses">
      <li v-for="(item, key, index) in addresses" :key="`addresses-${index}`">
        <span class="key"> {{ key }}: </span>
        <span>
          {{ item }}
        </span>
      </li>
    </ul>
  </div>
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
      addresses: undefined,
    };
  },
  created() {
    this.testFun();
  },
  methods: {
    async testFun() {
      const addresses = await addressesByNetwork[SupportedChainId.MAINNET];
      if (addresses) {
        this.addresses = addresses;
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
li {
  line-height: 20px;
  list-style: none;
  font-size: 12px;
  width: 60%;
  text-align: left;
}
.key {
  display: inline-block;
  width: 30%;
  margin-right: 10px;
}
</style>
