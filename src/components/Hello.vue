<template>
  <div class="hello">

    <div>{{ code }}</div> 
    <div>{{ status }}</div>
    
    <div>
        <button type="button" v-on:click="createContract">Create Contract</button>
        <button type="button" v-on:click="getOneValue">getOneValue</button>
    </div>

    <div>
      <h2> current value: {{ currentValue }}</h2>
      <input type="number" v-model="param" v-on:keyup.enter="setOneValue"></input>
    </div>

  </div>
</template>

<script>
/* eslint-disable no-console, quotes, quote-props, no-useless-return */
import Web3 from 'web3';

const web3 = new Web3();
web3.setProvider(new web3.providers.HttpProvider('http://localhost:8545'));

const contractName = 'OneValue';
const source = `
pragma solidity ^0.4.4;
contract ${contractName} {
    uint256 value;
    function ${contractName}(uint256 initValue) {
        value = initValue;
    }

    function setValue(uint256 v) {
        value = v;
    }

    function getValue() constant returns (uint256 result) {
        return value;
    }
}`;

export default {
  name: 'hello',
  data() {
    return {
      showCreate: false,
      code: '',
      abi: [],
      myContract: '',
      status: '',
      param: 1234,
      currentValue: '',
    };
  },
  methods: {
    createContract() {
      // compiled contract & store code, abi
      const compiled = web3.eth.compile.solidity(source);
      this.code = compiled[`<stdin>:${contractName}`].code;
      this.abi = compiled[`<stdin>:${contractName}`].info.abiDefinition;
      this.status = 'transaction sent, waiting for confirmation';
      web3.eth.defaultAccount = web3.eth.coinbase;
      web3.personal.unlockAccount(web3.eth.coinbase, '123456');
      web3.eth.contract(this.abi).new({ data: this.code }, (err, contract) => {
        if (err) {
          console.log(err);
          return;
        // callback fires twice, we only want the second call when the contract is deployed
        } else if (contract.address) {
          this.myContract = contract;
          this.status = 'Mined!';
        }
      });
    },
    getOneValue() {
      // if you have publish your smart contract
      // you can use `const contract = web3.eth.contract(abi).at(contractAddress);`
      this.currentValue = this.myContract.getValue().toString(10);
    },
    setOneValue() {
      web3.personal.unlockAccount(web3.eth.coinbase, '123456');
      this.myContract.setValue(this.param, { from: web3.eth.coinbase });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
