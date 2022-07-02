<template>
  <v-app dark>
    <v-navigation-drawer v-model="drawer" :mini-variant="miniVariant" :clipped="clipped" fixed app>
      <v-list>
        <v-list-item v-for="(item, i) in items" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-btn icon @click.stop="miniVariant = !miniVariant">
        <v-icon>mdi-{{ `chevron-${miniVariant ? 'right' : 'left'}` }}</v-icon>
      </v-btn>
      <v-btn icon @click.stop="clipped = !clipped">
        <v-icon>mdi-application</v-icon>
      </v-btn>
      <v-btn icon @click.stop="fixed = !fixed">
        <v-icon>mdi-minus</v-icon>
      </v-btn>
      <v-toolbar-title v-text="title" />
      <v-spacer />
      <v-toolbar-title v-if="currentAddress">Current account: {{ currentAddress ? currentAddress : 'No account detected'}}</v-toolbar-title>
      <v-btn light @click="connectWallet()">
        Conectar Wallet
      </v-btn>
      <v-btn class="mx-2" light color="green" @click="callGreeter()">
        Saluda Greeter.sol
      </v-btn>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-navigation-drawer v-model="rightDrawer" :right="right" temporary fixed>
      <v-list>
        <v-list-item @click.native="right = !right">
          <v-list-item-action>
            <v-icon light>
              mdi-repeat
            </v-icon>
          </v-list-item-action>
          <v-list-item-title>Switch drawer (click me)</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-footer :absolute="!fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>
 
<script>
import detectEthereumProvider from "@metamask/detect-provider";
import { ethers } from "ethers";
import greeter from '@/solidity/artifacts/contracts/Greeter.sol/Greeter.json'

export default {
  name: 'DefaultLayout',
  data() {
    return {
      clipped: false,
      signer: null,
      currentAddress: null,
      provider: null,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        }
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Vuetify.js'
    }
  },
  mounted() {
    window.ethereum.on('accountsChanged', (accounts) => {
      window.location.reload();
    })
  },
  methods: {
    async callGreeter() {
      let tokenContract = new ethers.Contract(
        "0xa73bE78907c4110d696F9Aff6Da6A07aF963D200",
        greeter.abi,
        this.signer
      );

      let contractSigner = tokenContract.connect(this.signer);
      const g = await contractSigner.greet();
      alert(g);
    },
    async connectWallet() {
      const provider = new ethers.providers.Web3Provider(
        window.ethereum,
        "any"
      );
      // Prompt user for account connections
      await provider.send("eth_requestAccounts", []);
      const signer = provider.getSigner();
      this.currentAddress = await signer.getAddress()
      console.log("Account:", await signer.getAddress());
      this.signer = signer;
    }
  }
}
</script>
