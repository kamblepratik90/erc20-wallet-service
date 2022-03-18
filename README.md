# ECC20-WALLET-SERVICE
Application to create address, transfer token, check balance and listing event transfer token and give notification to webhook and telegram if any event erc20 transfer

## Config
Please check [config/index.js](config/index.js) in that config, can set 
- runnin app port
- api key
- blockchain network
- wallet phasspharese
- token info that will be listened 
- webhook and telegream notification if any event erc20 tarnsfer 

## API
Please check [doc/rest.http](doc/rest.http), the api avaailable is
- create address  (when creating address keystore will save in folder [keystore](keystore))
- check balance
- info token
- send token

```
const config = require('../config');
const abi = require('../config/erc20abi');
const Web3 = require('web3');
const web3 = new Web3(new Web3.providers.WebsocketProvider(config.network.wssProvider));
const contractAddress = config.wallet.token.address;
const tokenContract = new web3.eth.Contract(abi,contractAddress);
const BigNumber = require('bignumber.js');
const notificationServices = require('./notification');
console.log(`listening event from ${config.network.wssProvider} with contract adddress ${contractAddress}`)
tokenContract.events.allEvents({fromBlock: 0, toBlock: "latest"}, function(error, event){
    console.log("allEvents");
    console.log(error);
    console.log("event : ",event);
    if(event.event=="Transfer"){
        let amount = event.returnValues._value;
        let decimals = config.wallet.token.decimal;
        let divisor = 10**decimals;
        divisor = new BigNumber(divisor);
        amount = new BigNumber(amount);
        let params={
            fromAddress:event.returnValues._from,
            toAddress:event.returnValues._to,
            amount:amount.dividedBy(divisor).toNumber()
        }
        notificationServices.sendNotificationTranfer(params);

    }
})
```
