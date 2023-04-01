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
    this.axiosPostFun();
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
        params: {
          address: '0x58c3c2547084CC1C94130D6fd750A3877c7Ca5D2', // 传接口参数
        },
      })
        .then((response) => {
          console.log(response, 'success'); // 成功的返回
        })
        .catch((error) => console.log(error, 'error')); // 失败的返回
    },
    axiosPostFun() {
      const add = '0x58c3c2547084CC1C94130D6fd750A3877c7Ca5D2';
      const data = {
        signature: '0x465E2C346Ef0230f8a904f7593a0395f063B579f',
        collection: add,
        tokenId: '', // ?
        signer: add,
        strategy: add,
        isOrderAsk: true,
        currency: add,
        nonce: '56',
        amount: '1',
        price: '40000',
        startTime: '1645553279',
        endTime: '1645554279',
        minPercentageToAsk: '', // ?
        params: '8500',
      };
      this.$axios({
        method: 'post',
        url: 'https://api.looksrare.org/api/v1/orders', // 接口地址
        data,
        headers: {
          'X-Looks-Api-Key':
            'b585acd2c2447ccfcc80c4c18cba4a4f2daa0791cc181ff5ae8e5ec7c1ccc594',
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
