<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title>
        <h3 class="mobile-span">Simple coincap table</h3>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn flat small @click="updateAssets">
        <span class="mr-2">update top Assets</span>
      </v-btn>
    </v-toolbar>

    <v-content>
      <HelloWorld v-bind:headers="tableHeaders" v-bind:assets="topAssets"/>
    </v-content>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld'
import axios from 'axios'

export default {
  name: 'App',
  components: {
    HelloWorld
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
      errorServer: ''
    }
  },
  computed: {
    topAssetsNames: function() {
      return this.topAssets.map((item) => item.id).join();
    }

  },
  methods: {
    getTopAssets: function() {
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
      this.getTopAssets();
      console.log('topAssets updated');
    },
    updateTopAssets: function(wsAnswer) {
      //console.log(wsAnswer);
      let ids = Object.keys(wsAnswer);
      this.topAssets.forEach((item, index) => {
        if (ids.includes(item.id)) {
          this.topAssets[index].priceUsd = wsAnswer[item.id];
        }
      })
    },
    createWsConnection: function(names) {
      let self = this;
      let socket = new WebSocket(`wss://ws.coincap.io/prices?assets=${names}`);
      // open connection - callback on opening connection
      socket.onopen = function(event) {
        console.log("Connection established");
      };
      // callback on message
      socket.onmessage = function(event) {
        let data = JSON.parse(event.data);
        self.updateTopAssets(data);
      };
      // callback on closing
      socket.onclose = function(event) {
        if (event.wasClean) { //
          console.log(`Connection closed cleanly, code=${event.code} reason=${event.reason}`);
        } else {
          console.log('Connection died');
        }
      };
      // callback on error
      socket.onerror = function(error) {
        console.log(`${error.message}`);
      };
    },

  },
  beforeMount() {
    // read data form localStorage if it is not empty
    if (localStorage["topAssets"]) {
      this.topAssets = JSON.parse(localStorage["topAssets"]);
      console.log('topAssets from localStorage received');
    }
    // read data from Rest and write it to localStorage
    if (!localStorage["xrp"]) {
      this.getTopAssets();
    }
  },
  mounted() {
    this.createWsConnection(this.topAssetsNames);
  }
}
</script>

<style>
/*  @media screen and (max-width: 400px) {
    .mobile-span {
      font-size: 70%
    }
  }*/

</style>
