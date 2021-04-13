<template>
    <div>
<v-card style="
        background-color:darkblue;
        border: 0;
        margin: 0 auto;
        display: block;
        border-radius: 10px;
        min-width: 400px;
        max-width: 800px;
      ">
            <v-col>
                    <v-select
                                    :items='bridgeNetworks'
                                    v-model="fromNetwork"
                                    rounded
                                    style='max-width: 150px'
                                    label='Source Network'>
                    </v-select>
                <v-col style=";
        border: 0;
        margin: 0 auto;
        display: block;
        border-radius: 10px;">
                    <v-row>
                        <v-select
                                :items='tokens'
                                v-model="transferToken"
                                rounded
                                style='max-width: 150px'
                                label='Select Token'>
                        </v-select>
                    </v-row>
                    <v-row>
                        <v-input>
                            <v-text-field
                                    v-model="inputValue"
                                    label="Transfer Quantity"
                                    @keypress="isNumber($event)"
                                    style=";border: 0; margin: 0 auto;"
                            ></v-text-field>
                        </v-input>
                    </v-row>
                    <v-row>
                        <v-select
                                :items='bridgeNetworks'
                                v-model="depositNetwork"
                                rounded
                                style='max-width: 150px'
                                label='Destination Network'>
                        </v-select>
                    </v-row>
                </v-col>
            </v-col>
            <v-col>
                <v-btn @click="switchNetwork">Switch</v-btn>
            </v-col>
            <v-col>
                <v-btn @click="depositETHToMatic">Transfer</v-btn>
            </v-col>
        </v-card>
    </div>
</template>

<script>

//import inchTokens from '../../inchTokens.json'
import Matic from '@maticnetwork/maticjs'
const MaticPOSClient = require('@maticnetwork/maticjs').MaticPOSClient
import { ConnextSdk } from "@connext/vector-sdk"
//import { CrossChainSwap} from "@/types.ts";
//import Web3 from 'web3'
import axios from 'axios'
import { ethers } from 'ethers'

