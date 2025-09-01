<script setup lang="ts">
  const { usdcBalance } = $(walletStore());
  const { setModal } = $(uiStore());
  const { token } = $(authStore());
  const { userInfo } = $(userStore())
  const avatar = computed(() => {
    return userInfo?.avatar || '/icons/usdc.svg'
  })
</script>

<template>
  <div>
    <button v-if="token?.accessToken" class="px-3 py-1 bg-gray-500/20 rounded-xl transition-colors"
      @click="setModal('balanceModal', true)">
      <div class="flex items-center">
        <img class="w-5 h-5 rounded-full mr-2" :src="avatar" alt="" />
        <span class="text-white/80">${{ Math.floor(usdcBalance * 100) / 100 }}</span>
      </div>
    </button>
    <van-button size="small" v-else round type="primary" class="px-3 text-white/80 right mr-2!"
      @click="setModal('loginModal', true)">{{ $t("Login") }}</van-button>
  </div>
</template>

<style></style>
