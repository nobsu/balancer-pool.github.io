<template>
  <UiTableTr>
    <div class="flex-auto text-left">
      <router-link
        :to="{ name: 'home', query: { token: [checksum], filter: 1 } }"
        class="text-white d-flex"
      >
        <Token :address="token.address" :symbol="token.symbol" class="mr-3" />
        {{ _ticker(token.checksum) }}
      </router-link>
    </div>
    <UiNum :value="token.weightPercent / 1e2" format="percent" class="column" />
    <UiNum :value="tokenBalance" class="column hide-sm" />
    <UiNum :value="myPoolBalance" class="column hide-sm hide-md" />
    <UiNum
      :value="myShareValue"
      format="currency"
      class="column hide-sm hide-md hide-lg"
    />
  </UiTableTr>
</template>

<script>
import { getAddress } from '@ethersproject/address';
import { normalizeBalance } from '@/helpers/utils';

export default {
  props: ['pool', 'token'],
  computed: {
    poolTokenBalance() {
      let balance = this.subgraph.poolShares[this.pool.id] || 0;
      const nodeBalance = this.web3.balances[getAddress(this.pool.id)];
      if (nodeBalance) balance = normalizeBalance(nodeBalance, 18);
      return balance;
    },
    checksum() {
      return getAddress(this.token.address);
    },
    myPoolBalance() {
      if (!this.poolTokenBalance) return 0;
      const balance =
        (this.poolTokenBalance / this.pool.totalShares) * this.token.balance;
      return this._precision(balance, this.token.checksum);
    },
    myShareValue() {
      const price = this.price.values[this.token.checksum];
      return price * this.myPoolBalance;
    },
    tokenBalance() {
      return this._precision(
        parseFloat(this.token.balance),
        this.token.checksum
      );
    }
  }
};
</script>
