<template>
  <trading-vue ref="tradeChart" :data="chart" />
</template>
<script>
import TradingVue from 'trading-vue-js'

export default {
  name: 'Chart',
  components: { TradingVue },
  props: {
    chart: Object
  },
  unmounted () {
    this.$refs.tradeChart.resetChart()
  }
}
</script>
