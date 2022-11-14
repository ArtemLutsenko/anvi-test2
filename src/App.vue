<template>
  <div class="container">
    <h2>Список возможных сделок</h2>
    <app-spinner v-if="isLoading"></app-spinner>
    <template v-else>
      <p v-if="symbols.length">{{exchangeHistory.length}} из {{symbols.length}}</p>
      <ul>
        <li v-for="(item, index) in exchangeHistory" :key="index">{{item}}</li>
      </ul>
    </template>
  </div>
</template>

<script>
import axios from 'axios'
import AppSpinner from "@/components/spinner";

export default {
  name: 'App',
  components: {AppSpinner},
  data() {
    return {
      symbols: [],
      successStory: [],
      exchangeHistory:[],
      isLoading: false
    }
  },
  methods: {
    async getExchangeInfo() {
      this.isLoading = true
      try{
        const exchangeInfo = await axios.get('https://api.binance.com/api/v3/exchangeInfo')
        this.symbols = exchangeInfo.data.symbols.map(symbol => {
          return {symbol: symbol.symbol, base: symbol.baseAsset, quote: symbol.quoteAsset}
        })
      } catch (e){
        console.log(e.message)
      } finally {
        this.isLoading = false
      }
      this.getPairInfo()
    },
    async getPairInfo(){
      //const slicedArray = this.symbols.slice(0, 10);
      for (const symbol of this.symbols) {
        try {
          const pairInfo = await axios.get(`https://api.binance.com/api/v1/depth?symbol=${symbol.symbol}`)
          if (pairInfo && pairInfo.data?.asks[0] && pairInfo.data?.bids[0]) {
            const askInfo = pairInfo.data?.asks[0]
            const bidInfo = pairInfo.data?.bids[0]
            const [result, persent] = this.makeOperation(askInfo, bidInfo)

            this.exchangeHistory.push(`${symbol.base} -> ${symbol.quote} -> ${symbol.base} Принесет ${result >= 0 ? 'прибыль' : 'убыток'} ${persent.toFixed(2)}%(${result.toFixed(7)}${symbol.base})`)
          }
        } catch (e){
          console.log(e.message)
        }
      }
    },
    makeOperation(ask, bid) {
      const quantity = Math.min(ask[1], bid[1])
      const result = bid[0] * quantity - ask[0] * quantity
      const spendedMoney = bid[0] * quantity
      const persent = Math.abs(result / spendedMoney * 100)

      return [result, persent]
    }
  }
}
</script>

<style>
.container{
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}
ul{
  margin: 0;
  padding: 0;
}
li{
  margin: 0;
  padding: 0;
  list-style: none;
}
</style>
