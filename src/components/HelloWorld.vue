<template>
  <div class="hello">
    <!-- <ul v-if="addresses">
      <li v-for="(item, key, index) in addresses" :key="`addresses-${index}`">
        <span class="key"> {{ key }}: </span>
        <span>
          {{ item }}
        </span>
      </li>
    </ul> -->
  </div>
</template>

<script>
import {
  addressesByNetwork,
  SupportedChainId,
  generateMakerOrderTypedData,
  encodeOrderParams,
  LooksRareExchangeAbi,
  // signMakerOrder,
} from "@looksrare/sdk";
import { ethers } from "ethers";
export default {
  name: "HelloWorld",
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
      const chainId = SupportedChainId.GOERLI;
      const addresses = addressesByNetwork[chainId];
      const privateKey =
        "";//钱包密钥
      const provider = new ethers.providers.JsonRpcProvider(
        `https://goerli.blockpi.network/v1/rpc/public`
      );
      const signer = new ethers.Wallet(privateKey).connect(provider);
      const signerAddress = await signer.getAddress();
      let wallet = new ethers.Wallet(privateKey, provider);
      let nonce = await wallet.getTransactionCount();
      const now = Math.floor(Date.now() / 1000);
      const paramsValue = [];
      // const netPriceRatio = BigNumber.from(10000).sub(protocolFees.add(creatorFees)).toNumber();
      const minNetPriceRatio = 9500; //税费
      const makerOrder = {
        isOrderAsk: true,
        signer: signerAddress,
        collection: "0x77566D540d1E207dFf8DA205ed78750F9a1e7c55",
        price: "200000000000000",
        tokenId: "462",
        amount: "1",
        strategy: addresses.STRATEGY_STANDARD_SALE,
        currency: addresses.WETH,
        nonce: nonce, // order nonce (must be unique unless new maker order is meant to override existing one e.g., lower ask price)
        startTime: now, // startTime in timestamp
        endTime: now + 86400, // endTime in timestamp
        minPercentageToAsk: minNetPriceRatio,
        params: paramsValue,
      };
      // const signatureHash = await signMakerOrder(signer, chainId, makerOrder);
      const { domain, value, type } = generateMakerOrderTypedData(
        signerAddress,
        chainId,
        makerOrder
      );
      console.log("=====makerOrder", makerOrder);
      const signature = await signer._signTypedData(domain, type, value);
      //Maker
      const { encodedParams } = encodeOrderParams(makerOrder.params);
      const takerOrder = {
        isOrderAsk: false,
        taker: signerAddress,
        price: makerOrder.price,
        tokenId: makerOrder.tokenId,
        minPercentageToAsk: 7500,
        params: encodedParams,
      };
      makerOrder.collection = makerOrder.collectionAddress;
      makerOrder.currency = makerOrder.currencyAddress;
      makerOrder.params = encodedParams;
      const exchangeAbi = [
        {
          inputs: [
            {
              internalType: "address",
              name: "_currencyManager",
              type: "address",
            },
            {
              internalType: "address",
              name: "_executionManager",
              type: "address",
            },
            {
              internalType: "address",
              name: "_royaltyFeeManager",
              type: "address",
            },
            {
              internalType: "address",
              name: "_WETH",
              type: "address",
            },
            {
              internalType: "address",
              name: "_protocolFeeRecipient",
              type: "address",
            },
          ],
          stateMutability: "nonpayable",
          type: "constructor",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "user",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "newMinNonce",
              type: "uint256",
            },
          ],
          name: "CancelAllOrders",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "user",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256[]",
              name: "orderNonces",
              type: "uint256[]",
            },
          ],
          name: "CancelMultipleOrders",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "currencyManager",
              type: "address",
            },
          ],
          name: "NewCurrencyManager",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "executionManager",
              type: "address",
            },
          ],
          name: "NewExecutionManager",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "protocolFeeRecipient",
              type: "address",
            },
          ],
          name: "NewProtocolFeeRecipient",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "royaltyFeeManager",
              type: "address",
            },
          ],
          name: "NewRoyaltyFeeManager",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "transferSelectorNFT",
              type: "address",
            },
          ],
          name: "NewTransferSelectorNFT",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "previousOwner",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "newOwner",
              type: "address",
            },
          ],
          name: "OwnershipTransferred",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: true,
              internalType: "address",
              name: "collection",
              type: "address",
            },
            {
              indexed: true,
              internalType: "uint256",
              name: "tokenId",
              type: "uint256",
            },
            {
              indexed: true,
              internalType: "address",
              name: "royaltyRecipient",
              type: "address",
            },
            {
              indexed: false,
              internalType: "address",
              name: "currency",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "amount",
              type: "uint256",
            },
          ],
          name: "RoyaltyPayment",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: false,
              internalType: "bytes32",
              name: "orderHash",
              type: "bytes32",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "orderNonce",
              type: "uint256",
            },
            {
              indexed: true,
              internalType: "address",
              name: "taker",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "maker",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "strategy",
              type: "address",
            },
            {
              indexed: false,
              internalType: "address",
              name: "currency",
              type: "address",
            },
            {
              indexed: false,
              internalType: "address",
              name: "collection",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "tokenId",
              type: "uint256",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "amount",
              type: "uint256",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "price",
              type: "uint256",
            },
          ],
          name: "TakerAsk",
          type: "event",
        },
        {
          anonymous: false,
          inputs: [
            {
              indexed: false,
              internalType: "bytes32",
              name: "orderHash",
              type: "bytes32",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "orderNonce",
              type: "uint256",
            },
            {
              indexed: true,
              internalType: "address",
              name: "taker",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "maker",
              type: "address",
            },
            {
              indexed: true,
              internalType: "address",
              name: "strategy",
              type: "address",
            },
            {
              indexed: false,
              internalType: "address",
              name: "currency",
              type: "address",
            },
            {
              indexed: false,
              internalType: "address",
              name: "collection",
              type: "address",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "tokenId",
              type: "uint256",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "amount",
              type: "uint256",
            },
            {
              indexed: false,
              internalType: "uint256",
              name: "price",
              type: "uint256",
            },
          ],
          name: "TakerBid",
          type: "event",
        },
        {
          inputs: [],
          name: "DOMAIN_SEPARATOR",
          outputs: [
            {
              internalType: "bytes32",
              name: "",
              type: "bytes32",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "WETH",
          outputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "uint256",
              name: "minNonce",
              type: "uint256",
            },
          ],
          name: "cancelAllOrdersForSender",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "uint256[]",
              name: "orderNonces",
              type: "uint256[]",
            },
          ],
          name: "cancelMultipleMakerOrders",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [],
          name: "currencyManager",
          outputs: [
            {
              internalType: "contract ICurrencyManager",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "executionManager",
          outputs: [
            {
              internalType: "contract IExecutionManager",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "user",
              type: "address",
            },
            {
              internalType: "uint256",
              name: "orderNonce",
              type: "uint256",
            },
          ],
          name: "isUserOrderNonceExecutedOrCancelled",
          outputs: [
            {
              internalType: "bool",
              name: "",
              type: "bool",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              components: [
                {
                  internalType: "bool",
                  name: "isOrderAsk",
                  type: "bool",
                },
                {
                  internalType: "address",
                  name: "taker",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "price",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "tokenId",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "minPercentageToAsk",
                  type: "uint256",
                },
                {
                  internalType: "bytes",
                  name: "params",
                  type: "bytes",
                },
              ],
              internalType: "struct OrderTypes.TakerOrder",
              name: "takerBid",
              type: "tuple",
            },
            {
              components: [
                {
                  internalType: "bool",
                  name: "isOrderAsk",
                  type: "bool",
                },
                {
                  internalType: "address",
                  name: "signer",
                  type: "address",
                },
                {
                  internalType: "address",
                  name: "collection",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "price",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "tokenId",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "amount",
                  type: "uint256",
                },
                {
                  internalType: "address",
                  name: "strategy",
                  type: "address",
                },
                {
                  internalType: "address",
                  name: "currency",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "nonce",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "startTime",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "endTime",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "minPercentageToAsk",
                  type: "uint256",
                },
                {
                  internalType: "bytes",
                  name: "params",
                  type: "bytes",
                },
                {
                  internalType: "uint8",
                  name: "v",
                  type: "uint8",
                },
                {
                  internalType: "bytes32",
                  name: "r",
                  type: "bytes32",
                },
                {
                  internalType: "bytes32",
                  name: "s",
                  type: "bytes32",
                },
              ],
              internalType: "struct OrderTypes.MakerOrder",
              name: "makerAsk",
              type: "tuple",
            },
          ],
          name: "matchAskWithTakerBid",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              components: [
                {
                  internalType: "bool",
                  name: "isOrderAsk",
                  type: "bool",
                },
                {
                  internalType: "address",
                  name: "taker",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "price",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "tokenId",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "minPercentageToAsk",
                  type: "uint256",
                },
                {
                  internalType: "bytes",
                  name: "params",
                  type: "bytes",
                },
              ],
              internalType: "struct OrderTypes.TakerOrder",
              name: "takerBid",
              type: "tuple",
            },
            {
              components: [
                {
                  internalType: "bool",
                  name: "isOrderAsk",
                  type: "bool",
                },
                {
                  internalType: "address",
                  name: "signer",
                  type: "address",
                },
                {
                  internalType: "address",
                  name: "collection",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "price",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "tokenId",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "amount",
                  type: "uint256",
                },
                {
                  internalType: "address",
                  name: "strategy",
                  type: "address",
                },
                {
                  internalType: "address",
                  name: "currency",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "nonce",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "startTime",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "endTime",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "minPercentageToAsk",
                  type: "uint256",
                },
                {
                  internalType: "bytes",
                  name: "params",
                  type: "bytes",
                },
                {
                  internalType: "uint8",
                  name: "v",
                  type: "uint8",
                },
                {
                  internalType: "bytes32",
                  name: "r",
                  type: "bytes32",
                },
                {
                  internalType: "bytes32",
                  name: "s",
                  type: "bytes32",
                },
              ],
              internalType: "struct OrderTypes.MakerOrder",
              name: "makerAsk",
              type: "tuple",
            },
          ],
          name: "matchAskWithTakerBidUsingETHAndWETH",
          outputs: [],
          stateMutability: "payable",
          type: "function",
        },
        {
          inputs: [
            {
              components: [
                {
                  internalType: "bool",
                  name: "isOrderAsk",
                  type: "bool",
                },
                {
                  internalType: "address",
                  name: "taker",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "price",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "tokenId",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "minPercentageToAsk",
                  type: "uint256",
                },
                {
                  internalType: "bytes",
                  name: "params",
                  type: "bytes",
                },
              ],
              internalType: "struct OrderTypes.TakerOrder",
              name: "takerAsk",
              type: "tuple",
            },
            {
              components: [
                {
                  internalType: "bool",
                  name: "isOrderAsk",
                  type: "bool",
                },
                {
                  internalType: "address",
                  name: "signer",
                  type: "address",
                },
                {
                  internalType: "address",
                  name: "collection",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "price",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "tokenId",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "amount",
                  type: "uint256",
                },
                {
                  internalType: "address",
                  name: "strategy",
                  type: "address",
                },
                {
                  internalType: "address",
                  name: "currency",
                  type: "address",
                },
                {
                  internalType: "uint256",
                  name: "nonce",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "startTime",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "endTime",
                  type: "uint256",
                },
                {
                  internalType: "uint256",
                  name: "minPercentageToAsk",
                  type: "uint256",
                },
                {
                  internalType: "bytes",
                  name: "params",
                  type: "bytes",
                },
                {
                  internalType: "uint8",
                  name: "v",
                  type: "uint8",
                },
                {
                  internalType: "bytes32",
                  name: "r",
                  type: "bytes32",
                },
                {
                  internalType: "bytes32",
                  name: "s",
                  type: "bytes32",
                },
              ],
              internalType: "struct OrderTypes.MakerOrder",
              name: "makerBid",
              type: "tuple",
            },
          ],
          name: "matchBidWithTakerAsk",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [],
          name: "owner",
          outputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "protocolFeeRecipient",
          outputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "renounceOwnership",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [],
          name: "royaltyFeeManager",
          outputs: [
            {
              internalType: "contract IRoyaltyFeeManager",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "newOwner",
              type: "address",
            },
          ],
          name: "transferOwnership",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [],
          name: "transferSelectorNFT",
          outputs: [
            {
              internalType: "contract ITransferSelectorNFT",
              name: "",
              type: "address",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "_currencyManager",
              type: "address",
            },
          ],
          name: "updateCurrencyManager",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "_executionManager",
              type: "address",
            },
          ],
          name: "updateExecutionManager",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "_protocolFeeRecipient",
              type: "address",
            },
          ],
          name: "updateProtocolFeeRecipient",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "_royaltyFeeManager",
              type: "address",
            },
          ],
          name: "updateRoyaltyFeeManager",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "_transferSelectorNFT",
              type: "address",
            },
          ],
          name: "updateTransferSelectorNFT",
          outputs: [],
          stateMutability: "nonpayable",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          name: "userMinOrderNonce",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
      ];

      const exchangeInterface = new ethers.utils.Interface(exchangeAbi);
      const exchangeContract = new ethers.Contract(
        addresses.EXCHANGE,
        exchangeInterface,
        signer
      );
      // await exchangeContract.functions.matchAskWithTakerBid(
      //   takerOrder,
      //   makerOrder
      // );
      console.log(takerOrder, exchangeContract, "exchangeContract=====");
      console.log(
        signature,
        ethers.utils,
        LooksRareExchangeAbi,
        "====signature===="
      );
      // this.orders(signature, makerOrder);
    },
    orders(signature, makerOrder) {
      let newData = {
        ...makerOrder,
        signature,
      };
      console.log(newData, "newData===");
      this.$axios({
        method: "post",
        url: "https://api-goerli.looksrare.org/api/v1/orders", // orders
        data: newData,
      }).then((res) => {
        console.log(res, "====");
      });
    },
    getUserNonce(signer) {
      this.$axios({
        method: "get",
        url: "https://api-goerli.looksrare.org/api/v1/orders/nonce", // 接口地址
        data: {
          signer, // 传接口参数
        },
      })
        .then((response) => {
          console.log(response, "success"); // 成功的返回
        })
        .catch((error) => console.log(error, "error")); // 失败的返回
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
