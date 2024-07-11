# Stellar Snap Documentation

## Stellar on Metamask
The metastellar.io team manages and maintains the stellar wallet plugin for metamask. If implemented correctly, <b>the end user should be aware they are using the stellar chain, but the experence should never
feel like they are using a 'plug-in'</b> hince the term <b>snap</b>.

The <b>metastellar snap</b> is a piece of code that lives inside the metamask wallet, and is automatically installed when requested by a web app.
Connecting to the Stellar network using the snap is covered in <a href="https://metastellar.io/docs/#/?id=%e2%9c%a8connect-and-install">✨connect and install</a> portion of the docs.

After the user installs the snap, a stellar wallet automatically created for them.
This wallet can be accessed, using the standard metamask rpc api. This means that if you have experence developing with metamask in ethereum this shouldn't be too different. (sadly, no ~~web3.js~~ stellar3.js yet 🤞).

As a developer basic idea, is you shouldn't have to focus on OUR wallet, you should focus on YOUR app.
Ideally the flow would be.

[connect Metamask] -> [<a href="https://github.com/stellar/js-stellar-sdk">create Stellar TXN</a>] -> [call <a href="https://metastellar.io/docs/#/?id=_39signtransaction39">signTxn</a>] -> [<a href="https://github.com/stellar/js-stellar-sdk">submit signed txn</a>] ✅ 

#### happy hacking

<span class="spacer"></span>
<hr>

# Quick Start
<span class="spacer"></span>
  <ol>
  <li>There is <b>NO npm package required!</b></li>
  <li>The only thing required is that the users computer has metamask flask<br/>(just normal metamask after launch)</li>
  <li><a href="https://docs.metamask.io/snaps/get-started/install-flask/">install flask</a></li>
  </ol>
<span class="spacer"></span>

  ## ✨ Connect and install
  The <b>wallet_requestSnaps</b> method is used to <b>connect</b> to MetaMask <b>and installs</b> the Stellar Wallet if it's not already installed. This also generates the user's wallet.
  ```typescript
  
  /* //request connection */
  async function connect(){
    const connected = await ethereum.request({
      method: 'wallet_requestSnaps',
      params: {
        [`npm:stellar-snap`]: {}
      },
    });
  }
  
  ```
<button id="connectButton">exec connect()</button>

<br>

<span class="spacer"></span>

  ## ✨ Calling Stellar Methods
  After the snap is connected the <b>wallet_invokeSnap</b> method is used to call Stellar Methods

  ### 🟠 get wallet address
```typescript 
  
      //evoke a stellar method
      
      const request = {
          method: 'wallet_invokeSnap',
          params: {snapId:`npm:stellar-snap`, 
            request:{
              method: `${'Stellar-Method-Name'}`
            }
          }
      }
      let address = await ethereum.request(request)
  
  
      // gets the stellar address
      address = await ethereum.request({
          method: 'wallet_invokeSnap',
          params: {snapId:`npm:stellar-snap`, request:{
              method: `getAddress`,
          }}
      })
    
  ```
  
  <button id="execAddressButton">get the users Address!</button>
  <span class="spacer"></span>

  ### 🌐 Specify a Network

  by default all methods are treated as mainnet, but any method can be issued to the testnet
  by using the testnet param. The testnet parameter can be used with all methods. If it dosn't make
  since for a certain method it is simply ignored silently.

  example:
  ```typescript
      const result = await ethereum.request({
          method: 'wallet_invokeSnap',
          params: {snapId:`npm:stellar-snap`, request:{
              method: `getBalance`,
              params:{
                testnet: true
              }
          }}
      })
  ```

  ### 🔏 Sign a Transaction
    
  <b>Parameters are nested,</b> parameters inside parameters
  
  ```typescript 
      //evoke a stellar method with arguments
      let stellarTransactionXDR = endTransaction.build().toXDR(); //transaction from the stellar-js-sdk
      const args = {
        transaction: String(stellarTransactionXDR),
        network:'testnet'
      }
      const request = { 
          method: 'wallet_invokeSnap', //constant across all method calls
          params:{snapId:'npm:stellar-snap', request:{  //this too
            method:`${'signTransaction'}`,
            params:args
          }
          }
      }
      let SignedTransactionXDR = await ethereum.request(request)
      
      // example method call with parameters
      SignedTransactionXDR = await ethereum.request({
          method: 'wallet_invokeSnap',
          params: {snapId:`npm:stellar-snap`, request:{
              method: `signTransaction`,
              params:{
                transaction: stellarTransactionXDR
                testnet:true
              }

          }}
      })
  ```

