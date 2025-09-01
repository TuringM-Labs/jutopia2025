<script setup lang="ts">
  import { showToast } from "vant";
  import { retrieveAttestation } from "@/api/cctp"
  import { encryptMiddle } from "@/utils/processing";
  import { getMessageTransmitter } from "@/config/networks"
  import { userMint } from "@/api/wallet";
  const { t } = useI18n()
  const {
    loginAddress,
    networks,
    selfBalance,
    connectWallet,
    switchNetwork,
    account,
    getSelfAllowance,
    getSelfBalance,
    approveUSDC,
    burnUSDC,
    updateWalletBalance
  } = $(walletStore());

  let { depositData } = $(depositStore());
  const { copy, copied, text } = useClipboard()

  let currentStep = $ref(1);
  let result = $ref("");
  let showChainPicker = $ref(false);
  let loading = $ref(false);
  let status = $ref("");
  let transactionTx = $ref("");

  let columns = networks.filter((item: any, index: number) => index !== 0).map((item: { name: string }) => ({
    text: item.name,
    value: item.name,
  }));
  let chain = $ref([columns[0]?.value]);

  const onConfirm = async ({
    selectedValues,
    selectedOptions,
  }: {
    selectedValues: string[];
    selectedOptions: Array<{ text: string; value: string }>;
  }) => {
    result = selectedOptions[0]?.text;
    depositData.chain = selectedValues;
    showChainPicker = false;
    await switchNetwork(result);
    getSelfBalance();
  };

  const deposit = async () => {
    console.log("depositData", depositData);
    loading = true;
    try {
      const allowance = await getSelfAllowance();
      console.log('allowance', allowance)
      if (allowance < depositData.tokenAmount) {
        const res = await approveUSDC(depositData.tokenAmount + 1); //depositData.tokenAmount
        if (res.error) {
          showToast(res.error);
          return;
        }
      }

      const { originDomain, transactionHash } = await burnUSDC(depositData.tokenAmount);

      if (!transactionHash) {
        showToast("Burn failed");
        return;
      }

      checkHash(originDomain, transactionHash)
      startCountdown()
      rateCountdown()
      currentStep = 2
      status = "Processing"
    } catch (error) {
      showToast("Deposit failed");
    } finally {
      loading = false;
    }
  };

  const checkHash = async (originDomain: number, transactionHash: string) => {
    const attestion = await retrieveAttestation(originDomain, transactionHash);
    if (attestion) {
      const messageTransmitter = getMessageTransmitter(account.chain)
      const res = await userMint({
        messageTransmitter: messageTransmitter,
        message: attestion.message,
        attestation: attestion.attestation
      })
      // const tx = await mintUSDC(attestion);
      if (res.code === 0) {
        status = "Successful";
        transactionTx = res.data;
        updateWalletBalance();
      } else {
        status = "Failed";
      }
      clearInterval(timer)
      clearInterval(rateTimer)
    }
  }

  const newWithdrawal = () => {
    currentStep = 1
    depositData.tokenAmount = 0
    timeLeft = duration
    rateLeft = duration
    getSelfBalance()
  };

  const duration = 60 // Countdown seconds
  let timeLeft = $ref(duration)
  let rateLeft = $ref(duration)
  let timer: any = null
  let rateTimer: any = null

  const rate = computed(() => ((duration - rateLeft) / duration) * 100)
  let percentage = $ref(0)

  const formattedTime = computed(() => {
    const m = String(Math.floor(timeLeft / 60)).padStart(2, "0")
    const s = String(timeLeft % 60).padStart(2, "0")
    return `${m}:${s}`
  })

  const startCountdown = () => {
    timer = setInterval(() => {
      if (timeLeft > 0) {
        timeLeft--
      } else {
        clearInterval(timer)
      }
    }, 1000)
  }

  const rateCountdown = () => {
    rateTimer = setInterval(() => {
      if (rateLeft > 0) {
        rateLeft -= 0.1
      } else {
        clearInterval(rateTimer)
      }
    }, 100)
  }

  watch(() => loginAddress, (newAddress) => {
    console.log('loginAddress', newAddress)
    if (newAddress) {
      depositData.depositToAddress = newAddress;
    }
  }, { immediate: true })

  watch([() => account.status, () => account.address, () => account.chain],
    ([newStatus, newAddress, newChain]) => {
      if (newStatus != 'connected') {
        depositData.depositFromAddress = '';
      } else {
        if (newAddress) {
          depositData.depositFromAddress = newAddress;
        }
        if (newChain) {
          result = newChain.name;
          depositData.chain = [...newChain.name];
        }
        getSelfBalance();
      }
    }, { immediate: true })
  onMounted(() => {
    getSelfBalance();
  })
