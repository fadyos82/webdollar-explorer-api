<template>
  <div class="pools">
  <h2> Tracked WebDollar genesis addresses</h2>

  <table class="poolsTable" style="font-size:0.84em;">

    <tr>
      <td>Address</td>
      <td>Genesis Amount</td>
      <td>Current Amount</td>
      <td>Owner</td>
    </tr>

    <tr v-bind:key="genesis.miner_address" v-for="genesis in genesis_addresses" :class="genesisChange(genesis.initial_amount, genesis.current_amount)">
      <td align="left">
        <a :href="'#/miner/' + genesis.miner_address">{{ genesis.miner_address }} </a>
      </td>

      <td align="right">
        {{ formatMoneyNumber(genesis.initial_amount,0) }}
      </td>

      <td align="right">
        {{ formatMoneyNumber(genesis.current_amount,0) }}
      </td>

      <td align="left" style="text-align:center">
       {{ genesis.owner }}
      </td>
   </tr>
    <tr :class="genesisChange(total_genesis, tracked_genesis)">
      <td align="right">
        TOTAL (4,156,801,128)
      </td>
      <td align="right">
        {{ formatMoneyNumber(total_genesis,0) }}
      </td>
      <td align="right">
        {{ formatMoneyNumber(tracked_genesis,0) }}
      </td>
      <td align="right">
        ALL TRACKED
      </td>
    </tr>
  </table>

  </div>

</template>

<script>
import BlocksService from '@/services/BlocksService'
import Utils from '@/services/utils'
import SpecialAddresses from '@/services/SpecialAddresses'

function sleep (ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}
export default {
  name: 'genesis',

  components: { },

  data () {
    return {
      genesis_addresses: [],
      total_genesis: 0,
      tracked_genesis: 0,
      destroyed: false
    }
  },
  destroyed () {
    this.destroyed = true
  },
  mounted () {
    this.getGenesis()
  },
  methods: {
    formatMoneyNumber (number, decimals) {
      return Utils.formatMoneyNumber(number * 10000, decimals)
    },
    genesisChange (former, current) {
      if (current < former) {
        return 'genesisChanged'
      }
      return 'genesisUnchanged'
    },
    async getGenesis () {
      for (let i = 0; i < SpecialAddresses.genesis_addresses.length; i++) {
        await sleep(300)
        if (this.destroyed) {
          break
        }
        let genesisAddress = SpecialAddresses.genesis_addresses[i]
        let minerData = await BlocksService.fetchMiner(genesisAddress.address)
        let miner = minerData.data
        if (miner && miner.address && miner.blocks) {
          this.tracked_genesis += miner.balance
          this.total_genesis += genesisAddress.initial_amount
          this.genesis_addresses.push({
            initial_amount: genesisAddress.initial_amount,
            miner_address: genesisAddress.address,
            current_amount: miner.balance,
            owner: genesisAddress.owner
          })
        }
      }
    }
  }
}
</script>
<style type="text/css">
</style>
