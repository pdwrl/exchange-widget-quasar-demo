<template>
  <q-page class="flex flex-center">
    <ExchangeWidget
      :initTickers="['USD', 'BTC']"
    />
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import ExchangeWidget from 'src/components/ExchangeWidget'

export default defineComponent({
  name: 'PageIndex',
  components: { ExchangeWidget }
})
</script>
