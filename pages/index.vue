<template>
  <div class="wrapper">
    <div class="container">
      <Chart :chart="this.$data.currencydata" />
    </div>
    <div class="controlls">
      <div
        v-for="(title, index) in this.$data.list"
        :key="index"
        ref="cryptoref"
        :class="{'active': index === 0}"
        @click="cryptoChange(index, title)"
      >
        <span>{{ title }}</span>
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
    this.timelist = await impdata.timemarks
  },
  data () {
    return {
      response: null,
      currencydata: {},
      list: [],
      timelist: [],
      selectedTime: 'histominute',
      options: {
        fsym: 'BTC',
        tsym: 'USD',
        limit: 700
      }
    }
  },
  mounted () {
    this.reviewChanges()
  },
  methods: {
    getFullURL (url, type, options) {
      const params = []
      for (const key in options) {
        params.push(`${key}=${options[key]}`)
      }
      return url + type + '?' + params.join('&')
    },
    reviewChanges () {
      const promise = apiRequest(getFullURL(impdata.baseUrl, this.selectedTime, this.options))
      promise.then((result) => {
        if (result.data.Data.Data) {
          this.$data.currencydata = {
            ohlcv: result.data.Data.Data.map(item => [
              item.time * 1000,
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
.timepick {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  margin-left: 2rem;
}
.controlls > div,
.timepick > div {
  background-color: rgba(80, 80, 80, 0.4);
  padding: 20px;
  color: #fff;
  border: 1px solid #fff;
  cursor: pointer;
  transition: .7s all;
}
.controlls > div:hover,
.timepick > div:hover {
  background-color: rgba(50, 50, 50, 0.7);
}
.controlls > div.active,
.timepick > div.active {
  background-color: rgba(250, 150, 0, 0.7);
}
</style>
