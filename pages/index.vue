<template>
  <div class="wrapper">
    <div class="container">
      <Chart :chart="this.$data.currencydata" />
    </div>
    <div class="status-bar">
      <span>{{ this.$data.time }}</span>
    </div>
    <div class="controlls">
      <div
        v-for="(obj, prop, index) in this.$data.allCurrencies"
        :key="prop"
        ref="cryptolist"
        :class="{'active': prop === 'BTC'}"
        @click="cryptoChange(prop, index)"
      >
        <span>{{ prop }}</span>
        <span>${{ obj[obj.length - 1].close }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import impdata from '../store/data.json'
const axios = require('axios')

const getFullURL = function (url, options) {
  const params = []
  for (const key in options) {
    params.push(`${key}=${options[key]}`)
  }
  return `${url}histominute?${params.join('&')}&api_key=${impdata.api_key}`
}

export default {
  async fetch () {
    this.list = await impdata.currencies
  },
  data () {
    return {
      time: Number,
      currencydata: {},
      list: [],
      allCurrencies: {},
      selectedTime: 'histominute',
      current: 'BTC'
    }
  },
  mounted () {
    this.getCurrentByAll()
    setInterval(() => this.tick(), 250)
    setInterval(() => this.getCurrentByAll(), 2000)
  },
  methods: {
    tick () {
      this.$set(this.$data, 'time', new Date().toLocaleString('ua-UA', { timeZone: 'Europe/Moscow' }))
      this.updateChart()
    },
    updateChart () {
      this.$data.currencydata = {
        ohlcv: this.$data.allCurrencies[this.$data.current].map(item => [
          item.time * 1000,
          item.open,
          item.high,
          item.low,
          item.close,
          item.volumeto
        ])
      }
    },
    cryptoChange (name, id) {
      this.$set(this.$data, 'current', name)
      this.updateChart()
      this.uncheck(this.$refs.cryptolist)
      this.check(this.$refs.cryptolist, id)
    },
    async getCurrentByAll () {
      const values = this.allCurrencies
      await this.list.map((name, id) => {
        const multiOptions = {
          fsym: name,
          tsym: 'USD',
          limit: 30
        }
        apiRequest(getFullURL(impdata.baseUrl, multiOptions))
          .then((result) => {
            if (result.data.Data.Data) {
              values[name] = [...result.data.Data.Data]
            } else {
              console.log(result)
            }
          })
      })
    },
    check (list, item) {
      list[item].className = 'active'
    },
    uncheck (list) {
      list.filter(el => el.className === 'active').forEach((element) => {
        element.className = ''
      })
    }
  }
}
async function apiRequest (link) {
  try {
    return await axios.get(link, impdata.headers)
  } catch (err) {
    console.error(err)
  }
}
</script>

<style>
.wrapper {
  margin: 0 auto;
  padding: 4rem;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}
.container {
  display: flex;
  justify-content: center;
}
.container, .controlls {
  width: 100%;
  margin: 2rem;
}
.controlls {
  display: grid;
  max-width: 800px;
  grid-gap: 1rem 2rem;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
}
.controlls > div {
  display: flex;
  flex-direction: column;
  background-color: rgba(10, 120, 10, 0.5);
  padding: 20px;
  color: #fff;
  border: 1px solid #fff;
  cursor: pointer;
  transition: .4s all;
}
.controlls > div:hover{
  background-color: rgba(10, 120, 10, 0.8);
}
.controlls > div.active{
  background-color: rgba(250, 120, 10, 0.8);
}

.controlls > div > span:first-child {
  font-size: 1.2em;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-shadow: 0 0 3px rgba(20, 20, 20, 0.7);
}
.controlls > div > span:last-child {
  margin-top: 5px;
  text-shadow: 0 0 1.5px rgba(20, 20, 20, 0.7);
  font-weight: 500;
}
</style>
