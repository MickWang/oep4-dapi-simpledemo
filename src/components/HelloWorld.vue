<style scoped>
h1, h2 {
  font-weight: normal;
}

label {
  color: black;
  font-weight: bold;
}
.content {
  margin:15px;
}
.address-input {
  width:400px;
}
p {
  margin-bottom:5px;
}

.transfer-content {
  border:1px solid #dddddd;
  padding:10px;
  width:550px;
  margin:10px auto;
}
.transfer-content div {
  margin-bottom:10px;
}
.transfer-content p {
  font-size:20px;
}
</style>
<template>
  <div class="hello">
    <h1>OEP-4 Demo</h1>
    <div class="content">
      <label>OEP-4 Contract</label>
      <a-input type="text" class="address-input" v-model="contractHash" placeholder="OEP4 contract hash" />
      <a-button type="primary" @click="setOep4Contract()">OK</a-button>
    </div>
    <div class="content">
      <!-- <a-button @click="init()">Init OEP4</a-button>
      <a-button @click="queryTotalSupply()">Query Total Supply</a-button>
      <a-button @click="queryDecimal()">Query Decimal</a-button>
      <a-button @click="queryName()">Query Name</a-button> -->
      <p><label for="">OEP4 Name: </label> {{oep4Name}}</p>
      <p><label for="">Symbol: </label> {{symbol}}</p>
      <p><label for="">Decimal:</label>  {{decimal}}</p>
      <p><label for="">Total supply:  </label> {{totalSupply}}</p>      
    </div>
    
    <div class="content">
      <label >Query Balance</label>
      <a-input type="text" class="address-input" v-model="address" placeholder="Please enter adderss" />
      <a-button type="primary" @click="queryBalance()" >OK</a-button>
    </div>

    <div class="transfer-content">
      <p>Transfer OEP4</p>
      <div>
        <label for="">Sender</label>
        <a-input :value="account"  class="address-input" />
      </div>
      <div>
        <label for="">Receiver</label>
        <a-input v-model="receiver"  class="address-input" />
      </div>
      <div>
        <label for="">Amount</label>
        <a-input v-model="amount"  class="address-input" />
      </div>
      <a-button type="primary" @click="transfer()">Submit</a-button>
    </div>
  </div>
</template>

