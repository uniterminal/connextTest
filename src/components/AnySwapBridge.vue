<template>
    <div>

        <v-card style="
        background-color:darkblue;
        border: 0;
        margin: 0 auto;
        display: block;
        border-radius: 10px;
        max-width: 400px;
      ">
            <v-col>
                <v-subheader>From: {{this.fromNetwork}}</v-subheader>
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
                        <v-input>
                            <v-text-field
                                    v-model="inputValue"
                                    label="Transfer Value"
                                    @keypress="isNumber($event)"
                            ></v-text-field>
                        </v-input>
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
//import Web3 from 'web3'
import axios from 'axios'
import { ethers } from 'ethers'

export default {
  name: 'AnySwapBridge',
  data()
  {
    return {
        tokens:['QUICK','DAI','USDC','aUSDC','ETH','MATIC','WBTC','WETH'],
        ethProvider: 'wss://still-nameless-star.quiknode.pro/',
        maticProvider:'https://snowy-polished-voice.matic.quiknode.pro/c25d600e60591c4c592eceee9359b61ec55b1592/',
        maticBridgeABI:[{"inputs":[{"internalType":"address","name":"_proxyTo","type":"address"}],"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_new","type":"address"},{"indexed":false,"internalType":"address","name":"_old","type":"address"}],"name":"ProxyOwnerUpdate","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"_new","type":"address"},{"indexed":true,"internalType":"address","name":"_old","type":"address"}],"name":"ProxyUpdated","type":"event"},{"stateMutability":"payable","type":"fallback"},{"inputs":[],"name":"implementation","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"proxyOwner","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"proxyType","outputs":[{"internalType":"uint256","name":"proxyTypeId","type":"uint256"}],"stateMutability":"pure","type":"function"},{"inputs":[{"internalType":"address","name":"newOwner","type":"address"}],"name":"transferProxyOwnership","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"address","name":"_newProxyTo","type":"address"},{"internalType":"bytes","name":"data","type":"bytes"}],"name":"updateAndCall","outputs":[],"stateMutability":"payable","type":"function"},{"inputs":[{"internalType":"address","name":"_newProxyTo","type":"address"}],"name":"updateImplementation","outputs":[],"stateMutability":"nonpayable","type":"function"},{"stateMutability":"payable","type":"receive"}],
        matic: null,
        depositNetwork:'Matic',
        fromNetwork:'Ethereum',
        sendQuantity: 0,
        chain: 0,
        account:`0xF705b9ba1908cA505537F309B08E6949C1b8f31F`,
        inputValue:0,
        transferToken: 'QUICK',
        maticTokenAddresses:
        {
            'QUICK':'0x831753DD7087CaC61aB5644b308642cc1c33Dc13',
            'DAI':'0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063',
            'USDC':'0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
            'aUSDC':'0x9719d867A500Ef117cC201206B8ab51e794d3F82',
            'ETH':'0x7ceB23fD6bC0adD59E62ac25578270cFf1b9f619',
            'MATIC':'0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270',
            'WBTC':'0x1BFD67037B42Cf73acF2047067bd4F2C47D9BfD6'
        },
        ethTokenAddresses:
        {
            'QUICK':'0x6c28AeF8977c9B773996d0e8376d2EE379446F2f',
            'DAI':'0x6b175474e89094c44da98b954eedeac495271d0f',
            'USDC':'0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48',
            'aUSDC':'0x9bA00D6856a4eDF4665BcA2C2309936572473B7E',
            'ETH': '0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee',
            'WETH':'0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2',
            'MATIC':'0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0',
            'WBTC':'0x2260fac5e5542a773aa44fbcfedf7c193bc2c599'
        },

    }
  },
  methods:
    {

        async approve()
        {
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

        async send()
        {
            console.log('testing')
            console.log(this.maticProvider)
            console.log('matic',this.matic)
            console.log('eth',this.ethProvider)
            console.log(this.depositNetwork)
            this.approveERC20ETH()

        },

        async switchNetwork()
        {
            console.log(this.tokenList)
            if(this.fromNetwork == 'Ethereum')
            {
                await this.switchToMatic()
                this.fromNetwork = 'Matic'

            }
            else
            {
                console.log()
                this.fromNetwork = 'Ethereum'
            }
        },

        async bridgeAPI()
        {
            console.log('calling bridge')
            const bridgeData = await axios.get('https://bridgeapi.anyswap.exchange/v2/serverInfo/chainid')
            if(bridgeData)
            {
                console.log('response from bridge',bridgeData)
            }
        },


        async approveERC20ETH()
        {
            //const depositProxy = await this.matic.getERC20TokenContract('0x401F6c983eA34274ec46f84D70b31C151321188b',true)
            const matic = new Matic({
            network: 'mainnet', // set network name
  version: 'v1',// set network version

  // Set Matic provider - string or provider instance
  // Example: <network.Matic.RPC> OR new Web3.providers.HttpProvide(<network.Matic.RPC>)
  // Some flows like startExitFor[Metadata]MintableBurntToken, require a webSocket provider such as new web3 providers.WebsocketProvider('ws://localhost:8546')
  maticProvider:this.maticProvider,

  // Set Mainchain provider - string or provider instance
  // Example: 'https://ropsten.infura.io' OR new Web3.providers.HttpProvider('http://localhost:8545')
  parentProvider:this.ethProvider,

})
            const tokenContract = await matic.getERC20TokenContract('0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0',true)
            if(tokenContract)
            {
                console.log('token contract',tokenContract)
            }

            const maticBalance = await tokenContract.methods.balanceOf('0xF705b9ba1908cA505537F309B08E6949C1b8f31F').call()
            if(maticBalance)
            {
                console.log(maticBalance)
            }

            const approval = await tokenContract.methods.allowance('0xF705b9ba1908cA505537F309B08E6949C1b8f31F','0x401F6c983eA34274ec46f84D70b31C151321188b').call()
            if(approval)
            {
                console.log(approval)
            }

            const depositAmount = 10 ** 17
            console.log(depositAmount)
            //console.log(this.web3.eth.net.getId())
            // const maticBalance = await this.matic.balanceOfERC20('0xF705b9ba1908cA505537F309B08E6949C1b8f31F','0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0')
            // if(maticBalance)
            // {
            //     console.log('matic balance',maticBalance)
            // }
            //
            // const ethBalance = await this.matic.balanceOfERC20('0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270','0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0')
            // if(ethBalance)
            // {
            //     console.log('matic balance',ethBalance)
            // }

        },

        async depositERC20ToMatic()
        {
            console.log('deposit ERC20 to Matic')
        },

        async depositETHToMatic()
        {
            const maticPoSClient = this.posClientParent()
            const x = this.inputValue * 1000000000000000000; // 18 decimals
            const x1 = x.toString();
            console.log('deposit ETH to Matic')
            await maticPoSClient.depositEtherForUser(this.account, x1, {
            from: this.account,
            });
        },

        async depositERC20ToEthereum()
        {

            console.log('deposit ERC20 to Ethereum')
        },

        posClientParent() {
            const maticPostClient = new MaticPOSClient({
          network:'mainnet',
          version: 'v1',
          maticProvider: this.maticProvider,
          parentProvider:window.ethereum
          });
            return maticPostClient;
        },

        posClientChild() {
            const maticPostClient = new MaticPOSClient({
          network:'mainnet',
          version: 'v1',
          maticProvider: window.ethereum,
          parentProvider:this.ethProvider
          });
            return maticPostClient;
        },

        async depositETHToEthereum()
        {
            console.log('deposit ETH to Ethereum')
        },

        setTransferMatic()
        {
            this.depositNetwork = 'Matic'
            console.log(this.chain)
        },

        setTransferEthereum()
        {
            this.depositNetwork = 'Ethereum'
            console.log(this.chain)
        },

        async getChainId()
        {
            return window.ethereum.chainId
        },

        switchToMatic: async function () {

            let ethereum = window.ethereum;
            const data = [{
                chainId: '0x89',
                chainName: 'Matic',
                nativeCurrency:
                    {
                        name: 'Matic',
                        symbol: 'Matic',
                        decimals: 18
                    },
                rpcUrls: ['https://rpc-mainnet.maticvigil.com'],
                blockExplorerUrls: ['https://explorer-mainnet.maticvigil.com/'],
            }]
            /* eslint-disable */
            const tx = await ethereum.request({method: 'wallet_addEthereumChain', params:data}).catch()
            if (tx) {
                console.log(tx)
            }
        },

        switchToBinance: async function () {

            let ethereum = window.ethereum;
            const data = [{
                chainId: '0x38',
                chainName: 'Binance Smart Chain',
                nativeCurrency:
                    {
                        name: 'BNB',
                        symbol: 'BNB',
                        decimals: 18
                    },
                rpcUrls: ['https://bsc-dataseed.binance.org/'],
                blockExplorerUrls: ['https://bscscan.com/'],
            }]
            /* eslint-disable */
            const tx = await ethereum.request({method: 'wallet_addEthereumChain', params:data}).catch()
            if (tx) {
                console.log(tx)
            }
        },

        switchToOptimism: async function () {

            let ethereum = window.ethereum;
            const data = [{
          chainId: '0xA',
          chainName: 'Optimism',
          nativeCurrency: {
            name: 'Ethereum',
            symbol: 'ETH',
            decimals: 18,
          },
          rpcUrls: ['https://green-lingering-feather.optimism.quiknode.pro/9b5ab369a9cf5af7d42ecad8b57952ed58f70b1d/'],
          blockExplorerUrls: ['https://etherscan.io'],
        }]
            /* eslint-disable */
            const tx = await ethereum.request({method: 'wallet_addEthereumChain', params:data}).catch()
            if (tx) {
                console.log(tx)
            }
        },

        async pancakeSwap(){
            console.log('pancake')

        },

        // NOT WORKING
        switchToEthereum: async function () {

            let ethereum = window.ethereum;
            const data = [{
          chainId: '0x1',
          chainName: 'Ethereum',
          nativeCurrency: {
            name: 'Ethereum',
            symbol: 'ETH',
            decimals: 18,
          },
          rpcUrls: ['https://mainnet.infura.io/v3/undefined'],
          blockExplorerUrls: ['https://etherscan.io'],
        }]
            /* eslint-disable */
            const tx = await ethereum.request({method: 'wallet_addEthereumChain', params:data}).catch()
            if (tx) {
                console.log(tx)
            }
        },

        },

    async mounted()
    {
        console.log('starting')
        this.bridgeAPI()
        console.log('bridge called')
        this.getChainId()
        console.log('testing')
        console.log(this.maticProvider)
        this.currentProvider = new ethers.providers.Web3Provider(window.ethereum)
        // *eslint-disable*
        var chainId = await window.ethereum.request({method:'eth_chainId'})
        if(chainId)
        {
            console.log('chainID',chainId)
        }
        console.log(this.currentProvider)
        console.log(this.chain)
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