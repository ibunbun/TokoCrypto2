<template>
  <div class="wrapper">
    <div class="header">
      <div class="logo">
        <h1 class="title">TokoCrypto</h1>
        <p>Jual beli cryptocurrency yang aman dan terpercaya.</p>
      </div>
      <div class="menu">
        <a href="" v-on:click="toggleEwallet()" class="ewallet-button">eWallet</a>
      </div>
    </div>

    <Cryptos v-on:clickCrypto="handleForm" />

    <!-- eWallet pop up -->
    <div class="ewallet" v-bind:class="{'show': ewalletShow,  '': !ewalletShow}">
      <h2>Your Current Balance</h2>
      <table class="ewallet-table">
        <thead>
          <tr>
            <th>Currency</th>
            <th>Amount</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(value, key) in ewallet" v-if="key === 'Rupiah' || (key !== 'Rupiah' && value > 0)" :key="key">
            <td>{{ key }}</td>
            <td>{{ parseFloat(value).toLocaleString('id', { minimumFractionDigits: 0, maximumFractionDigits: 15}) }}</td>
          </tr>
          <tr v-else style="display: none;"></tr>
        </tbody>
      </table>
      <a href="" v-on:click="toggleEwallet()" class="close-button">CLOSE</a>
    </div>

     <!--Transaction form pop up -->
    <div class="transact-form" v-bind:class="{'show': formShow,  '': !formShow}">
      <h2>{{ cryptName }}</h2>
      <p>Price: {{ cryptPrice }} IDR</p>
      <div class="transact-option">
        <a href="" v-on:click="toggleTransact()" class="buy-tab" v-bind:class="{'': transactShow,  'active': !transactShow}">BUY</a>
        <a href="" v-on:click="toggleTransact()" class="sell-tab" v-bind:class="{'active': transactShow,  '': !transactShow}">SELL</a>
      </div>
      <div class="buy-form" v-bind:class="{'': transactShow,  'show': !transactShow}">
        <p class="balance">Your Rupiah: {{ ewallet['Rupiah'] }}</p>
        <p>Input Rupiah:</p>
        <input id="rupiahVal" type="number" v-on:input="handleCalc('buy')" />
        <p>Get {{ cryptName }}:</p>
        <input type="text" :value="cryptGet" disabled />
        <a href="" v-on:click="handleBuy()">BUY</a>
      </div>
      <div class="sell-form" v-bind:class="{'show': transactShow,  '': !transactShow}">
        <p class="balance">Your {{ cryptName }}: {{ ewallet[cryptName] ? ewallet[cryptName] : '0' }}</p>
        <p>Input {{ cryptName }}:</p>
        <input id="cryptVal" type="number" v-on:input="handleCalc('sell')" />
        <p>Get Rupiah:</p>
        <input type="text" :value="cryptGetRp" disabled />
        <a href="" v-on:click="handleSell()">SELL</a>
      </div>
      <div style="text-align: center"><a href="" v-on:click="toggleForm()" class="close-button">CLOSE</a></div>
    </div>

    <!-- overlay -->
    <div class="overlay" v-bind:class="{'show': overlayShow,  '': !overlayShow}"></div>

  </div> <!-- .wrapper -->
</template>

<script>
import Cryptos from './components/Cryptos.vue'

