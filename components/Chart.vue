<template>
  <trading-vue :data="data" />
</template>
<script>
import TradingVue from 'trading-vue-js'
export default {
  name: 'Chart',
  components: { TradingVue },
  props: {
    data: {}
  }
}
</script>
