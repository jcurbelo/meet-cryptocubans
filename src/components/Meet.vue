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

    <v-dialog v-model="videoDialog" width="640">
      <iframe
        v-if="videoDialog && video"
        :src="video"
        width="640"
        height="480"
        allow="autoplay"
      ></iframe>
    </v-dialog>

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
              <p><strong># Cryptocubans you own: </strong> {{ balanceOf }}</p>
            </div>
            <div class="seperator-line"></div>
            <v-alert color="#480058" dark dense prominent v-if="balanceOf <= 0">
              <strong> Sorry, you don't own any Cryptocubans. </strong>
            </v-alert>
            <div v-else>
              <div class="seperator-line"></div>
              <v-row v-if="!revealed" align="center" justify="space-around">
                <v-col sm="12" md="4">
                  <v-btn
                    :disabled="loading"
                    :loading="loading"
                    class="default-button button-filled"
                    elevation="2"
                    rounded
                    x-large
                    target="_blank"
                    v-on:click="reveal"
                  >
                    REVEAL
                  </v-btn>
                </v-col>
              </v-row>
              <v-row v-else>
                <v-col sm="12" md="4" v-for="asset in assets" :key="asset.id">
                  <v-card class="mx-auto">
                    <v-img
                      :src="asset.image_preview_url"
                      max-height="200px"
                    ></v-img>

                    <v-card-title>
                      <p style="font-size: 3rem">
                        {{ asset.name }}
                      </p>
                    </v-card-title>

                    <v-card-actions>
                      <v-btn
                        text
                        color="#480058"
                        class="button-small"
                        target="_blank"
                        v-if="asset.video"
                        @click="
                          videoDialog = true;
                          video = asset.video;
                        "
                      >
                        VIDEO
                      </v-btn>
                      <v-btn
                        text
                        color="#480058"
                        class="button-small"
                        target="_blank"
                        v-if="asset.permalink"
                        :href="asset.permalink"
                      >
                        OPENSEA
                      </v-btn>
                      <v-btn
                        text
                        color="#480058"
                        class="button-small"
                        target="_blank"
                        v-if="asset.image_original_url"
                        :href="asset.image_original_url"
                      >
                        IPFS
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </v-col>
              </v-row>
            </div>
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
import Web3Modal from "web3modal";
import WalletConnectProvider from "@walletconnect/web3-provider";
import Torus from "@toruslabs/torus-embed";
import WalletLink from "walletlink";
import axios from "axios";
import ABI from "../assets/contract/ABI.json";
import LoadingScreen from "./LoadingScreen.vue";
import METAMASK_BASE64 from "../assets/config/logos";

const isMobile = () => {
  return deviceType() !== "desktop";
};

const deviceType = () => {
  const ua = navigator.userAgent;
  if (/(tablet|ipad|playbook|silk)|(android(?!.*mobi))/i.test(ua)) {
    return "tablet";
  }
  if (
    /Mobile|Android|iP(hone|od)|IEMobile|BlackBerry|Kindle|Silk-Accelerated|(hpw|web)OS|Opera M(obi|ini)/.test(
      ua
    )
  ) {
    return "mobile";
  }
  return "desktop";
};

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
      contractAddress: "0xca9eE3460D84Eac6C2F2284CFe3E3B35A2267d78",
      loading: false,
      mounted: false,
      snackbar: false,
      snackBarMsg: "",
      balanceOf: 1,
      assets: [],
      revealed: false,
      videoDialog: false,
      video: null,
    };
  },
  methods: {
    connectWallet: async function () {
      try {
        this.loading = true;
        const infuraId = "23cf1ohT2vMJPnvg9xycCGCt7hr";

        let providerOptions = {
          walletconnect: {
            package: WalletConnectProvider,
            options: {
              infuraId: infuraId,
            },
          },
          torus: {
            package: Torus,
          },
          walletlink: {
            package: WalletLink,
            options: {
              appName: "Cryptocuban Social Club",
              infuraId,
            },
          },
        };

        if (isMobile()) {
          providerOptions = {
            ...providerOptions,
            "custom-metamask": {
              display: {
                logo: METAMASK_BASE64,
                name: "MetaMask",
                description: "Connect to your MetaMask Wallet",
              },
              package: true,
              connector: () => {
                const url = `${window.location.origin}${window.location.pathname}`;
                window.location.href = `https://metamask.app.link/dapp/${url}`;
              },
            },
          };
        }

        const web3Modal = new Web3Modal({
          providerOptions,
        });

        const instance = await web3Modal.connect();
        if (!instance) {
          throw new Error("No wallet selected");
        }
        this.provider = new ethers.providers.Web3Provider(instance);

        if (instance.on) {
          instance.on("disconnect", () => {
            window.location.reload();
          });
          instance.on("accountsChanged", () => {
            window.location.reload();
          });
          instance.on("chainChanged", () => {
            window.location.reload();
          });
        }
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
    reveal: async function () {
      try {
        this.loading = true;

        const baseUrl =
          "https://us-central1-cryptocuban-social-club.cloudfunctions.net/";

        let res = await axios.get(
          `${baseUrl}auth?address=${this.user.address}`
        );
        const nonce = res.data.nonce;

        const signature = await this.signer.signMessage(nonce.toString());
        res = await axios.get(
          `${baseUrl}verify?address=${this.user.address}&signature=${signature}`
        );

        this.assets = res.data;
        // itereates through the assets and formats the balance to be in ether
        this.assets.forEach((asset) => {
          if (asset.video) {
            asset.video = asset.video.replace("/view?usp=drivesdk", "/preview");
          }
        });
        this.revealed = true;
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

.button-small {
  font-size: 2rem !important;
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
