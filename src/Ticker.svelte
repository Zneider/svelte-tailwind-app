<script type="ts">
  import dayjs from 'dayjs';

  interface Ticker {
    ticker: string,
    name: string,
    market: string,
    locale: string,
    type: string,
    currency: string,
    active: boolean,
    primaryExch: string,
    updated: number,
    codes: Object,
    url: string,
    price: number,
    change?: number,
    changePercent?: number,
    hasChanged?: boolean
  }
  interface Data {
    page?: number, 
    perPage?: number,
    count?: number,
    status?: string,
    tickers?: Array<Ticker>,
  }

  const url = 'ws://localhost:8999'

  const ws = new WebSocket(url)

  let data: Data = { tickers: []}
  $: tickers = data.tickers

  const parseJson = (str: string): Data => {
    try {
      return JSON.parse(str)
    } catch(_) {
      return { tickers: []}
    }
  }


  const handleMessage = (ev) => {
    if(ev.data) {
        const newData = parseJson(ev.data)
        const { tickers: newTickers } = newData
        const enrichedTickers = newTickers.map((ticker) => {
          const oldTicker = tickers.find(t => t.ticker === ticker.ticker)
          let change = 0
          let changePercent = 0
          let hasChanged = false
          if(oldTicker) {
            hasChanged = !!(ticker.price - oldTicker.price)
            change = hasChanged ? Number((ticker.price - oldTicker.price).toFixed(2)) : oldTicker.change
            changePercent = hasChanged ? ((ticker.price - oldTicker.price) / oldTicker.price) : oldTicker.changePercent
            hasChanged = hasChanged || oldTicker.hasChanged && (Date.now() - oldTicker.updated < 500 ) 
          }
          return {...ticker, change, changePercent, hasChanged}
        })
        data = {...newData, tickers: enrichedTickers }
    }
  }

  const handleError = (ev) => {
    console.error(ev)
  }
  
  ws.onmessage = handleMessage
  ws.onerror = handleError

  const currencyStyle = {style: 'currency', currency: 'USD', currencyDisplay: 'narrowSymbol'}

</script>

<style>
  @layer components {
    .row {
      @apply container flex w-screen justify-between bg-gray-300 py-5 px-2 transition-colors duration-300;
    }
    .row:nth-of-type(odd) {
        @apply bg-gray-100;
    }

    .entry {
      @apply flex-shrink-0 flex-grow-0 w-1/12 flex justify-end;
    }
    .entry:first-of-type {
      @apply justify-start;
    }

    .row[data-hasChanged="true"]:nth-of-type(odd) {
      @apply bg-yellow-100 transition-colors duration-300;
    }
    .row[data-hasChanged="true"]:nth-of-type(even) {
      @apply bg-yellow-200 transition-colors duration-300;
    }

    [data-change="positive"] .entry:nth-last-of-type(2), [data-change="positive"] .entry:nth-last-of-type(3) {
      @apply text-green-500;
    }

    [data-change="negative"] .entry:nth-last-of-type(2), [data-change="negative"] .entry:nth-last-of-type(3) {
      @apply text-red-500;
    }
  }
</style>

<div class="flex flex-col justify-between w-auto mx-auto">
{#each tickers as ticker (ticker.ticker)}
<div class="row" data-hasChanged={ticker.hasChanged} data-change={ticker.change > 0 ? 'positive' : 'negative' }>
  <span class="entry">{ticker.ticker}</span>
  <span class="flex-auto">{ticker.name}</span>
  <span class="entry">{new Intl.NumberFormat('da-DK', currencyStyle).format(ticker.price)}</span>
  <span class="entry">{new Intl.NumberFormat('da-DK', currencyStyle).format(ticker.change)}</span>
  <span class="entry">{new Intl.NumberFormat('da-DK', {style: 'percent', maximumFractionDigits: 3, minimumFractionDigits: 3}).format(ticker.changePercent)}</span>
  <span class="entry">{dayjs(new Date(ticker.updated)).format('HH:mm:ss')}</span>
</div>
{/each}
</div>



