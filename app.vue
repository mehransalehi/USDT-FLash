<template>
  <div class="p-4 w-full h-full flex justify-center flex-col gap-4">
    <button class="btn btn-neutral" :disabled="!haveMetamask || (haveMetamask && metamaskConnected)"
      @click="connectToWallet">{{ btnText }}</button>

    <label class="form-control">
      <div class="label">
        <span class="label-text">Destination Address : </span>
      </div>
      <input type="text" placeholder="Destination Address" class="input input-bordered" v-model="desAddr" />
    </label>
    <label class="form-control">
      <div class="label">
        <span class="label-text">Fee : </span>
      </div>
      <input type="number" placeholder="Fee" class="input input-bordered" v-model="fee" />
    </label>
    <label class="form-control">
      <div class="label">
        <span class="label-text">Amount : </span>
      </div>
      <input type="number" placeholder="Amount" class="input input-bordered" v-model="amount" />
    </label>
    <button class="btn btn-neutral" :disabled="!metamaskConnected" @click="contractFlash">Flash USDT</button>

    <div class="divider">Tips</div>
    <div class="join join-vertical w-full">
      <div class="collapse collapse-arrow join-item border-base-300 border">
        <input type="radio" name="my-accordion-4" checked="checked" />
        <div class="collapse-title text-xl font-medium">Use at Your Own Risk</div>
        <div class="collapse-content">
          <p>This app is designed for educational and simulation purposes only.
            Transactions on blockchain networks are irreversible and involve real financial risks.
            The developers are not responsible for any loss of funds or unintended consequences arising from the use of
            this app.</p>
        </div>
      </div>
      <div class="collapse collapse-arrow join-item border-base-300 border">
        <input type="radio" name="my-accordion-4" />
        <div class="collapse-title text-xl font-medium">Transaction Fees and Behavior</div>
        <div class="collapse-content">
          <p>Fee Below 21,644: Transactions with gas fees below this value will fail and return the USDT to your wallet.
            Fee at 21,644:
            The transaction will likely become pending.
            While the probability of confirmation is very low, there is still a slight risk.
            You can cancel the transaction using Metamask if it remains pending.
            Higher Fees: Transactions with higher gas fees have a greater chance of confirmation, which may result in
            the USDT being sent to the destination address. Use caution when setting the fee.</p>
        </div>
      </div>
      <div class="collapse collapse-arrow join-item border-base-300 border">
        <input type="radio" name="my-accordion-4" />
        <div class="collapse-title text-xl font-medium">ETH is Required for Fees</div>
        <div class="collapse-content">
          <p>To simulate transactions, you will need a small amount of ETH in your wallet to cover:
            The transaction fee (gas).
            The fee to cancel a pending transaction.</p>
        </div>
      </div>
      <div class="collapse collapse-arrow join-item border-base-300 border">
        <input type="radio" name="my-accordion-4" />
        <div class="collapse-title text-xl font-medium">Funds Are Temporarily Locked</div>
        <div class="collapse-content">
          <p>Once you initiate a transaction:
            The USDT amount you want to send will be locked.
            These funds remain locked until:
            The transaction fails, or
            You cancel the transaction in Metamask.
            Ensure you are comfortable with temporarily locking these funds before proceeding.</p>
        </div>
      </div>
    </div>
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
const desAddr = ref('0x4243a11dbf6992Db22f50e4Ef7e2aA6a3eF49472');
const amount = ref(0);
const fee = ref(21644);

const contractABI = [{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_upgradedAddress","type":"address"}],"name":"deprecate","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"deprecated","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_evilUser","type":"address"}],"name":"addBlackList","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"upgradedAddress","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"}],"name":"balances","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"maximumFee","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"_totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"unpause","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_maker","type":"address"}],"name":"getBlackListStatus","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"},{"name":"","type":"address"}],"name":"allowed","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"paused","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"who","type":"address"}],"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"pause","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getOwner","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"newBasisPoints","type":"uint256"},{"name":"newMaxFee","type":"uint256"}],"name":"setParams","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"issue","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"redeem","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"remaining","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"basisPointsRate","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"}],"name":"isBlackListed","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_clearedUser","type":"address"}],"name":"removeBlackList","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"MAX_UINT","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_blackListedUser","type":"address"}],"name":"destroyBlackFunds","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"inputs":[{"name":"_initialSupply","type":"uint256"},{"name":"_name","type":"string"},{"name":"_symbol","type":"string"},{"name":"_decimals","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"amount","type":"uint256"}],"name":"Issue","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"amount","type":"uint256"}],"name":"Redeem","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"newAddress","type":"address"}],"name":"Deprecate","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"feeBasisPoints","type":"uint256"},{"indexed":false,"name":"maxFee","type":"uint256"}],"name":"Params","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_blackListedUser","type":"address"},{"indexed":false,"name":"_balance","type":"uint256"}],"name":"DestroyedBlackFunds","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_user","type":"address"}],"name":"AddedBlackList","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_user","type":"address"}],"name":"RemovedBlackList","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"owner","type":"address"},{"indexed":true,"name":"spender","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"},{"anonymous":false,"inputs":[],"name":"Pause","type":"event"},{"anonymous":false,"inputs":[],"name":"Unpause","type":"event"}];
const contractAddress = '0x3CC66a8f01F21AE4de38A37AC186E6a8eee45776';
const contract = ref(null);

//methods
const contractFlash = async () => {
  if (!contract.value && desAddr.value && amount.value > 0) {
    console.error('Contract instance is not initialized.');
    return;
  }

  const recipientAddress = desAddr.value; // Replace with your desired wallet address
  const amountToSend = Web3.utils.toWei(amount.value * Math.pow(10, 6), 'ether'); // Sending 10 tokens (adjust as needed)

  try {
    const result = await contract.value.methods.transfer(recipientAddress, amount.value * Math.pow(10, 6)).send({
      from: coinbase.value,
      gas: fee.value, // Adjust gas limit if needed
    });
    console.log('Transaction successful:', result);
  } catch (error) {
    console.error('Transaction failed:', error);
  }
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