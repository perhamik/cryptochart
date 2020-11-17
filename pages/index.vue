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
        v-for="(obj, index) in this.$data.allCurrencies"
        :key="index"
        ref="cryptoref"
        :class="{'active': index === 0}"
        @click="cryptoChange(index, obj.currency)"
      >
        <span>{{ obj.currency }}</span>
        <span>${{ obj.exchrate }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import impdata from '../store/data.json'
const axios = require('axios')

const getFullURL = function (url, type, options) {
  const params = []
  for (const key in options) {
    params.push(`${key}=${options[key]}`)
  }
  return url + type + '?' + params.join('&')
}

export default {
  async fetch () {
    this.list = await impdata.currencies
  },
  data () {
    return {
      time: Number,
      response: null,
      currencydata: {},
      list: [],
      allCurrencies: [],
      selectedTime: 'v2/histominute',
      options: {
        fsym: 'BTC',
        tsym: 'USD',
        limit: 100
      },
      mulOptions: {
        fsyms: '',
        tsyms: 'USD',
        limit: 1
      }
    }
  },
  mounted () {
    this.tick()
    this.reviewChanges()
    setTimeout(() => this.getCurrentByAll(), 50)
    setInterval(() => this.tick(), 2000)
  },
  methods: {
    tick () {
      const current = new Date().toLocaleString('ua-UA', { timeZone: 'Europe/Moscow' })
      this.$set(this.$data, 'time', current)
      this.reviewChanges()
      this.getCurrentByAll()
    },
    reviewChanges () {
      const promise = apiRequest(getFullURL(impdata.baseUrl, this.selectedTime, this.options))
      promise.then((result) => {
        if (result.data.Data.Data) {
          this.$data.currencydata = {
            ohlcv: result.data.Data.Data.map(item => [
              (item.time + 7200) * 1000,
              item.open,
              item.high,
              item.low,
              item.close,
              item.volumeto
            ])
          }
        }
      })
    },
    cryptoChange (index, item) {
      this.options.fsym = item
      this.reviewChanges()
      this.uncheck(this.$refs.cryptoref)
      this.check(this.$refs.cryptoref, index)
    },
    getCurrentByAll () {
      const values = this.allCurrencies || []
      const queue = this.list.map((name, id) => {
        if (values.length < 15) {
          values.push({ currency: name, exchrate: null })
        }
        const multiOptions = {
          fsym: name,
          tsym: 'USD',
          limit: 1
        }
        return getFullURL(impdata.baseUrl, this.selectedTime, multiOptions)
      })
      queue.map((item, id) => {
        return apiRequest(item)
          .then((result) => {
            values[id].exchrate = result.data.Data.Data[1].close
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
