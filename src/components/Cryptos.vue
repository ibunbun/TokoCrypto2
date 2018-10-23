<template>
  <div class="crypto-table-wrapper">
    <h2>Top 100 Cryptocurrency in the Market</h2>
    <h6>Tip: click on the column title to sort specifically by asc/desc</h6>
    <div style="overflow-x: auto">
      <table class="crypto-table">
        <thead>
          <tr>
            <th v-on:click="sort('rank')" data-sort="desc" title="Sort by Rank">Rank&nbsp;&nbsp;<span class="sort"><span>&#9650;</span><span>&#9660;</span></span></th>
            <th v-on:click="sort('name')" data-sort="asc" title="Sort by Name">Name&nbsp;&nbsp;<span class="sort"><span>&#9650;</span><span>&#9660;</span></span></th>
            <th v-on:click="sort('price_idr')" data-sort="asc" title="Sort by Price">Price&nbsp;&nbsp;<span class="sort"><span>&#9650;</span><span>&#9660;</span></span></th>
            <th v-on:click="sort('percent_change_24h')" data-sort="desc" title="Sort by Change in 24 hour">Change (24 hour)&nbsp;&nbsp;<span class="sort"><span>&#9650;</span><span>&#9660;</span></span></th>
          </tr>
        </thead>
        <tbody v-if="cryptos && cryptos.length">
          <tr v-for="crypto of cryptos" :data-price="crypto.price_idr" :data-name="crypto.name" :key="crypto.name" v-on:click="$emit('clickCrypto', crypto)">
            <td data-title="rank">{{ crypto.rank }}</td>
            <td data-title="name" class="name-col">{{ crypto.name }}</td>
            <td data-title="price" class="price-col">Rp {{ parseFloat(crypto.price_idr).toLocaleString('id', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
            <td v-if="Math.sign(crypto.percent_change_24h) === -1" data-title="change" class="dec">{{ crypto.percent_change_24h }}</td>
            <td v-else data-title="change" class="inc">{{ crypto.percent_change_24h }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Cryptos',
  data() {
    return {
      cryptos: [],
      errors: []
    }
  },
  created() {
    this.loadCryptos()
    // Fetch new cryptos every 5 min
    setInterval( ()=> this.loadCryptos(), 300000 )
  },
  methods: {
    loadCryptos: function() {
      axios.get('https://api.coinmarketcap.com/v1/ticker/?limit=100&convert=IDR')
      .then(response => {
        this.cryptos = response.data
      })
      .catch(e => {
        this.errors.push(e)
      })
    },
    sort: function(sortKey) {
      const data = this.cryptos
      // Get sort direction
      let order = event.target.dataset['sort']

      data.sort(this.compareValues(sortKey,order))

      let newOrder = ""

      if (order === 'asc') {
        newOrder = 'desc'
      } else {
        newOrder = 'asc'
      }

      // Update column title sort direction
      event.target.setAttribute('data-sort',newOrder)

      this.cryptos = data
    },
    compareValues: function(key, order='asc') {
      return function(a, b) {

        let varA = "";
        let varB = "";

        if (key === 'name'){
          varA = a[key].toUpperCase()
          varB = b[key].toUpperCase()
        } else {
          varA = parseFloat(a[key])
          varB = parseFloat(b[key])
        }

        let comparison = 0;

        if (varA > varB) {
          comparison = 1;
        } else if (varA < varB) {
          comparison = -1;
        }
        
        return (
          (order === 'desc') ? (comparison * -1) : comparison
        )
      }
    }
  },
}
</script>