export default {
  name: 'app',
  components: {
    Cryptos
  },
  data() {
      return {
        ewallet: {'Rupiah':'10000000'},
        cryptName: '',
        cryptPrice: '0',
        cryptGet: '0',
        cryptGetRp: '0',
        ewalletShow: false,
        formShow: false,
        transactShow: false,
        overlayShow: false,
      }
  },
  created: function() {
    // Check for data in local storage then put it into state
    let localEwallet = localStorage.getItem('ewallet');
    if (localEwallet) {
      localEwallet = JSON.parse(localEwallet);
      this.ewallet = localEwallet
    }
  },
  methods: {
    toggleEwallet: function() {
      event.preventDefault();
      this.ewalletShow = !this.ewalletShow
      this.overlayShow = !this.overlayShow
    },
    toggleTransact: function() {
      event.preventDefault()
      this.transactShow = !this.transactShow
      this.cryptGet = '0'
      this.cryptGetRp = '0'
      // Clear input box
      document.getElementById("cryptVal").value = ""
      document.getElementById("rupiahVal").value = ""
    },
    toggleForm: function() {
      event.preventDefault()
      this.formShow = !this.formShow
      this.overlayShow = !this.overlayShow
      this.cryptGet = '0'
      this.cryptGetRp = '0'
      // Clear input box
      document.getElementById("cryptVal").value = "";
      document.getElementById("rupiahVal").value = "";
    },
    handleForm: function(crypto) {
      this.cryptName = crypto.name
      this.cryptPrice = crypto.price_idr
      this.toggleForm()
    },
    handleCalc: function(transact) {
      let inputVal = event.target.value
      let price = this.cryptPrice

      // Validate positive value
      if (Math.sign(inputVal) === 1 && Math.sign(price) === 1){

        // Convert string to number
        inputVal = parseFloat(inputVal);
        price = parseFloat(price)
        
        if (transact === 'buy') {

          // Calculate how many crypto user will get
          let get = inputVal/price

          this.cryptGet = get
        } else {

          // Calculate how many rupiah user will get
          let get = price * inputVal

          this.cryptGetRp = get
        }
      } else {
        // Clear conversion box
        this.cryptGet = '0'
        this.cryptGetRp = '0'
      }
    },
    handleBuy: function() {
      event.preventDefault()
      let rupiah = parseFloat(this.ewallet['Rupiah'])
      let valRupiah = parseFloat(document.getElementById("rupiahVal").value)
      let cryptName = this.cryptName

      if (valRupiah > rupiah) {
        alert("Saldo tidak mencukupi.")
      } else {
        if ( valRupiah > 0 && this.cryptGet > 0) {

          let data = this.ewallet
          rupiah = rupiah - valRupiah
          data['Rupiah'] = rupiah

          // Check if crypto already exist in user's eWallet
          if (data[cryptName] > 0) {
            // Update crypto amount
            data[cryptName] = data[cryptName] + this.cryptGet
          } else {
            // Insert new crypto
            data[cryptName] = this.cryptGet
          }
          
          this.ewallet = data,
          this.cryptGet = '0'

          // Update user's eWallet local storage
          localStorage.setItem("ewallet", JSON.stringify(data));

          // Clear input box
          document.getElementById("rupiahVal").value = "";

          let msg = "Selamat, transaksi Anda berhasil!\nSaldo Rupiah Anda saat ini: " + rupiah + "\nSaldo " + cryptName + " Anda saat ini: " + data[cryptName];

          alert(msg);
        } else {
          alert("Masukkan jumlah rupiah.");
        }
      }
    },
    handleSell: function() {
      event.preventDefault();
      let rupiah = parseFloat(this.ewallet['Rupiah']);
      let valCrypt = parseFloat(document.getElementById("cryptVal").value);
      let cryptName = this.cryptName;

      let data = this.ewallet;
      let crypt = 0;

      // Check if crypto exist in user's eWallet
      if (data[cryptName] > 0) {
        crypt = data[cryptName];
      }

      if (valCrypt > crypt) {
        alert("Saldo tidak mencukupi.");
      } else {

        if ( valCrypt > 0 && this.cryptGetRp > 0) {

          rupiah = rupiah + this.cryptGetRp;
          crypt = crypt - valCrypt;

          data[cryptName] = crypt;
          data['Rupiah'] = rupiah;

          this.ewallet = data
          this.cryptGetRp = '0'

          // Update user's eWallet local storage
          localStorage.setItem("ewallet", JSON.stringify(data));

          // Clear input box
          document.getElementById("cryptVal").value = "";

          let msg = "Selamat, transaksi Anda berhasil!\nSaldo Rupiah Anda saat ini: " + rupiah + "\nSaldo " + cryptName + " Anda saat ini: " + crypt;

          alert(msg);
        } else {
          alert("Masukkan jumlah yang ingin dijual.");
        }
      }
    },
  }
}
</script>

<style>
  @import './assets/css/reset.css';
  @import './assets/css/App.css';
</style>
