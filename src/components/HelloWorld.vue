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
    this.axiosFun();
  },
  methods: {
    async testFun() {
      const addresses = await addressesByNetwork[SupportedChainId.MAINNET];
      if (addresses) {
        this.addresses = addresses;
      }
    },
    axiosFun() {
      this.$axios({
        method: 'get',
        url: 'https://api.looksrare.org/api/v1/accounts', // 接口地址
        data: {
          address: '1', // 传接口参数
        },
      })
        .then((response) => {
          console.log(response, 'success'); // 成功的返回
        })
        .catch((error) => console.log(error, 'error')); // 失败的返回
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
