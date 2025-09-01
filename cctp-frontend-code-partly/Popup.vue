<script setup lang="ts">
  const { modalIsShow } = $(uiStore());
  const closeModal = () => {
    modalIsShow.balanceModal = false;
  }

  const {
    wallet,
    usdcBalance,
  } = $(walletStore());
</script>

<template>
  <van-dialog closeable v-model:show="modalIsShow.balanceModal" :showConfirmButton="false" :showCancelButton="false"
    :z-index="50">
    <div class="text-gray-500 font-xs mt-4 py-2 px-4 text-xs bg-gray-100 space-y-2 mt-10">
      <div class="flex justify-between">
        <div>{{ $t('Your Wallet Address:') }}</div>
        <div> {{ shortenAddress(wallet?.address) }}</div>
      </div>
      <div class="flex justify-between ">
        <div>{{ $t('Your Wallet Cash:') }}</div>
        <div>{{ usdcBalance }} USDC</div>
      </div>
    </div>
    <van-tabs>
      <van-tab :title="$t('Deposit')">
        <BalanceDeposit @close="closeModal" class="pt-4" />
      </van-tab>
      <van-tab :title="$t('Withdraw')">
        <BalanceWithdraw @close="closeModal" class="pt-4" />
      </van-tab>
    </van-tabs>
  </van-dialog>
</template>