<span class="spacer"></span>

<script>
  let connectButton = document.getElementById("connectButton");
  let connected = false;
  let testnet = true;
  let testnetFunded = true;
  console.log(connectButton)

  async function connectSnap(){
    try{
      console.log("here")
      connected = await callMetaStellar('connect');
      
      
      await fund();
      await alert("💸testnet account funded💸");
      console.log(connected)
      
    }catch(e){
      if (e.toString() === "ReferenceError: ethereum is not defined"){
         alert("Install metamask flask")
      }
      alert(e);
    }
  }
  connectButton.addEventListener('click', async ()=>await connectSnap());

  async function callMetaStellar(method, params){
    if(method === 'connect'){
        return await ethereum.request({
          method: 'wallet_requestSnaps',
          params: {
            ['npm:stellar-snap']: {}
          },
        });
    }
    if(params === undefined){
      params = {}
    }
    const rpcPacket = {
      method: 'wallet_invokeSnap',
      params:{
        snapId:'npm:stellar-snap',
        request: {'method':method, params:params}
      }
    }
    console.log("in call Metastellar here");
    return await ethereum.request(rpcPacket);
  }

  function alertObject(obj){
    console.log(obj);
    function stringObj(obj){
      let outputString = '{';
      let keys = Object.keys(obj);
      for(let i = 0; i<keys.length; i++){
        if(typeof obj[keys[i]] === 'object'){
          obj[keys[i]] = JSON.stringify(obj[keys[i]]);
        }
        outputString+=`${keys[i]} : ${obj[keys[i]]},\n`
      }
      outputString+='}';
      return outputString;
    }

    alert(stringObj(obj));
  }

  async function fund(){
      testnetFunded = await callMetaStellar('fund');
      return testnetFunded;
  }

  const getAddress = async function(){
    console.log("here2")
    try{
        console.log("about to run request");
        const request = {
            method: 'wallet_invokeSnap',
            params: 
            
              {
                snapId:'npm:stellar-snap', 
                request:{
                  method: `${'getAddress'}`
                }
              }
            
        }
        console.log("request in memory")
        let address = await ethereum.request(request);
        console.log("request complete");
        console.log(address)
        // gets the stellar address
        address = await ethereum.request({
            method: 'wallet_invokeSnap',
            params: 
              {
                snapId:'npm:stellar-snap', 
                request:{
                    method: 'getAddress',
                }
              }
            
        });
        alert(address);
    }
    catch(e){
      console.log("error");
      console.log(e);
      alert(e);
    }
  }
  let execButton = window.document.getElementById("execAddressButton");
  console.log(execButton);
  execButton.addEventListener('click', getAddress);
  



  //methods 

  //'getAccountInfo'

  //fund

  //displayAddress
  //importAccount
  //exportAccount
  //transfer
  let sendXLMButton = document.getElementById('sendXLMButton');
  async function sendXLM(){
    
    console.log("connected is");
    console.log(connected);
    if(!connected){
      connected = await callMetaStellar('connect');
    }
    if(!testnetFunded){
      testnetFunded = await callMetaStellar('fund', {testnet:true})
    }
    const recipentAddress = await prompt("to (stellar Address): ");
    let balance = await callMetaStellar('getBalance', {testnet:true})
    console.log(`balance: ${balance}`);
    let amount = NaN;
    while(true){
      amount = Number(await prompt(`amount of xlm to send?\nyou have ${balance}`));
      console.log(amount);
      if(amount){
        break
      }
      if(amount === 0){
        return
      }
      else{
        alert(`${amount} is not a number`);
      }
    }
    amount = String(amount);
    
    const result = await callMetaStellar('transfer', {to:recipentAddress, amount:amount, testnet:true});
    console.log(result);

  }
  sendXLMButton.addEventListener('click', sendXLM);

  //listAccounts
  //showAddress
  let showAddressButton = document.getElementById('showAddressbtn')
  showAddressButton.addEventListener('click', ()=>callMetaStellar('showAddress'))


  let getDataPacketbtn = document.getElementById('getDataPacketbtn');

  async function getDataPacket(){
    let dataPacket = await callMetaStellar('getDataPacket');
    console.log(dataPacket);
    alertObject(dataPacket);
  }
  getDataPacketbtn.addEventListener('click', getDataPacket);

  let getAccountInfobtn = document.getElementById('getAccountInfobtn');
  async function getAccountInfo(){
   let result = await callMetaStellar('getAccountInfo', {testnet:true});
   alertObject(result);
  }
  getAccountInfobtn.addEventListener('click', getAccountInfo);
  

  const getBalancebtn = document.getElementById('getBalanceButton');
  getBalancebtn.addEventListener('click', async ()=>alert(`${await callMetaStellar('getBalance', {testnet:true})} testnet xlm`))

  const createAccountButton = document.getElementById('createAccountButton');
  async function createAccount(){
    let name = await prompt("account name");
    result = await callMetaStellar('createAccount', {name:name});
    console.log(result);
    alert(result);
  }
  
  createAccountButton.addEventListener('click', )
