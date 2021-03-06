<template>
  <div class="checkout mb-4">
    <div class="mb-2">
      <i class="iconfont icon-clock mr-2"/>
      {{ pendingAmount ? timeToWait : (updateTime) | ms }}
    </div>
    <button
      :class="{ progress: pendingAmount }" :disabled="isLoading || pendingAmount || notEnough || inProgress"
      @click="handleSubmit()"
      class="button btn-block button-green mb-2"
    >
        <template v-if="isLoading || waitingConfirmation">
      <SmallLoading/>
    </template>
      <template v-else>
        <i class="iconfont icon-person"/>
        <span v-if="!isLoading && pendingAmount === 0">
                {{ notEnough ? 'Miss resources' : 'Recruit' }}  </span>
     <span v-if="pendingAmount >0">Recruiting {{pendingAmount}}</span> <div v-else-if="isLoading"><div class="pt-2">    <SmallLoading/></div></div>
      </template>
    </button>

    <div class="mb-2">Instant recruit</div>
    <button
      :disabled="isLoading" v-if="steemAccount"
      @click="handleRequestPayment()"
      class="button btn-block button-blue mb-2"
    >
      <i class="iconfont icon-zap"/>
      ${{ price * quantity | amount }} =
      {{ priceInSteem | amount }} STEEM
    </button> 
    <button
      :disabled="isLoading || notEnoughFuture ||pendingAmount >0"
      @click="handleSubmit('future')"
      class="button btn-block button-yellow mb-2"
    >
    <img class="futureicon" src="/img/icons/future.png"/>
      ${{ ((price - price /100*20) * this.quantity) | amount }} =
      {{ priceInFuture | amount }} FUTURE
    </button>
  </div>
</template>

<script>
import { mapActions } from 'vuex';
// import { utils } from 'drugwars';

export default {
  props: ['id', 'level', 'coeff', 'inProgress', 'price', 'quantity', 'notEnough'],
  data() {
    return {
      isLoading: false,
      waitingConfirmation: false,
    };
  },
  watch: {
    inProgress(val) {
      if (val) {
        this.waitingConfirmation = false;
      }
    },
  },
  computed: {
    updateTime() {
      // return (this.coeff * 160 - (this.level * 70) / 100) * this.quantity * 1000;
      return (this.coeff * 200 - (this.coeff * 240 * this.level) / 100) * (this.quantity * 1000);
      // utils.calculateTimeToTrain(this.coeff, this.level, this.quantity);
    },
    steemAccount() {
      if (this.$store.state.auth.account) return this.$store.state.auth.account;
      return false;
    },
    pendingAmount() {
      if (this.$store.state.game.user.units.find(b => b.unit === this.id))
        return this.$store.state.game.user.units.find(b => b.unit === this.id).pending_amount || 0;
      return 0;
    },
    priceInSteem() {
      return ((this.price * this.quantity) / this.$store.state.game.prizeProps.steemprice).toFixed(
        3,
      );
    },
    priceInFuture() {
      return ((this.price / 0.005 - ((this.price / 100) * 20) / 0.005) * this.quantity).toFixed(0);
    },
    notEnoughFuture() {
      return (
        ((this.price / 0.005 - ((this.price / 100) * 20) / 0.005) * this.quantity).toFixed(3) >
        this.$store.state.game.user.user.future
      );
    },
    timeToWait() {
      const unit = this.$store.state.game.user.units.find(b => b.unit === this.id);
      if (unit) {
        if (unit.pending_update) {
          const nextUpdate = new Date(unit.pending_update).getTime();
          const now = this.$store.state.ui.timestamp;
          const timeToWait = nextUpdate - now;
          return timeToWait > 0 ? timeToWait : 0;
        }
        const nextUpdate = new Date(unit.next_update).getTime();
        const now = this.$store.state.ui.timestamp;
        const timeToWait = nextUpdate - now;
        return timeToWait > 0 ? timeToWait : 0;
      }
      return 0;
    },
  },
  methods: {
    ...mapActions(['recruitUnit', 'requestPayment']),
    handleSubmit(use) {
      this.isLoading = true;
      if (this.quantity > 0) {
        let payload = {};
        if (use === 'future')
          payload = { unit: this.id, unit_amount: Number(this.quantity), use: 'future' };
        else {
          payload = { unit: this.id, unit_amount: Number(this.quantity), use: 'resources' };
        }
        this.recruitUnit(payload)
          .then(() => {
            this.isLoading = false;
          })
          .catch(e => {
            console.error('Failed', e);
            this.isLoading = false;
          });
      }
    },
    handleRequestPayment() {
      this.requestPayment({
        memo: `unit:${this.id},amount:${this.quantity}`,
        amount: `${this.priceInSteem} STEEM`,
      });
    },
  },
};
</script>

<style scoped lang="less">
.checkout {
  text-align: center;
  width: 180px;
}

.futureicon {
  width: 22px;
  left: 0px;
  position: relative;
  float: left;
  top: 5px;
}
</style>
