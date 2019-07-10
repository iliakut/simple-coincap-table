<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title>
        <h3 class="mobile-span">Simple coincap table</h3>
      </v-toolbar-title>

      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-spacer></v-spacer>
          <v-btn flat small @click="updateAssets" v-on="on">
            <span class="mr-2">update top Assets</span>
          </v-btn>
        </template>
        <span>update 15 currencies with the biggest market cup
        and rewrite localStorage with it</span>
      </v-tooltip>
    </v-toolbar>

    <v-content>
      <CoinTable v-bind:headers="tableHeaders" v-bind:assets="topAssets"/>
    </v-content>
  </v-app>
</template>

<script>
import CoinTable from './components/CoinTable'
import axios from 'axios'

export default {
  name: 'App',
  components: {
    CoinTable
  },
  data () {
    return {
      topAssets: [],
      tableHeaders: [
        {
          text: 'Name',
          align: 'left',
          value: 'name'
        },
        {
          text: 'Price',
          value: 'priceUsd'
        },
        {
          text: 'Market Cap',
          value: 'marketCapUsd'
        },
        {
          text: 'Volume 24Hr',
          value: 'volumeUsd24Hr'
        }
      ],
      errorServer: '',
      socket: null
    }
  },
  computed: {
    topAssetsNames: function() {
      return this.topAssets.map((item) => item.id).join();
    },
    socketReadyState: function() {
      return this.socket.readyState;
    }
  },
  methods: {
    getTopAssets: function() {
      /*
      * get assets from REST
      * fill local storage
      * */
      axios
        .get('https://api.coincap.io/v2/assets')
        .then(response => {
          this.topAssets = response.data.data.splice(0, 15);
          localStorage["topAssets"] = JSON.stringify(this.topAssets);
          console.log('topAssets from server received');
        })
        .catch(error => {
          this.errorServer = error.message;
          console.log(error);
        })
    },
    updateAssets: function() {
      /*
      * update top Assetds and fill
      * local storage with new data
      * */
      localStorage.removeItem("topAssets");
      this.getTopAssets();
      console.log('topAssets updated');
    },
    updatePrice: function(wsAnswer) {
      /*
      * update price of top 15 Assets
      */
      let ids = Object.keys(wsAnswer);
      this.topAssets.forEach((item, index) => {
        if (ids.includes(item.id)) {
          this.topAssets[index].priceUsd = wsAnswer[item.id];
        }
      })
    },
    createWsConnection: function(names) {
      /*
      * create WebSocket connection
      * and define its callbacks
      */
      let self = this;
      this.socket = new WebSocket(`wss://ws.coincap.io/prices?assets=${names}`);
      // open connection - callback on opening connection
      this.socket.onopen = function(event) {
        console.log("Connection established");
      };
      // callback on message
      this.socket.onmessage = function(event) {
        let data = JSON.parse(event.data);
        self.updatePrice(data);
      };
      // callback on closing
      this.socket.onclose = function(event) {
        if (event.wasClean) { //
          console.log(`Connection closed cleanly, code=${event.code} reason=${event.reason}`);
        } else {
          console.log('Connection died');
          self.errorServer = "error WebSocket died";
        }
      };
      // callback on error
      this.socket.onerror = function(error) {
        self.errorServer = "error WebSocket";
        console.log(`${error.message}`);
      };
    }
  },
  beforeMount() {
    // read data form localStorage if it is not empty
    if (localStorage["topAssets"]) {
      this.topAssets = JSON.parse(localStorage["topAssets"]);
      console.log('topAssets from localStorage received');
    }
    // read data from Rest and write it to localStorage
    if (!localStorage["topAssets"]) {
      this.getTopAssets();
    }
  },
  mounted() {
    // create WebSocket connection with top Assets
    this.createWsConnection(this.topAssetsNames);
  }
}
</script>
