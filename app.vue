<template>
  <div class="p-4 w-full h-full flex justify-center flex-col gap-4">
    <button class="btn btn-neutral" :disabled="!haveMetamask || (haveMetamask && metamaskConnected)"
      @click="connectToWallet">{{ btnText }}</button>
    <button class="btn btn-neutral" :disabled="!metamaskConnected" @click="contractFlash">Flash USDT With
      Contract</button>
  </div>
</template>

<script setup lang="js">
import { onMounted } from 'vue'
import { Web3 } from 'web3'


const haveMetamask = ref(false);
const metamaskConnected = ref(false);
const btnText = ref('Metamask Not Installed');
const networkId = ref(null);
const coinbase = ref(null);
const balance = ref(null);

const contractABI = [{ "inputs": [], "stateMutability": "nonpayable", "type": "constructor" }, { "anonymous": false, "inputs": [{ "indexed": false, "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "SetValue", "type": "event" }, { "inputs": [{ "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "setValue", "outputs": [], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [], "name": "value", "outputs": [{ "internalType": "uint256", "name": "", "type": "uint256" }], "stateMutability": "view", "type": "function" }];
const contractAddress = '0x02417422227864Fa58D39f2595f206E067a128F5';
const contract = ref(null);
//methods
const contractFlash = async () => {
  const result = await contract.value.methods.setValue(10).send({
    from: coinbase.value,
    gas: 3000000, // Optional: Specify gas limit
  });
  console.log(result);
}
const connectToWallet = async () => {
  if (haveMetamask.value) {
    await window.ethereum.send('eth_requestAccounts');
    const instance = new Web3(window.ethereum);
    // Get necessary info on your node
    networkId.value = await instance.eth.net.getId();
    coinbase.value = await instance.eth.getCoinbase();
    balance.value = await instance.eth.getBalance(coinbase.value)
    console.log(networkId.value, coinbase.value, balance.value);

    btnText.value = 'Connected To ' + coinbase.value;
    metamaskConnected.value = true;

    //create the contract instanse
    contract.value = new instance.eth.Contract(contractABI, contractAddress);
  }
}
onMounted(() => {
  console.log("Mounted");
  if (window.ethereum) {
    haveMetamask.value = true;
    btnText.value = 'Connect To Metamask'
  }
});
</script>