export default {
    name: 'BinanceBridge',
    data() {
        return {
            tokens: ['BNB', 'DAI', 'USDC', 'ETH', 'WBTC'],
            bridgeNetworks:['ETH','BSC','xDai','Matic'],
            networkIDs:{'ETH':1,'BSC':56,'xDai':100,'Fantom':125,'Huobi':128,'Matic':137,},
            ethProvider: 'https://still-nameless-star.quiknode.pro/',
            xDaiProvider:'https://fragrant-sparkling-surf.xdai.quiknode.pro/6a5ded94b3d8f2505e124d87cbb19d2dec3fb59d/',
            maticProvider: 'https://snowy-polished-voice.matic.quiknode.pro/c25d600e60591c4c592eceee9359b61ec55b1592/',
            bscProvider: 'https://patient-small-sound.bsc.quiknode.pro/e9d9c6fc165de5266e008da316e7b5220b1e1775/',
            matic: null,
            connextSdk: null,
            depositNetwork: 'BSC',
            fromNetwork: 'ETH',
            sendQuantity: 0,
            chain: 0,
            publicIdentifier: 'vector892GMZ3CuUkpyW8eeXfW2bt5W73TWEXtgV71nphXUXAmpncnj8',
            account: `0xF705b9ba1908cA505537F309B08E6949C1b8f31F`,
            inputValue: 0,
            transferToken: 'USDC',
            baseUrl: 'api.binance.org/bridge',
            binanceTokenAddresses:
                {
                    'BNB': '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c',
                    'DAI': '0x1af3f329e8be154074d8769d1ffa4ee058b1dbc3',
                    'USDC': '0x8ac76a51cc950d9822d68b83fe1ad97b32cd580d',
                    'ETH': '0x2170ed0880ac9a755fd29b2688956bd959f933f8',
                    'WBTC': '0x7130d2a12b9bcbfae4f2634d864a1ee1ce3ead9c'
                },
            ethTokenAddresses:
                {
                    'DAI': '0x6b175474e89094c44da98b954eedeac495271d0f',
                    'USDC': '0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
                    'ETH': '0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee',
                    'WETH': '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2',
                    'WBTC': '0x2260fac5e5542a773aa44fbcfedf7c193bc2c599'
                },
            crossChainAddresses:
                {
                    ETH:{
                        USDC:'0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
                        DAI:'0x6b175474e89094c44da98b954eedeac495271d0f',
                        ETH:'0x0000000000000000000000000000000000000000'
                    },
                    BSC:{
                        USDT:'0x55d398326f99059fF775485246999027B3197955',
                        USDC:'0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d',
                        DAI:'0x1AF3F329e8BE154074D8769D1FFa4eE058B1DBc3',
                        UNI:'0xBf5140A22578168FD562DCcF235E5D43A02ce9B1',
                    },
                    MATIC:{
                        ETH:'0x7ceB23fD6bC0adD59E62ac25578270cFf1b9f619',
                        USDT:'0xc2132D05D31c914a87C6611C10748AEb04B58e8F',
                        UNI:'0xb33EaAd8d922B1083446DC23f610c2567fB5180f',
                    }

                }

        }
    },
    methods:
        {

            async approve() {
                console.log('test approve')
            },

            isNumber: function (evt) {
                evt = (evt) ? evt : window.event;
                var charCode = (evt.which) ? evt.which : evt.keyCode;
                if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
                    evt.preventDefault()
                } else {
                    return true;
                }
            },
            

            async getChainId() {
                return window.ethereum.chainId
            },


        },
    async mounted() {

        let ethereum = window.ethereum;
        await ethereum.enable();
        console.log('ethereum',ethereum)
        const currentProvider = new ethers.providers.Web3Provider(ethereum)
        const connextSdk = new ConnextSdk();
        if (currentProvider) {

            try {
                // await secondHop.init({
                //     routerPublicIdentifier: 'vector892GMZ3CuUkpyW8eeXfW2bt5W73TWEXtgV71nphXUXAmpncnj8',
                //     loginProvider: secondProvider, // Web3/JsonRPCProvider
                //     senderChainProvider: 'https://snowy-polished-voice.matic.quiknode.pro/c25d600e60591c4c592eceee9359b61ec55b1592/', // Rpc Provider Link
                //     senderAssetId: "0x2791bca1f2de4661ed88a30c99a7a9449aa84174", // Asset/Token Address on Sender Chain
                //     recipientChainProvider: 'https://patient-small-sound.bsc.quiknode.pro/e9d9c6fc165de5266e008da316e7b5220b1e1775/', // Rpc Provider Link
                //     recipientAssetId: "0x8ac76a51cc950d9822d68b83fe1ad97b32cd580d", // Asset/Token Address on Recipient Chain
                // });
                await connextSdk.init({
                    routerPublicIdentifier: 'vector632zuTHFqVa8NXQcYAio8EqVyZXMwGmqYPFv1949EB8Ng3zsTR',
                    loginProvider: currentProvider, // Web3/JsonRPCProvider
                    senderChainProvider: 'https://still-nameless-star.quiknode.pro/', // Rpc Provider Link
                    senderAssetId: "0x0000000000000000000000000000000000000000", // Asset/Token Address on Sender Chain
                    recipientChainProvider: 'https://patient-small-sound.bsc.quiknode.pro/e9d9c6fc165de5266e008da316e7b5220b1e1775/', // Rpc Provider Link
                    recipientAssetId: "0x2170ed0880ac9a755fd29b2688956bd959f933f8", // Asset/Token Address on Recipient Chain
                });
                const estimateFeeParams = {input: '0.0050'}
                const quote = await connextSdk.estimateFees(estimateFeeParams)
                if (quote) {
                    try {
                        // const depositParams = {
                        //     transferAmount: '0.0050',
                        //     webProvider: currentProvider,
                        //     onDeposited: (txHash) => console.log(txHash)
                        // }
                        // await connextSdk.deposit(depositParams)
                        // console.log('deposit complete')
                        try {
                        //console.log('second hop object',secondHop.senderChainChannelAddress)
                            const data = {
                            recipientAddress: '0x54BABF466A61447f3ea64d0EE80207C9ae458a02',
                            onFinished: (params) => console.log(params)
                        }
                        await connextSdk.crossChainSwap(data);
                    } catch (e) {
                        console.log('error cross chain swap')
                    }
                    } catch (e) {
                        console.log('error deposit')
                    }
                }

            } catch (e) {
                const message = 'Error initalizing';
                console.log(e, message);
                throw e;
            }
        }

    },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>