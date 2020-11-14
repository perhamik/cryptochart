<template>
  <div class="wrapper">
    <div class="container">
      <Chart :data="this.$data.currencydata" />
      <div
        v-if="this.$data.timelist"
        class="timepick"
      >
        <div
          v-for="(title, index) in this.$data.timelist"
          :key="index"
          ref="timeref"
          :class="{'active': index === 0}"
          @click="timeChange(index)"
        >
          <span>{{ title }}</span>
        </div>
      </div>
    </div>
    <div
      v-if="this.$data.list"
      class="controlls"
    >
      <div
        v-for="(title, index) in this.$data.list"
        :key="index"
        ref="cryptoref"
        :class="{'active': index === 0}"
        @click="cryptoChange(index)"
      >
        <span>{{ title }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import impdata from '../store/data.json'
const axios = require('axios')

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
      timelist: []
    }
  },
  mounted () {
    axios
      .get(fullURL, impdata.headers)
      .then(
        (response) => {
          this.response = response.data.Data.Data
          this.currencydata = this.getData(this.response)
        },
        (error) => {
          this.answer = error
        }
      )
  },
  methods: {
    getData (source) {
      const ohlcv = {
        ohlcv: source.map(item => [
          item.time * 1000,
          item.open,
          item.high,
          item.low,
          item.close,
          item.volumeto
        ])
      }
      return ohlcv
    },
    timeChange (item) {
      this.uncheck(this.$refs.timeref)
      this.check(this.$refs.timeref, item)
    },
    cryptoChange (item) {
      this.uncheck(this.$refs.cryptoref)
      this.check(this.$refs.cryptoref, item)
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
const getFullURL = function (url, type, options) {
  const params = []
  for (const key in options) {
    params.push(`${key}=${options[key]}`)
  }
  return url + type + '?' + params.join('&')
}
const timeSelect = 'histominute'
const options = {
  fsym: 'BTC',
  tsym: 'USD',
  limit: 20
}

const fullURL = getFullURL(impdata.baseUrl, timeSelect, options)
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
  grid-gap: 1rem 2rem;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
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
/*
@media screen and (max-width: 576px){
  #trading-vue-js {
    max-width: 540px;
  }
}
@media screen and (max-width: 768px){
  #trading-vue-js {
    max-width: 540px;
  }
}
@media screen and (max-width: 990px){
  #trading-vue-js {
    max-width: 540px;
  }
}
@media screen and (max-width: 1200px){
  #trading-vue-js {
    max-width: 540px;
  }
}
@media screen and (min-width: 1201px){
} */
</style>