</script>

# Stellar RPC Methods

## 📎  example method call
```typescript 
    const result = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, request:{
            method: `${methodName}`,
            params:{
              paramName: `${paramValue}`
            }
        }}
    })
```


## 'getAddress'
returns the accounts address as a string
```typescript
    const address = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {snapId:`npm:stellar-snap`, request:{
            method: `getAddress`,
        }}
    })
```

## 'getAccountInfo'
grabs infomation related to the account
requires account to be funded
```typescript
    const info = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {snapId:`npm:stellar-snap`, request:{
            method: `getAccountInfo`,
            params:{
                testnet?: true | false
            }
        }}
    })
```

<button id="getAccountInfobtn">getAccountInfo</button>


## 'getBalance'
gets the XLM balance of a wallet, returns 0 in unfunded wallets

```typescript
    const balance = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {snapId:`npm:stellar-snap`, request:{
            method: `getBalance`,
            params:{
                testnet?: true | false
            }
        }}
    })
```

<button id="getBalanceButton">getBalance</button>

## 'transfer'
this method is used to transfer xlm and requires a funded account.
after being called the wallet will generate a transaction, then prompt a user to accept
if the user accepts the transaction it will be signed and broadcast to the network.
will return transaction infomation. And send a notification stating whether the transaction was
successful.

returns: StellarSDK.TransactionResult
```typescript
const transactionInfomation = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {snapId:`npm:stellar-snap`, request:{
            method: `getBalance`,
            params:{
                to: 'stellarAddress' //string
                amount: '1000.45' //string represention of amount xlm to send
                testnet?: true | false
            }
        }}
    })

```

<button id="sendXLMButton">send xlm</button>

## 'fund'
this method funds the users wallet on the testnet,
```typescript
const success = await ethereum.request({
    method: 'wallet_invokeSnap',
    params: {snapId:`npm:stellar-snap`, 
        request:{
            method: 'fund'
        }
    }
    })
```

## 'signTransaction'
This method signs an Arbitary Transaction
```typescript
    async function signTransaction(){
      const transaction = new StellarSdk.TransactionBuilder(account, { fee, networkPassphrase: "Test SDF Network ; September 2015" });
      // Add a payment operation to the transaction
      console.log("transaction builder initilazed");
      await transaction.addOperation(StellarSdk.Operation.payment({
        destination: receiverPublicKey,
        // The term native asset refers to lumens
        asset: StellarSdk.Asset.native(),
        // Specify 350.1234567 lumens. Lumens are divisible to seven digits past
        // the decimal. They are represented in JS Stellar SDK in string format
        // to avoid errors from the use of the JavaScript Number data structure.
        amount: '350.1234567',
      }));
      console.log("operations added")
      // Make this transaction valid for the next 30 seconds only
      await transaction.setTimeout(30);
      console.log("timeout set");
      // Uncomment to add a memo (https://www.stellar.org/developers/learn/concepts/transactions.html)
      // .addMemo(StellarSdk.Memo.text('Hello world!'))
      const endTransaction = await transaction.build();
      const xdrTransaction = endTransaction.toXDR();
      console.log(xdrTransaction);
      const response = await ethereum.request({
        method: 'wallet_invokeSnap',
        params:{snapId:snapId, request:{
          method: 'signTransaction',
          params:{
            transaction: xdrTransaction,
            testnet: testnet
          }
        }}
      })
      console.log(response);
    }
```
## 'getDataPacket'

retreves wallet info about the user, including names, addressess, and balances

returns <a href="/#/?id=datapacket">DataPacket</a>

```typescript
    const walletInfo: DataPacket = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, request:{
            method: `getDataPacket`,
        }}
    })
```
<button id="getDataPacketbtn">get data packet</button>

