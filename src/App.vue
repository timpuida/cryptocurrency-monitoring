<template>
  <div id="app">
    <div class="header">
      <div class="logo">
        <img src="@/assets/images/LOGO.svg" alt="CRYPTO">
      </div>
    </div>
    <div class="main-content">
        <h1 class="container dark-grey d-md-none">Панель котировок</h1>
      <div class="plot">
        <div class="currency-title">
          <p class="h1">USD | {{currentCurrency}}</p>
          <div id="chart">
            <amchart
            v-if="ohlcvData"
            :chartProps="ohlcvData"
            :socketRecieve="dataRecieved">
            </amchart>
             <br>
             <br>
          </div>
        </div>
      </div>
      <div class="left-side">
        <h1 class="container dark-grey d-none d-md-block">Панель котировок</h1>
        <table class="table">
          <thead>
           <tr>
             <th>Валютные пары</th>
             <th>Изменение</th>
             <th>Цена</th>
           </tr>
          </thead>
          <tbody>
            <tr 
            v-for="currency in currencies"
            :key="currency.name"
            @click="changeCurrency(currency.name)"
            :class="{active:currency.name==currentCurrency}"
            >
              <td>USD | {{currency.name}}</td>
              <td>
                <span class="icon"><img :src="require('./assets/images/'+currency.icon+'.svg')" alt=""></span>
              {{currency.change | abs() | toPrecision(2) }} %</td>
              <td> {{currency.price}} </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import amchart from './components/chart';

export default {
  name: 'App',
  components: {
    amchart
  },
  data() {
    return {
      currentCurrency: 'BTC',
      currencies: [
        {
          name: 'BTC',
          change: '0',
          icon: 'up',
          price: '0',
        },
        {
          name: 'BCH',
          change: '0',
          icon: 'up',
          price: '0',
        },
        {
          name: 'ETH',
          change: '0',
          icon: 'up',
          price: '0',
        },
      ],
      dataRecieved: null,
      ohlcvData: null,
      subscriptionMessage: {
          "action": "SubAdd",
          "subs": ["0~Coinbase~BTC~USD", "0~Coinbase~BCH~USD", "0~Coinbase~ETH~USD"]
      },
      unSubscriptionMessage: {
          "action": "SubRemove",
         "subs": ["0~Coinbase~BTC~USD", "0~Coinbase~BCH~USD", "0~Coinbase~ETH~USD"]
      },

    }
  },
  mounted() {
    this.getOHLCV(this.currentCurrency);
    this.recieveCurrentData()
  },
  methods: {
    changeCurrency(val){
       this.currentCurrency=val;
      this.getOHLCV(val);
    },
    recieveCurrentData(){
      const socket = new WebSocket(process.env.VUE_APP_BASEURL+'?api_key='+process.env.VUE_APP_KEY);
      socket.onopen =() => {
        socket.send(JSON.stringify(this.subscriptionMessage))
      }
      socket.onmessage = (e) => {
        const msg = JSON.parse(e.data)
        this.currencies = this.currencies.map((item)=>{
          if (item.name===msg.FSYM){
            const diff = msg.P-item.price
            if((diff) !== 0){
              diff>0?item.icon='up':item.icon='down';

              item.change = Math.round((diff)/item.price *100 *100)/100;
              item.price = msg.P;
            }
         }
          return item 
        })
        if (msg.TYPE==='0' && msg.FSYM===this.currentCurrency){
          this.dataRecieved=msg;
        }
      }
      socket.onclose = (e) => {
        console.log(`${e.code}; ${e.reason};`);
      }
      socket.onerror = (err) => alert(err);
    },
    getOHLCV(curr){
      const options = {
        params: {
          fsym: curr,
          tsym: 'USD',
          limit: 299,
        }
      }
      this.$axios.get(process.env.VUE_APP_RESTURL,options)
      .then((res)=> this.ohlcvData = res.data.Data.Data)
      .catch((err)=>console.log(err))
    }
  },
  filters: {
    toPrecision(val,to){
      return Number(val).toFixed(to);
    },
    abs(val){
      return Math.abs(val)
    }
  }
}


</script>

<style lang="scss">
@import './assets/scss/main.scss';
</style>
