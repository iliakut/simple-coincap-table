<template>
  <v-container>
    <v-layout justify-center>
      <v-flex xl8 lg10 md12>
        <v-data-table
        :headers="headers"
        :items="assets"
        class="elevation-1"
        hide-actions
      >
        <template slot="headerCell" slot-scope="props">
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
            <span v-on="on">
              {{ props.header.text }}
            </span>
            </template>
            <span>
                {{ props.header.text }}
              </span>
          </v-tooltip>
        </template>
        <template v-slot:items="props">
          <td>{{ props.item.name }}</td>
          <td>{{ '$' + optimizeNumber(props.item.priceUsd) }}</td>
          <td>{{ '$' + optimizeNumber(props.item.marketCapUsd) }}</td>
          <td>{{ '$' + optimizeNumber(props.item.volumeUsd24Hr) }}</td>
        </template>
      </v-data-table>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  export default {
    name: 'CoinTable',
    props: {
      headers: {
        type: Array,
        default: function() {
          return [{
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
            }]
        }
      },
      assets: {
        type: Array,
        default: function() {
          return [{
            name:"Currency",
            priceUsd:"0.0",
            marketCapUsd:"0.0",
            volumeUsd24Hr:"0.0"
          }]
        }
      }
    },
    data () {
      return {}
    },
    methods: {
      optimizeNumber: function(number) {
        /*
        * transform Price into short
        */
        let tempNumber = Number(number);
        // if trillion
        if ( tempNumber > 1000000000000 ) {
          let pointIndex = number.indexOf(".");
          let trillions = Number(number.slice(0, pointIndex - 9)) / 1000;
          return `${trillions.toFixed(1)}t`;
        }
        // if billion
        else if ( tempNumber > 1000000000) {
          let pointIndex = number.indexOf(".");
          let billions = Number(number.slice(0, pointIndex - 6)) / 1000;
          return `${billions.toFixed(1)}b`;
        }
        // if million
        else if ( tempNumber > 1000000) {
          let pointIndex = number.indexOf(".");
          let millions = Number(number.slice(0, pointIndex - 3)) / 1000;
          return `${millions.toFixed(1)}m`;
        }
        // if thousand
        else if ( tempNumber > 1000 ) {
          let pointIndex = number.indexOf(".");
          let millions = Number(number.slice(0, pointIndex + 4));
          return `${millions.toFixed(3)}`;
        }
        // if about 1
        else if ( tempNumber > 1 ) {
          return tempNumber.toFixed(3);
        }
        // if <= 1
        return tempNumber.toFixed(9);
  }
    }
  }
</script>

<style>
  /*
  * table style
  */
  thead{
    table-layout: fixed;
  }
  tbody{
    display:block;
    width: 100%;
    height: 70vh;
    overflow: auto;
  }
  thead tr {
    display: block;
    background-color: rgba(255, 116, 0, 0.27);
  }
  th, td {
    width: 10%;
  }
  tr th {
    position: relative;
    top: 20px;
  }
  /*
  * mobile view
  */
  @media screen and (max-width: 640px) {
    th:nth-child(n+3) {
      display: none;
    }
    td:nth-child(n+3) {
      display: none;
    }
    tbody{
      height: 50vh;
    }
  }
  /*
  * scrollbar style
  */
  tbody:hover::-webkit-scrollbar-thumb { background-color: rgba(255, 75, 3, 0.86); }
  ::-webkit-scrollbar { width: 7px; height: 7px;}
  ::-webkit-scrollbar-thumb { height: 50px; border-radius: 3px;}
</style>