## setCurrentAccount

changes the connected account

```typescript
  interface setCurrentAccountParams :{ 
    address:string
  }
  const switchAccountParams:setCurrentAccountParams = {
    address:`${WalletAddress}`
  }
  const result = await ethereum.request({
    method: 'wallet_invokeSnap',
    params: {`npm:stellar-snap`, 
      request:{
        method: switchAccountParams,
        params
      }
    }
  })
```

## showAddress

displays the stellar address and a qr code in the extension

returns: boolean
```typescript
    const result = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, 
          request:{
              method: `showAddress`,
          }
        }
    })
```
<button id="showAddressbtn">Show Address</button>

## 'createAccount'

creates a new Account on the wallet

```typescript
    interface createAccountParams{
      name: string
    }

    const createAccountResult = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, 
        request: {
            method: `createAccount`,
            params: {
              name: `${"Account-name"}`
            }
        }}
    })
    
```
<button id="createAccountButton">CreateAccount</button>

## listAccounts

  returns a list of all stellar accounts in the wallet

  ```typescript
    const accountList = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, 
        request: {
            method: `listAccounts`,
        }}
    })
  ```
## renameAccount

  selects an account by address and changes its name

  ```typescript

    const result = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, 
        request:{
            method: `renameAccount`,
            params:{
              address: `${accountAddress}`,
              name: `${"New-Account-Name"}`
            }
        }}
    })
  ```

## importAccount

  opens a dialog where the user is prompted to import their private key, if they choose

  throws on error

  ```typescript
    const success:boolean = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {
        snapId:`npm:stellar-snap`, 
        request:{
            method: "importAccount",
        }}
    })
  ```

## fund

  funds the current account with testnet stellar

  ```typescript
  const success:boolean = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {
          snapId:`npm:stellar-snap`, 
          request:{
              method: `fund`
          }
        }
    })
  ```




## getAssets

gets all assets for the current Wallet

returns <a href="#/?id=wallet-asset">walletAsset[]</a>

```typescript
  const assets: walletAsset[] = await ethereum.request({
    method: 'wallet_invokeSnap',
    params: {
    snapId:`npm:stellar-snap`, 
    request:{
        method: `getAssets`,
        params:{
          testnet: true
        }
    }}
})
```

## sendAuthRequest

sendAuthRequest is used to sign-in with 

```typescript
    const result = await ethereum.request({
        method: 'wallet_invokeSnap',
        params: {`npm:stellar-snap`, request:{
            method: `sendAuthRequest`,
            params:{
              url: `${endpoint}`,
              data: 
            }
        }}
    })
```

    case 'sendAuthRequest':
      const auth_client = new Auth(wallet.keyPair);
      return await auth_client.signOnPost(params.url, params.data, params.challenge)

## signStr
      const auth = new Auth(wallet.keyPair);
      return await auth.signData(params.challenge);

## dispPrivateKey
      return await Screens.revealPrivateKey(wallet);
    // -------------------------------- Methods That Require a funded Account ------------------------------------------

## getAccountInfo
      if(!wallet_funded){
        await Screens.RequiresFundedWallet(request.method, wallet.address);
        throw new Error('Method Requires Account to be funded');
      }
      return await client.getAccount(wallet.address)
## transfer
      if(!wallet_funded){
        await Screens.RequiresFundedWallet(request.method, wallet.address);
        throw new Error('Method Requires Account to be funded');
      }
      return await operations.transfer(params.to, params.amount);

## sendAsset
      if(!wallet_funded){
        await Screens.RequiresFundedWallet(request.method, wallet.address);
        throw new Error('Method Requires Account to be funded');
      }
      return await operations.transferAsset(params.to, params.amount, params.asset);

## signTransaction
      if(!wallet_funded){
        await Screens.RequiresFundedWallet(request.method, wallet.address);
        throw new Error('Method Requires Account to be funded');
      }
      const txn = await operations.signArbitaryTxn(params.transaction);
      return txn.toXDR();

## signAndSubmitTransaction
      if(!wallet_funded){
        await Screens.RequiresFundedWallet(request.method, wallet.address);
      }
      return await operations.signAndSubmitTransaction(params.transaction);

## createFederationAccount
      return await Screens.setUpFedAccount(wallet);


