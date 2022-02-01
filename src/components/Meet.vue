<template>
  <div class="white-b" id="main">
    <loading-screen v-if="!mounted" />
    <v-snackbar v-model="snackbar" :timeout="-1">
      <p style="font-size: 2rem; color: white">
        {{ snackBarMsg }}
      </p>

      <template v-slot:action="{ attrs }">
        <v-btn
          color="pink"
          text
          v-bind="attrs"
          @click="closeSnackbar"
          style="font-size: 2rem"
        >
          X
        </v-btn>
      </template>
    </v-snackbar>

    <div
      class="section height-100 y-center"
      style="padding-bottom: 2rem; padding-right: 2rem; margin-top: 20rem"
      id="intro"
    >
      <!-- CONTAINER -->
      <div class="container">
        <p class="tag m-b-4">Cryptocubans</p>
        <h1 class="m-b-5 m-l-3">Meet your Cryptocuban</h1>
        <div class="seperator-line"></div>
        <v-btn
          :disabled="loading"
          :loading="loading"
          v-if="!user"
          class="m-t-3 default-button button-filled"
          elevation="2"
          rounded
          x-large
          v-on:click="connectWallet"
        >
          CONNECT WALLET
        </v-btn>
        <div v-else class="row">
          <div class="col">
            <div>
              <p><strong>Address: </strong> {{ user.formattedAddress }}</p>
              <p><strong>Balance: </strong> {{ user.balance }}</p>
              <p><strong>Cryptocubans you own: </strong> {{ balanceOf }}</p>
            </div>
            <div class="seperator-line"></div>
            <div class="seperator-line"></div>
            <v-row align="center" justify="space-around">
              <v-col sm="12" md="4">
                <v-btn
                  class="default-button button-filled"
                  elevation="2"
                  rounded
                  x-large
                  :href="'https://etherscan.io/token/' + contractAddress"
                  target="_blank"
                >
                  SMART CONTRACT
                </v-btn>
              </v-col>
              <v-col sm="12" md="4">
                <v-btn
                  class="default-button button-filled"
                  elevation="2"
                  rounded
                  x-large
                  :href="'https://mint.cryptocubansocialclub.com'"
                  target="_blank"
                >
                  MINT
                </v-btn>
              </v-col>
              <v-col sm="12" md="4">
                <v-btn
                  class="default-button button-filled"
                  elevation="2"
                  rounded
                  x-large
                  :href="'https://opensea.io/collection/cryptocuban-social-club'"
                  target="_blank"
                >
                  OPENSEA
                </v-btn>
              </v-col>
            </v-row>
          </div>
        </div>
      </div>
      <!-- /CONTAINER -->
    </div>
    <!-- /SECTION ( INTRODUCTION ) -->
  </div>
  <!-- /SECTION CONTAINER -->
</template>
<script>
import { ethers } from "ethers";
// import axios from "axios";
import ABI from "../assets/contract/ABI.json";
import LoadingScreen from "./LoadingScreen.vue";

export default {
  name: "Meet",
  components: {
    "loading-screen": LoadingScreen,
  },
  data: () => {
    return {
      totalSupply: 0,
      user: null,
      provider: null,
      signer: null,
      contract: null,
      abi: ABI.abi,
      contractAddress: "0xca9eE3460D84Eac6C2F2284CFe3E3B35A2267d78", // Replace it with your contract address,
      loading: false,
      mounted: false,
      snackbar: false,
      snackBarMsg: "",
      balanceOf: 0,
    };
  },
  methods: {
    connectWallet: async function () {
      try {
        this.loading = true;
        // The "any" network will allow spontaneous network changes
        this.provider = new ethers.providers.Web3Provider(
          window.ethereum,
          "any"
        );

        // Best practice: reload the page if the network changes
        this.provider.on("network", (newNetwork, oldNetwork) => {
          // When a Provider makes its initial connection, it emits a "network"
          // event with a null oldNetwork along with the newNetwork. So, if the
          // oldNetwork exists, it represents a changing network
          if (oldNetwork) {
            window.location.reload();
          }
        });

        // Prompt user for account connections
        await this.provider.send("eth_requestAccounts", []);
        this.signer = this.provider.getSigner();
        const network = await this.provider.getNetwork();
        if (network.chainId !== 1) {
          throw new Error(
            `You are not on the correct network (${network.name}) please switch to the Ethereum Mainnet.`
          );
        }
        this.user = {
          address: await this.signer.getAddress(),
          balance: await this.signer.getBalance(),
          formattedAddress: "",
        };

        // formats the balance to be in ether
        this.user.balance = ethers.utils.formatEther(this.user.balance);
        this.user.balance = Math.round(this.user.balance * 1e4) / 1e4;

        // formats the address to be more readable
        this.user.formattedAddress =
          this.user.address.substring(0, 6) +
          "..." +
          this.user.address.substring(
            this.user.address.length - 4,
            this.user.address.length
          );

        // Loads the contract
        this.contract = new ethers.Contract(
          this.contractAddress,
          this.abi,
          this.signer
        );

        // Gets balance of the current address
        this.balanceOf = await this.contract.balanceOf(this.user.address);
      } catch (e) {
        this.handleError(e);
      } finally {
        this.loading = false;
      }
    },
    openSnackbar: function (msg) {
      this.snackbar = true;
      this.snackBarMsg = msg;
    },
    closeSnackbar() {
      this.snackbar = false;
      this.snackBarMsg = "";
    },
    handleError: function (e) {
      const msg = e?.error?.message || e.message;
      // checks if the user has metamask installed
      if (msg.includes("missing provider")) {
        this.openSnackbar(
          "Please install the MetaMask extension. If you are on mobile, open your MetaMask app and browse to this page."
        );
        return;
      }

      if (msg.includes("err: insufficient funds for gas * price + value")) {
        this.openSnackbar("You do not have enough balance.");
        return;
      }

      this.openSnackbar(msg);
    },
  },
  beforeMount() {},
  mounted() {
    this.mounted = true;
  },
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.white-b {
  background-color: #fdc60c;
}
::selection {
  background: #480058;
}

p {
  color: #480058;
}

p::selection {
  color: #fff;
}

.button-filled {
  background-color: #480058 !important;
  background: #480058;
  padding: 0 4.5rem !important;
  color: #fff !important;
  font-size: 2rem !important;
  min-width: none !important;
}

::v-deep .v-snack__wrapper {
  font-size: 2rem !important;
}
</style>
