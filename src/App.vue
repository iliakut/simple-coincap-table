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
let socket = new WebSocket("wss://ws.coincap.io/prices?assets=bitcoin,ethereum,monero,litecoin"); // create
socket.onopen = function(e) { // open connection - callback on opening connection
  //console.log("Connection established, send -> server");
  socket.send("My name is John"); // message to the server
};
socket.onmessage = function(event) { // callback on message
  //console.log(`Data received: ${event.data} <- server`);
};
socket.onclose = function(event) { // callback on closing
  if (event.wasClean) { //
   // console.log(`Connection closed cleanly, code=${event.code} reason=${event.reason}`);
  } else {
    // e.g. server process killed or network down
    // event.code is usually 1006 in this case
   // console.log('Connection died');
  }
};
socket.onerror = function(error) { // callback on error
  //console.log(`${error.message}`);
};

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
      tableData: [],
      errorServer: ''
    }
  },
  computed: {
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
      this.fillHeaders();
    },
    fillHeaders: function() {
      for (let item of this.topAssets) {
        this.tableData.push({
          name: item.name,
          priceUsd: item.priceUsd,
          marketCapUsd: item.marketCapUsd,
          volumeUsd24Hr: item.volumeUsd24Hr
        })
      }
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