</script>
<template>
  <van-cell-group>
    <div v-if="currentStep === 1" class="step-one w-full pb-3">
      <van-form>
        <van-field v-model="result" is-link readonly name="picker" :label="$t('From Chain')"
          :placeholder="$t('From Chain')" label-class="font-bold" input-align="right" @click="showChainPicker = true" />
        <van-popup v-model:show="showChainPicker" destroy-on-close position="bottom">
          <van-picker :columns="columns" v-model="chain" @confirm="onConfirm" @cancel="showChainPicker = false" />
        </van-popup>
        <van-field name="toAddress">
          <template #input>
            <BalanceForm v-model="depositData.depositFromAddress" :maxlength="42" :label="$t('From address')"
              :disabled="true" name="depositToAddress" placeholder="0x...">
              <template #input-right>
                <div class="absolute right-1 top-1/2 transform -translate-y-1/2 flex items-center space-x-2">
                  <button
                    class="px-3 bg-blue-50 text-blue-600 text-xs font-semibold rounded-md hover:bg-blue-100 transition-colors border border-blue-200"
                    type="button" @click="connectWallet">
                    {{ $t('Use connect') }}
                  </button>
                </div>
              </template>
            </BalanceForm>
          </template>
        </van-field>
        <van-field name="fromAddress">
          <template #input>
            <BalanceForm v-model="depositData.depositToAddress" :maxlength="42" :label="$t('Recipient address')"
              name="depositFromAddress" placeholder="0x..." />
          </template>
        </van-field>
        <van-field name="tokenAmount">
          <template #input>
            <BalanceForm v-model="depositData.tokenAmount" :label="$t('Amount')" name="tokenAmount" placeholder="0.00">
              <template #input-right>
                <div class="absolute right-1 top-1/2 transform -translate-y-1/2 flex items-center space-x-2">
                  <span class="text-gray-500 font-medium">USDC</span>
                  <button
                    class="px-3 bg-blue-50 text-blue-600 text-xs font-semibold rounded-md hover:bg-blue-100 transition-colors border border-blue-200"
                    type="button" @click="depositData.tokenAmount = selfBalance">
                    MAX
                  </button>
                </div>
              </template>
              <template #input-tips>
                <div class="flex justify-between text-xs text-gray-400 space-x-2">
                  <div class=" font-medium ">{{ chain[0] }}</div>
                  <div>
                    {{ $t('Balance') }}:{{ selfBalance }} USDC
                  </div>
                </div>
              </template>
            </BalanceForm>
          </template>
        </van-field>
        <van-cell>
          <van-button class="rounded-lg" block type="primary" native-type="submit" @click="deposit" :loading="loading">
            {{ $t('Deposit') }}
          </van-button>
        </van-cell>
      </van-form>
    </div>
    <div v-if="currentStep === 2" class="step-two w-full pb-3">
      <div class="flex justify-center my-2">
        <van-icon v-if="status == 'Successful'" name="checked" size="60" class="text-green-500" />
        <van-icon v-else-if="status == 'Failed'" name="clear" size="60" class="text-red-500" />
        <van-circle v-else v-model:current-rate="percentage" :rate="rate" :text="formattedTime" speed="10" size="60"
          class="my-4" />
      </div>
      <van-cell-group>
        <van-cell title="Fill status" :value="status"
          :value-class="{ '!text-green-500': status == 'Successful', '!text-red-500': status == 'Failed' }" />
        <van-cell title="You receive" :value="`≈ ${depositData.tokenAmount}`" />
        <van-cell title="Transcation ID">
          <template #value>
            <a class="text-primary" target="_blank">{{ encryptMiddle(transactionTx) }}</a>
          </template>
          <template #right-icon>
            <span v-if="copied && text == transactionTx" class="w-4 text-green-500 ml-1">✔</span>
            <img v-else class="w-4 h-4 ml-1 mt-1" src="/icons/copy.svg" @click="copy(transactionTx)" />
          </template>
        </van-cell>
        <van-notice-bar color="#a7a7a7" background="#f9f9f9" left-icon="info-o">
          <span class="font-xs">Experiencing problems?</span> <a class="text-black !underline" href=""
            target="_blank">Get
            help</a>
        </van-notice-bar>
      </van-cell-group>
      <van-cell-group>
        <div class="flex mt-2 px-4 gap-3">
          <van-button block type="primary" plain native-type="submit" @click="newWithdrawal(); $emit('close')">
            Close
          </van-button>
          <van-button block type="primary" native-type="submit" @click="newWithdrawal">
            New Withdrawal
          </van-button>
        </div>
      </van-cell-group>
    </div>
  </van-cell-group>
</template>