## 'Soroban'
The Wallet also supports sorroban, To sign a SorobanCall
futurenet must be set to true on the params object.
```javascript
    async function callContract() {
      console.log("here in callContract");
  const sourcePublicKey = await ethereum.request({
          method: 'wallet_invokeSnap',
          params: {snapId:snapId, request:{
            method: 'getAddress',
          }}
      })
  const server = new SorobanClient.Server('https://rpc-futurenet.stellar.org');

  console.log("getting account")
  const account = await server.getAccount(sourcePublicKey);
  console.log("account is: ")
  console.log(account);

  console.log(SorobanClient);

  const contract = new SorobanClient.Contract("CCNLUNUY66TU4MB6JK4Y4EHVQTAO6KDWXDUSASQD2BBURMQT22H2CQU7")
  console.log(contract)
  const arg = SorobanClient.nativeToScVal("world")
  console.log("arg is: ")
  console.log(arg)
  let call_operation = contract.call('hello', arg);
  console.log(call_operation)

  let transaction = new SorobanClient.TransactionBuilder(account, { fee: "150", networkPassphrase: SorobanClient.Networks.FUTURENET })
    .addOperation(call_operation) // <- funds and creates destinationA
    .setTimeout(30)
    .build();

  console.log(transaction)


    const preparedTransaction = await server.prepareTransaction(transaction, SorobanClient.Networks.FUTURENET);
    console.log("prepairedTxn: ");
    console.log(preparedTransaction);
    const tx_XDR = preparedTransaction.toXDR();
    const signedXDR = await ethereum.request(
      {method: 'wallet_invokeSnap',
          params: {
            snapId:snapId, 
            request:{
              method: 'signTransaction',
              params:{
                transaction: tx_XDR,
                futurenet: true
              }
            }
          }
      }
    )
  console.log(signedXDR)
  try{
    
    const transactionResult = await server.sendTransaction(signedXDR);
    console.log(JSON.stringify(transactionResult, null, 2));
    console.log('\nSuccess! View the transaction at: ');
    console.log(transactionResult)
  } catch (e) {
    console.log('An error has occured:');
    console.log(e);
  }
}



```

# types

## DataPacket
An interface that contains infomation about the wallet
```typescript

export interface DataPacket{
    name: string, //comment
    currentAddress: string,
    mainnetAssets?: walletAsset[],
    testnetAssets?: walletAsset[],
    accounts: Array<{name:String, address:String}>
    mainnetXLMBalance: string,
    testnetXLMBalance: string,
    fedName: string | null
}

```

## Wallet Asset

a type that represents a the balance of asset held by a wallet
```typescript

export type walletAsset = AssetBalance | NativeBalance

```

## NativeBalance

```typescript
export interface NativeBalance {
    balance:string,
    liquidity_pool_id?:string,
    limit: string,
    buying_liabilites: string,
    selling_liabilites: string,
    sponser?: string,
    last_modified_ledger: number,
    is_authorized: boolean,
    is_authorized_to_maintain_liabilites: boolean,
    is_clawback_enabled: boolean,
    asset_type: "native",
    asset_issuer: "native"
    asset_code: "XLM"
}
```

## AssetBalance

```typescript
export interface AssetBalance {
    balance: string, //number
    liquidity_pool_id?: string, //number
    limit: string, //number
    buying_liabilites: string, //number
    selling_liabilites: string, //number
    sponser?: string, //address
    last_modified_ledger: number,
    is_authorized: boolean,
    is_authorized_to_maintain_liabilites: boolean,
    is_clawback_enabled: boolean,
    asset_type: "credit_alphanum4"|"credit_alphanum12"
    asset_code: string,
    asset_issuer: string, //address
}

```


## building from Source

```shell
foo@bar:~$ yarn
...

foo@bar:~$ npx mm-snap build

...
Build success: 'src\index.ts' bundled as 'dist\bundle.js'!
Eval Success: evaluated 'dist\bundle.js' in SES!

foo@bar:npx mm-snap serve

Starting server...
Server listening on: http://localhost:8080
```
and just like that you should be good to go.

## Key Generation and Storeage
keys are generated on the fly, anytime a method is invoked.
This works by requesting private entropy from the metamask wallet inside
of the snaps secure execution enviroment, and using that entropy to generate
a users keys. This entropy is static, and based on the users ethereum account.
This means that we at no point store keys, and the fissile material is handled
by metamask.

## Account Recovery
Because keys are handled in this way, when a user recovers their metamask account, they will also recover their stellar
account, which means that there isn't another mnemonic to save. 