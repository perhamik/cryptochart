<template>
  <trading-vue :data="chart" />
</template>
<script>
import TradingVue from 'trading-vue-js'

export default {
  name: 'Chart',
  components: { TradingVue },
  props: {
    chart: {}
  }
}
</script>
