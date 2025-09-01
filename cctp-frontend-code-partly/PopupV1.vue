<script setup lang="ts">
  import { getNetworks } from '~/config/networks';
  const { modalIsShow } = $(uiStore());
  const { usdcBalance, loginAddress } = $(walletStore());

  const config = useRuntimeConfig()
  const testnet = config.public.testnet as boolean
  const chainName = getNetworks(testnet)[0]?.name
  const { copy, copied, text } = useClipboard()
  const selectedText = ref('USDC')
  const showPicker = ref(false)
  const selectedToken = ref(['USDC'])
  const showChainPicker = ref(false)

  const tokenColumns = [
    {
      text: 'USDC',
      img: '/icons/usdc.svg',
    },
  ]

  const chainColumns = [
    {
      text: chainName,
      img: '/icons/Avalanche.svg',
    },
  ]

  const closeModal = () => {
    modalIsShow.balanceModal = false;
  }
</script>

<template>
  <van-dialog closeable v-model:show="modalIsShow.balanceModal" :title="$t('Transfer Crypto')"
    :showConfirmButton="false" :showCancelButton="false" :z-index="50">
    <div class="text-sm px-4 pb-5">
      <div class="text-center text-gray-400 border-b-[1px] border-gray-100 pb-3 mb-3">
        <span>{{ $t('Balance') }}: ${{ usdcBalance }}</span>
      </div>
      <div class="font-semibold">{{ $t('Supported token') }}</div>
      <van-field class="items-center !px-0 after:!border-0" v-model="selectedText" :left-icon="tokenColumns[0]?.img"
        is-link readonly @click="showPicker = true" />
      <div class="font-semibold mt-2">{{ $t('Supported chain') }}</div>
      <van-field class="items-center !px-0 after:!border-0" v-model="chainName" :left-icon="chainColumns[0]?.img"
        is-link readonly @click="showChainPicker = true" />
      <div class="flex justify-between mt-3">
        <span class="font-semibold">{{ $t('Your deposit address') }}</span>
        <a class="!underline" href="https://turingm.io/terms" target="_blank">{{ $t('Terms apply') }}</a>
      </div>
      <div class="">
        <p
          class="break-words text-center px-3 py-2 border border-b-0 border-gray-200 rounded-lg rounded-b-none mt-1 text-gray-500">
          {{ loginAddress }}
        </p>
        <div
          class="flex justify-center items-center bg-[var(--van-button-primary-background)] text-white rounded-b-lg font-semibold px-3 py-2"
          @click="copy(loginAddress)">
          <span v-if="copied && text == loginAddress" class="w-4 text-green-500 ml-1">✔</span>
          <img v-else class="w-4 h-4 mr-1" src="/icons/copy-white.svg" />
          <span>{{ $t('Copy address') }}</span>
        </div>
      </div>
    </div>
  </van-dialog>
  <van-popup v-model:show="showPicker" position="bottom">
    <van-picker :columns="tokenColumns" :value-key="'text'" v-model="selectedToken" @confirm="showPicker = false"
      @cancel="showPicker = false">
    </van-picker>
  </van-popup>
  <van-popup v-model:show="showChainPicker" position="bottom">
    <van-picker :columns="chainColumns" :value-key="'text'" v-model="selectedChain" @confirm="showChainPicker = false"
      @cancel="showChainPicker = false">
    </van-picker>
  </van-popup>
</template>
