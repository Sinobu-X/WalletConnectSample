<script setup lang="ts">
import { useWallet } from '@tronweb3/tronwallet-adapter-vue-hooks';
import { ElButton, ElInput, ElOption, ElSelect } from 'element-plus';
import { tronWeb } from '../tronweb';
import { ref } from 'vue';
const { wallets, wallet, address, connected, select, connect, disconnect, signMessage, signTransaction } = useWallet();
console.log(wallets, wallet, address, connected);

const receiver = ref();
const approveTokenAddress = ref();
const approveSpenderAddress = ref();
const transferTokenAddress = ref();
const transferReceiveAddress = ref();


async function onSelect(v: any) {
    console.log('onselect', v);
    await select(v);
}

async function onConnect() {
    await connect();
}

async function onDisconnect() {
    await disconnect();
}

async function onSignMessage() {
    const res = await signMessage('Hello world!');
    alert(res);
}

async function onSignTransaction() {
    const transaction = await tronWeb.transactionBuilder.sendTrx(
        receiver.value,
        tronWeb.toSun(0.000001),
        wallet.value?.adapter.address
    );
    const signedTransaction = await signTransaction(transaction);
    // const signedTransaction = await tronWeb.trx.sign(transaction);
    const res = await tronWeb.trx.sendRawTransaction(signedTransaction);
    alert(JSON.stringify(res));
}

async function onSignTransactionForApproveTrc20() {
  // 构造合约调用交易
  const functionSelector = 'approve(address,uint256)';

  const parameter = [
    { type: 'address', value: approveSpenderAddress.value },
    { type: 'uint256', value: 1_000_000_000 }
  ];

  // 调用合约（这里只构造交易，不自动广播）
  const transaction = await tronWeb.transactionBuilder.triggerSmartContract(
      approveTokenAddress.value,
      functionSelector,
      {
        feeLimit: 100_000_000, // 最大消耗能量费用上限
        callValue: 0           // 调用 approve 不转账TRX
      },
      parameter,
      wallet.value?.adapter.address// 调用者地址
  );

  const signedTransaction = await signTransaction(transaction.transaction);
  const res = await tronWeb.trx.sendRawTransaction(signedTransaction);
  alert(JSON.stringify(res));
}

async function onSignTransactionForTransferTrc20() {
  // 构造合约调用交易
  const functionSelector = 'transfer(address,uint256)';

  const parameter = [
    { type: 'address', value: transferReceiveAddress.value },
    { type: 'uint256', value: 1_000_000 }
  ];

  // 调用合约（这里只构造交易，不自动广播）
  const transaction = await tronWeb.transactionBuilder.triggerSmartContract(
      transferTokenAddress.value,
      functionSelector,
      {
        feeLimit: 100_000_000, // 最大消耗能量费用上限
        callValue: 0           // 调用 approve 不转账TRX
      },
      parameter,
      wallet.value?.adapter.address// 调用者地址
  );

  const signedTransaction = await signTransaction(transaction.transaction);
  const res = await tronWeb.trx.sendRawTransaction(signedTransaction);
  alert(JSON.stringify(res));
}
</script>

<template>
    <p>Current Wallet: {{ wallet?.adapter.name }}</p>
    <p>Connection State: {{ wallet?.adapter.state }}</p>
    <p>Address : {{ address }}</p>
    <ElSelect :modelValue="wallet?.adapter.name" @change="onSelect">
        <ElOption v-for="wallet of wallets" :key="wallet.adapter.name" :value="wallet.adapter.name"></ElOption>
    </ElSelect>
    <ElButton :disabled="connected" @click="onConnect">connect</ElButton>
    <ElButton :disabled="!connected" @click="onDisconnect">disconnect</ElButton>
    <ElButton :disabled="!connected" @click="onSignMessage">signMessage</ElButton>
    <div>
        <ElInput v-model="receiver" placeholder="Input the receiver address" />
        <ElButton :disabled="!connected" @click="onSignTransaction">transfer</ElButton>
    </div>
  <div>
    <ElInput v-model="approveTokenAddress" placeholder="Input a trc20 token address" />
    <ElInput v-model="approveSpenderAddress" placeholder="Input a spender address" />
    <ElButton :disabled="!connected" @click="onSignTransactionForApproveTrc20">approve trc20</ElButton>
  </div>
  <div>
    <ElInput v-model="transferTokenAddress" placeholder="Input a trc20 token address" />
    <ElInput v-model="transferReceiveAddress" placeholder="Input a receive address" />
    <ElButton :disabled="!connected" @click="onSignTransactionForTransferTrc20">transfer trc20</ElButton>
  </div>
</template>

<style scoped>
p {
    color: #888;
}
</style>
