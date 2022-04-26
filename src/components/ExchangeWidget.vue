<template>
  <q-card>
    <q-card-section>
      <div
        v-for="index in [0, 1]"
        :key="index"
        class="row items-center q-gutter-md"
      >
        <span v-if="index === 0">I have</span>

        <span v-else>I want</span>

        <q-select
          :model-value="wData.ticker[index]"
          :options="options[index].value"
          @update:model-value="applyChange('ticker', index, $event)"
        />

        <q-input
          :model-value="wData.amount[index]"
          placeholder="amount"
          type="number"
          :debounce="700"
          :loading="index === indexLoading"
          style="width: 160px;"
          @update:model-value="applyChange('amount', index, $event)"
        />
      </div>
    </q-card-section>
  </q-card>
</template>

<script>
import { defineComponent, reactive, computed, ref } from 'vue'

const tickers = ['USD', 'EUR', 'BTC', 'ETH']
const roundDigitsByTicker = {
  USD: 2,
  EUR: 2,
  BTC: 8,
  ETH: 8
}
const fee = 0.01

async function getRates (ticker1, ticker2) {
  const formData = new FormData()
  formData.append('currency_from', ticker1.toLowerCase())
  formData.append('currency_to', ticker2.toLowerCase())

  const answer = await fetch('https://crypto-verse.info/api/calculator/exchange/calculate/', {
    method: 'POST',
    headers: {
      Accept: 'application/json'
    },
    body: formData
  })
  const res = await answer.json()
  return res
}

const anotherIndex = index => (index + 1) % 2

const tickersExlude = excludingTicker => tickers
  .filter(_ => _ !== excludingTicker)

const round = (num, digits) => (Math.round(num * Math.pow(10, digits)) / Math.pow(10, digits))
  .toFixed(digits)

export default defineComponent({
  name: 'ExchangeWidget',
  props: {
    initTickers: {
      type: Array,
      default: () => ['USD', 'BTC']
    }
  },

  setup (props) {
    const wData = reactive({
      ticker: [...props.initTickers],
      amount: [null, null]
    })

    const options = [
      computed(() => tickersExlude(wData.ticker[1])),
      computed(() => tickersExlude(wData.ticker[0]))
    ]

    const indexLoading = ref(null)

    const applyChange = async (subject, index, value) => {
      wData[subject][index] = value
      if (wData.amount[index]) {
        const srcIndex = subject === 'ticker' ? anotherIndex(index) : index
        const targetIndex = anotherIndex(srcIndex)

        indexLoading.value = targetIndex
        const res = await getRates(wData.ticker[0], wData.ticker[1])
        indexLoading.value = null
        const rate = res.data.conversion_rate

        const k = srcIndex === 0 ? 1 : -1
        const x = Number(rate) ** k * Number(wData.amount[srcIndex])

        const targetRoundDigits = roundDigitsByTicker[wData.ticker[targetIndex]]
        wData.amount[targetIndex] = round(x + k * fee * x, targetRoundDigits)
      } else {
        wData.amount = [null, null]
      }
    }

    return {
      wData,
      tickers,
      options,
      indexLoading,
      applyChange
    }
  }
})
</script>
