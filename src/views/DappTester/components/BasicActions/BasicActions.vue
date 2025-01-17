<template>
  <CustomCard title="Basic Actions">
    <div class="font-weight-bold">Network: {{ network }}</div>
    <div class="font-weight-bold">ChainId: {{ chainId }}</div>
    <div class="font-weight-bold">
      Accounts: {{ accounts.list.length > 0 ? accounts.list[0] : '' }}
    </div>
    <CustomBtn :disabled="button.disabled" @click="onClickConnect">
      {{ button.text }}
    </CustomBtn>
    <CustomBtn @click="getAccounts()"> eth_accounts </CustomBtn>
    <CustomTextbox title="eth_accounts result">{{
      accounts.result
    }}</CustomTextbox>
  </CustomCard>
</template>

<script setup lang="ts">
import CustomCard from '@/components/CustomCard/CustomCard.vue';
import CustomTextbox from '@/components/CustomTextbox/CustomTextbox.vue';
import CustomBtn from '@/components/CustomBtn/CustomBtn.vue';
import { onMounted, reactive, ref } from 'vue';
import type { TypeAccounts, TypeButton } from './types';

const accounts: TypeAccounts = reactive({
  list: [],
  result: ''
});
const button: TypeButton = reactive({
  disabled: false,
  text: 'Connect'
});
const chainId = ref<string>('');
const network = ref<string>('');
const ethereum = window.ethereum;

const emits = defineEmits<{
  (e: 'setAccounts', newAccounts: string[]): void;
  (e: 'setFromAccount', address: string): void;
  (e: 'setIsConnected', bool: boolean): void;
  (e: 'setChainId', chain: string): void;
  (e: 'setNeworkId', network: string): void;
}>();

onMounted(() => {
  initialize();
});

const isWeb3Connected = () => {
  return accounts.list && accounts.list.length > 0;
};
const onClickConnect = async () => {
  try {
    const newAccounts = await ethereum.request({
      method: 'eth_requestAccounts'
    });
    handleNewAccounts(newAccounts);
  } catch (error) {
    return error;
  }
};
const getAccounts = async () => {
  try {
    const _accounts = await ethereum.request({
      method: 'eth_accounts'
    });
    accounts.result = _accounts[0] || 'Not able to get accounts';
  } catch (err) {
    console.log(err);
    accounts.result = `Error: ${err}`;
  }
};

const handleNewAccounts = (newAccounts: string[]) => {
  accounts.list = newAccounts;
  if (isWeb3Connected()) {
    emits('setAccounts', newAccounts);
    emits('setFromAccount', newAccounts[0]);
    emits('setIsConnected', true);
  }
  updateButtons();
};

const handleNewChain = (chainID: string) => {
  chainId.value = chainID;
  emits('setChainId', chainId.value);
};

const handleNewNetwork = (networkID: string) => {
  network.value === networkID;
};

const getNetworkAndChainId = async () => {
  try {
    const chain = await ethereum.request({
      method: 'eth_chainId'
    });
    handleNewChain(chain);
    chainId.value = chain;
    emits('setChainId', chainId.value);
    const networkId = await ethereum.request({
      method: 'net_version'
    });
    handleNewNetwork(networkId);
    network.value = networkId;
    emits('setNeworkId', network.value.toString());
  } catch (err) {
    return err;
  }
};

const updateButtons = () => {
  if (isWeb3Connected()) {
    button.text = 'Connected';
    button.disabled = true;
  } else {
    button.text = 'Connect';
    button.disabled = false;
  }
};

const initialize = async () => {
  ethereum.autoRefreshOnNetworkChange = false;
  getNetworkAndChainId();

  ethereum.autoRefreshOnNetworkChange = false;
  getNetworkAndChainId();

  ethereum.on('chainChanged', (chain: string) => {
    handleNewChain(chain);
  });
  ethereum.on('chainChanged', handleNewNetwork);
  ethereum.on('accountsChanged', (newAccounts: string[]) => {
    handleNewAccounts(newAccounts);
  });
  try {
    const newAccounts = await ethereum.request({
      method: 'eth_accounts'
    });
    handleNewAccounts(newAccounts);
  } catch (err) {
    return err;
  }
};
</script>