<script>
import {client} from 'ontology-dapi'
import {RestClient, Crypto, utils, Parameter, ParameterType} from 'ontology-ts-sdk'
const gasPrice = '500';
const gasLimit = '20000';
export default {
  name: 'HelloWorld',
  data () {
    return {
      oep4: null,
      contractHash: '',
      contractAddr: '',
      account: null,
      address: '',
      receiver: 'AQyDzhRBQr1trEbvfDcLpEzxff7dtFcyRc',
      amount: 0,

      oep4Name: '',
      symbol: '',
      totalSupply: 0,
      decimal: ''
    }
  },
  async mounted() {
    // get provider
      try { 
        const provider = await client.api.provider.getProvider();
        console.log('onGetProvider: ' + JSON.stringify(provider));
        this.provider = provider;
      } catch (e) {
        console.log('No dAPI provider istalled.');
        this.$message.warning('No provider installed. Please install the "Cyano Wallet" ')
        return null;
      }
      // get account
      let account;
      try { 
        account = await client.api.asset.getAccount();
        this.account = account;
        console.log(account);
      } catch (err) {
        console.log(err)
        this.$message.warning('No account found in the provider. Please prepare an account')
        return 'NO_ACCOUNT';
      }
  },
  methods: {
    async setOep4Contract() {
      if(this.contractHash){
        const restClient = new RestClient();// Default connect to Testnet
        const res = await restClient.getContract(this.contractHash);
        console.log(JSON.stringify(res))
        if(!res.Result) {
          this.$message.error('The contract has not been deployed yet.')
          return;
        }
        const address = new Crypto.Address(utils.reverseHex(this.contractHash))
        this.contractAddr = address;
        const params1 = {
          contract: this.contractHash,
          method: 'Name',
          parameters: []
        }
        let name = await this.scInvoke(params1, true); 
        if(name) {
          name = utils.hexstr2str(name);
        } 
        this.oep4Name = name;
        const params2 = {
          contract: this.contractHash,
          method: 'TotalSupply',
          parameters: []
        }
        let totalSupply = await this.scInvoke(params2, true);
        if(totalSupply) {
          totalSupply = parseInt(utils.reverseHex(totalSupply), 16)
        }
        this.totalSupply = totalSupply;
        const params3 = {
          contract: this.contractHash,
          method: 'Decimal',
          parameters: []
        }
        let decimal = await this.scInvoke(params3, true);
        if(decimal) {
          decimal = parseInt(utils.reverseHex(decimal), 16)
        }
        this.decimal = decimal;
        const params4 = {
          contract: this.contractHash,
          method: 'Symbol',
          parameters: []
        }
        let symbol = await this.scInvoke(params1, true);
        symbol = utils.hexstr2str(symbol)
        this.symbol = symbol;
        this.$message.success('The contract has been deployed successfully!')

      }
    },
    async init() {
      const contract = this.contractHash;
      const method = 'Init';
      const parameters = [];
      const params = {
        contract,
        method,
        parameters,
        gasPrice,
        gasLimit
      }
      const res = await this.scInvoke(params, false);
      if(res) {
        this.$message.success('Init contract hash success!')
      }
    },

    async queryTotalSupply() {
      const contract = this.contractHash;
      const method = 'TotalSupply';
      const parameters = [];
      const params = {
        contract,
        method,
        parameters,
        gasPrice,
        gasLimit
      }
      const res = await this.scInvoke(params, true);
      if(res) {
        const val = parseInt(utils.reverseHex(res), 16);
        this.$message.success('Total supply: ' + val);
      }
    },

    async queryDecimal() {
      const contract = this.contractHash;
      const method = 'Decimal';
      const parameters = [];
      const params = {
        contract,
        method,
        parameters,
        gasPrice,
        gasLimit
      }
      const res = await this.scInvoke(params, true);
      if(res) {
        const val = parseInt(utils.reverseHex(res), 16);
        this.$message.success('Decimal: ' + val);
      }
    },

    async queryName() {
      const contract = this.contractHash;
      const method = 'Name';
      const parameters = [];
      const params = {
        contract,
        method,
        parameters,
        gasPrice,
        gasLimit
      }
      const res = await this.scInvoke(params, true);
      if(res) {
        const val = utils.hexstr2str(res);
        this.$message.success('Name: ' + val);
      }
    },

    async queryBalance() {
      const contract = this.contractHash;
      const method = 'BalanceOf';
      const address = new Crypto.Address(this.address);
      const parameters = [
        new Parameter('account', ParameterType.ByteArray, address.serialize())
      ];
      const params = {
        contract,
        method,
        parameters,
        gasPrice,
        gasLimit
      }
      const res = await this.scInvoke(params, true);
      if(res) {
        const val = parseInt(utils.reverseHex(res), 16);
        this.$message.success('Balance: ' + val);
      } else {
        this.$message.success('Balance: ' + 0);
      }
    },

    async transfer() {
      const contract = this.contractHash;
      const method = 'Transfer';
      const from = new Crypto.Address(this.account);
      const to = new Crypto.Address(this.receiver);
      const amount = parseInt(this.amount);
      const parameters = [
        new Parameter('from', ParameterType.ByteArray, from.serialize()),
        new Parameter('to', ParameterType.ByteArray, to.serialize()),
        new Parameter('amount', ParameterType.Integer, amount)          
      ];
      const params = {
        contract,
        method,
        parameters,
        gasPrice,
        gasLimit
      }
      const res = await this.scInvoke(params, false);
      if(res) {
        this.$message.success('Transfer succeed!');
      }
    },

    async scInvoke(params, preExec) {
      if(!this.contractAddr) {
        this.$message.error('The contract has not been deployed yet.')
        return;
      }
      try {
        let result;
        if(preExec) {
           result = await client.api.smartContract.invokeRead(params);       
        } else {
           result = await client.api.smartContract.invoke(params);
        }
        console.log('onScCall finished, result:' + JSON.stringify(result));        
        return result;
      } catch (e) {
        console.log('onScCall error:', e);
        this.$message.error('Some error happens. Please try later.')
        return null;
      }
    },
  }
}
</script>


