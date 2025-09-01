<script setup lang="ts">
  import { withdrawRequest } from "@/api/wallet";
  import { parseUnits } from "viem";

  const { wallet, publicClient } = $(privyStore());
  const { signWithdraw, walletConfig, userBalance } = $(walletStore());
  const { getNonce } = $(authStore());

  // let toAddress = $ref("");
  let tokenAmount = $ref(1);
  let currentStep = $ref(1);
  let status = $ref("processing");
  let toAddress = $ref('');

  const waitTransaction = async (hash: string) => {
    try {
      const receipt = await publicClient.waitForTransactionReceipt({
        hash,
      });
      if (receipt && receipt.status === "success") {
        return receipt.transactionHash;
      } else {
        throw new Error("Transaction failed");
      }
    } catch (error) {
      console.log("waitTransaction error:", error);
      throw error;
    }
  };

  const newWithdrawal = () => {
    currentStep = 1;
  };

  const withdraw = async (form) => {
    const { toAddress } = form;
    currentStep = 2;
    // get a new nonce here
    const { data: nonce } = await getNonce(wallet.address);
    const amount_ = 10; // amount in USDC
    const signParams = {
      from: wallet.address,
      to: toAddress,
      amount: parseUnits(amount_.toString(), 6),
      tokenAddress: walletConfig!.main.address,
      nonce,
    };
    console.log(signParams, "signParams");
    const userSign = await signWithdraw(signParams);
    const requestParams = {
      toAddress,
      amount_,
      nonce,
      userSign,
    };
    const hash = withdrawRequest(requestParams);
    const tx = await waitTransaction(hash);
    console.log(tx);
  };
</script>

<template>
  <div>
    <div v-if="currentStep === 1" class="step-one w-full pb-3">
      <van-form @submit="withdraw">
        <van-field name="toAddress">
          <template #input>
            <BalanceForm v-model="toAddress" :maxlength="42" label="Recipient address" name="toAddress"
              placeholder="0x..." />
          </template>
        </van-field>
        <van-field name="tokenAmount">
          <template #input>
            <BalanceForm v-model="tokenAmount" label="Amount" name="tokenAmount" placeholder="0.00">
              <template #input-right>
                <div class="absolute right-1 top-1/2 transform -translate-y-1/2 flex items-center space-x-2">
                  <span class="text-gray-500 font-medium">USDC</span>
                  <button
                    class="px-3 bg-blue-50 text-blue-600 text-xs font-semibold rounded-md hover:bg-blue-100 transition-colors border border-blue-200"
                    type="button" @click="tokenAmount = userBalance">
                    MAX
                  </button>
                </div>
              </template>
              <template #input-tips>
                <div class="flex justify-between space-x-2">
                  <span></span>
                  <!-- <span class="text-gray-400 font-medium text-xs">${{ tokenAmount }}</span> -->
                  <span class="text-gray-400 text-xs ml-2">Balance:{{ userBalance }}
                  </span>
                </div>
              </template>
            </BalanceForm>
          </template>
        </van-field>
        <van-cell>
          <van-button class="rounded-lg" block type="primary" native-type="submit">
            {{ $t('withdraw') }}
          </van-button>
        </van-cell>
      </van-form>
    </div>
    <div v-if="currentStep === 2" class="step-two w-full">
      <van-cell-group>
        <van-cell>
          <van-count-down :time="time">
            <template #default="timeData">
              <span class="block">{{ timeData.hours }}</span>
              <span class="colon">:</span>
              <span class="block">{{ timeData.minutes }}</span>
              <span class="colon">:</span>
              <span class="block">{{ timeData.seconds }}</span>
            </template>
          </van-count-down>
        </van-cell>
        <van-cell title="Fill status" :value="status" />
        <van-cell title="You receive" value="1.0007" />
        <van-cell title="Transcation ID" value="0x123qsdq123sdad" />
        <van-notice-bar color="#a7a7a7" background="#f9f9f9" left-icon="info-o">
          <span class="font-xs">Experiencing problems?</span> <a>Get help</a>
        </van-notice-bar>
      </van-cell-group>
      <van-cell>
        <div class="flex mt-2 gap-2">
          <van-button block type="primary" plain native-type="submit">
            Close
          </van-button>
          <van-button block type="primary" native-type="submit" @click="newWithdrawal">
            New Withdrawal
          </van-button>
        </div>
      </van-cell>
    </div>
  </div>
</template>

<style>
  .colon {
    display: inline-block;
    margin: 0 4px;
    color: #1989fa;
  }

  .block {
    display: inline-block;
    width: 22px;
    color: #fff;
    font-size: 12px;
    text-align: center;
    background-color: #1989fa;
  }
</style>
