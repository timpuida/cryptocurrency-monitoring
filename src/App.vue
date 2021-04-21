<template>
  <div id="app">
    <div class="header">
      <div class="logo">
        <img src="@/assets/images/LOGO.svg" alt="CRYPTO">
      </div>
    </div>
   
    <div class="main-content">
      <!-- <div > -->
        <h1 class="container dark-grey d-md-none">Панель котировок</h1>
      <!-- </div> -->
      <div class="plot">
        <div class="currency-title">
          <p class="h1">USD | {{currentCurrency}}</p>
          <div id="chart">
            <amchart
            v-if="ohlcvData"
            :chartProps="ohlcvData"
            :socketRecieve="dataRecieved">
            </amchart>
             <!-- <button @click="add">add</button> -->
             <br>
             <br>
             <button @click="closeSocket">Close socket</button>
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
              {{currency.change | toPrecision(2)}} %</td>
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
          price: '57093.95',
        },
        {
          name: 'BCH',
          change: '0',
          icon: 'up',
          price: '964',
        },
        {
          name: 'ETH',
          change: '0',
          icon: 'up',
          price: '964',
        },
        {
          name: 'XRP',
          change: '0',
          icon: 'up',
          price: '964',
        }
      ],
      apiKey: process.env.VUE_APP_KEY,
      baseUrl: 'wss://streamer.cryptocompare.com/v2',
      restUrl:'https://min-api.cryptocompare.com/data/v2/histominute',
      dataRecieved: null,
      ohlcvData: null,
      subscriptionMessage: {
          "action": "SubAdd",
          "subs": ["0~Coinbase~BTC~USD", "0~Coinbase~BCH~USD", "0~Coinbase~ETH~USD","0~Coinbase~XRP~USD"]
      },
      unSubscriptionMessage: {
          "action": "SubRemove",
         "subs": ["0~Coinbase~BTC~USD", "0~Coinbase~BCH~USD", "0~Coinbase~ETH~USD","0~Coinbase~XRP~USD"]
      },
      socket: null

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
      // this.unsubscribe(val.unsubscriptionMessage);
      // this.recieveCurrentData(val.subscriptionMessage)
    },
    recieveCurrentData(){
      const socket = new WebSocket(this.baseUrl+'?api_key='+this.apiKey);
      socket.onopen =() => {
        socket.send(JSON.stringify(this.subscriptionMessage))
      }
      socket.onmessage = (e) => {
        const msg = JSON.parse(e.data)
        // console.log(msg);
        this.currencies = this.currencies.map((item)=>{
          if (item.name===msg.FSYM){
            
            let diff = msg.P-item.price
            if((diff) !== 0){
              diff>0?item.icon='up':item.icon='down';

              item.change = Math.round((diff)/item.price *100 *100)/100;
              item.price = msg.P;
            }
            // console.log('msg.P:',msg.P,'item.price: ',item.price, ' (msg.P-item.price)/item.price',(msg.P-item.price)/item.price);
         }
          return item 
        })
        if (msg.TYPE==='0' && msg.FSYM===this.currentCurrency){
          this.dataRecieved=msg;
           console.log('fine: ',msg.FSYM);
          //  console.log(msg);
        }else{
          console.log(`debug ${msg.TYPE}`);
          // console.log(msg)
        }
      }
      socket.onclose = (e) => {
        console.log(`${e.code}; ${e.reason}; from mount`);
      }
      socket.onerror = (err) => console.log(err.message);
      this.socket = socket;
    },
    closeSocket(){
      if (this.socket){

        console.log('close click');
        this.socket.send(JSON.stringify(this.unSubscriptionMessage))
        this.socket.close(1000, 'btn clicked')
        this.socket.onclose = (e) => {
          console.log(`${e.code}; ${e.reason} reactive`);
        }
      }
    },
    // unsubscribe(){
    //   this.socket.send(JSON.stringify(this.unSubscriptionMessage))
    //   console.log('this.unsubscribe:',JSON.stringify(unsubscribeMsg));
    // },
    getOHLCV(cur){
      const options = {
        params: {
          fsym: cur,
          tsym: 'USD',
          limit: 99,
        }
      }
      this.$axios.get(this.restUrl,options)
      .then((res)=>{
          console.log(res);
          // console.log( Array.isArray(res.data.Data.Data))
          this.ohlcvData = res.data.Data.Data;
        })
      .catch((err)=>console.dir(err))
    }
  },
  filters: {
    toPrecision(val,to){
      return Number(val).toFixed(to);
    }
  }
}


</script>

<style lang="scss">
@import './assets/scss/main.scss';
</style>
