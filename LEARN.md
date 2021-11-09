## Staking on Solana ##

### Introduction ###

In this quest we will deep dive into the whats, hows of staking on Solana Blockchain. Staking is one of the ways of generating more money on the existing SOL coins we hold. One can easily choose validators on which to stake their amount or can stake on web applications.

### What will you get after going through this quest ? ###

1. You will understand the difference between Staking and Lending
2. You will understand what Staking is.
3. You will understand the concepts like Validators, Cooling Period, Staking Account, Stake Authority
4. You will understand how to do Staking via Web3.js
5. You will unders

### What is the difference between Staking and Lending? ###


**Staking**: Staking is a process of delegating SOL tokens to different validators available. Validators use this staked amount to validate the block on the blockchain and return the rewards to the stakers.

It enables us to lock some part of the cryptocurrencies we hold (SOL in solana blockchain) as a way to contribute to a blockchain network directly or indirectly to the validators.

People who often opt to try crypto staking, required to agree to not withdraw their cryptocurrencies from this staking process until the end of the agreed time period.

**Lending:** It is a process of lending your crypto holding to someone with some agreed return percentage. It is often called P2P lending as well which means no middleman involved.

But there are other options as well where lending is performed via the application which takes some platform cost.

Interest which you can earn on your crypto can be from 5% to 43% depending on the needs of the other person. It feels like you are lending your fiat to another person at agreed interest rates.

### Why to stake SOL ? ###


Staking SOL is one of the ways of generating income on the idle sitting crypto in your wallet. Everyone wants to earn extra on cryptos which are with them which can eventually turn out a win win situation for the stakers as value of staked crypto gets appreciated as well along with the staking rewards.

### Prerequisites to work on this quest ###

1. Basic knowledge of crypto wallet to approve the transaction (Phantom wallet specifically).
2. A basic react app with wallet connectivity feature.
3. Solana cluster can be : local, devnet or testnet.
4. Funded Solana wallet atleast 0.5 SOL .
5. For better understanding, please go through our first quest on **creating a wallet connection** with react app.

### Folder structure expected of the react app to run this quest ###




* Assuming app name as "Stake SOL"
    * App-Name
        * node_modules
        * public
        * src
            * utils
                * stakeSOL.js
            * App.js
            * Index.js
            * Package.json

### Basic terminologies to digest before we jump into the quest ###


**Validators**: A blockchain validator is someone who is responsible for verifying transactions on a blockchain. Once transactions are verified, they are added to the distributed ledger. In proof of work (PoW) systems like Bitcoin known as miners. 

In proof of stake (PoS) and proof of history (PoH) systems like Ethereum, Solana are known as validators.

Validators are chosen in the blockchain based on the number of tokens they can put on stake (SOL in Solana) or (ETH in Ethereum). Hence the more number of tokens you can put on stake, the more chances will be for the validator to be selected to perform the validation and earn the rewards.

**Staking Account:** To stake the SOL with the validators, a staking account is required. This account act as a holder account for the staked SOL which will have properties like 



* Stake Authority: This property tells who is the owner of the staked amount and can perform below actions
    * Delegating stake
    * Deactivating the stake delegation
* Withdrawer: This property tells who can withdraw the staked amount after the cooling period.
    * Withdrawing un-delegated stake into a wallet address
    * Setting a new withdraw authority
    * Setting a new stake authority

**Delegation Warmup: **When a stake account is delegated, or a delegation is deactivated, the operation does not take effect immediately.

A delegation or deactivation takes several epochs to complete, with a fraction of the delegation becoming active or inactive at each epoch boundary after the transaction containing the instructions has been submitted to the cluster.

**Lockups: **Stake accounts can have a lockup which prevents the tokens they hold from being withdrawn before a particular date or epoch has been reached.

A lockup can only be added when a stake account is first created, but it can be modified later, by the lockup authority.

## Subquest: Staking flow ##

![alt_text](./learn_src/learn_assets/staking-solana.png "Staking Solana Flow")

### Subquest: Staking code walkthrough ###


Let’s understand the **createAccount** util function

**StakeProgram.createAccount :** This instruction is important as it is going to create the staking account with passed parameters. It expects below parameters to be passed as argument



*  **fromPubkey**: This denotes the public key for which staking account is being created
*  **stakePubkey**: This denotes the public key of the newly created keypair which will act as a staking account after the transaction gets successful.
*  **authorized**: This parameter expects the instance of the Authorized class which holds the information of staker and withdrawer. In our case it is **authorizedStakerInstance.**
*  **lamports**: This denotes how many lamports can this staking account hold and which can eventually be used to delegate it to the validator.

File: src/stakeSOL.js


```
StakeProgram.createAccount({
           fromPubkey: provider.publicKey,
           stakePubkey: newStakingAccount.publicKey,
           authorized: authorizedStakerInstance,
           lamports:totalSolToStake
         })
```


Let’s understand the **delegate** util function

**StakeProgram.delegate: **This instruction is important as it is going to delegate the lamports from the staking account which was created in the previous step.

It expects below parameters to perform the delegation



* **stakePubkey**: This expects the public key of the newly created keypair which was used as a staking account.
* **authorizedPubkey**: This expects the staker’s public key as we authorized it in the previous instruction to delegate the lamports.
* **votePubkey**: This expects the public key of the validator to which we want to stake/delegate our SOL tokens.

File: src/stakeSOL.js


```
StakeProgram.delegate({
           stakePubkey: newStakingAccount.publicKey,
           authorizedPubkey:staker,
           votePubkey: votingAccountToDelegate
         })
```


As a next step, we need to sign the transaction with the correct signer. In our case, the singer will be the wallet provider which we also made as the staker of the newly created staking account.

To get these transactions on the blockchain it needs to be signed from the wallet owner and sent for the validation.

Few key variables to be used in above function



* transaction.recentBlockhash : As solana works on **proof of history**, hence every transaction in the solana blockchain requires the latest blockhash to be associated with the new transaction. Without the getRecentBlockhash, the validators will not be able to verify the transaction.
* transaction.partialSign: It is an important step as our transaction contains multiple instructions hence we need to partially sign the first transaction to proceed with the other intermediate instructions to be signed successfully.
* transaction.feePayer: It is an important step as our transaction needs the fee payer to sign the transaction, in our case it will be the wallet provider.

As the next steps, we are signing the transaction and then sending it over the Solana Blockchain to be validated and added on the Solana Blockchain.


```
const transaction = new Transaction().add(
       // createAccount
         StakeProgram.createAccount({
           fromPubkey: provider.publicKey,
           stakePubkey: newStakingAccount.publicKey,
           authorized: authorizedStakerInstance,
           lamports:totalSolToStake
         })
     );
     transaction.recentBlockhash = (
         await connection.getRecentBlockhash()
       ).blockhash;
   transaction.feePayer = provider.publicKey;
   transaction.add(
       StakeProgram.delegate({
           stakePubkey: newStakingAccount.publicKey,
           authorizedPubkey:staker,
           votePubkey: votingAccountToDelegate
         })
   );
   transaction.partialSign(newStakingAccount);
```


Let’s understand the **stakeSOL** util function

**stakeSOL: **This is a utility which will initiate the staking account creation process along with staking amount with the validator.

This function expects below parameters



1. **totalSolToStake: **The amount of SOL in lamports you want to delegate for staking.
2. **provider**: This is the wallet provider who will be signing the transaction. This provider will also act as an owner of the staking account unless changed.
3. **connection**: This is the connection instance to one of the Solana cluster (devnet for testing purpose)

* We will be using a hardcoded validator for testing purposes, in real application you are free to change it to another validator or fetch validators from other resources.

**votingAccountToDelegate: **This variable holds the public key of the validator to which we will delegate the SOL. For the tutorial purpose we are using the hardcoded public key of the validator.

**newStakingAccount: **This variable holds the newly created keypair which will act as a staking account and can delegate the SOL to the validator.

**staker: **This variable holds the stake authority owner of the staking account as we discussed on the top. Most of the time it will be kept the same as the wallet owner and withdraw unless changed.

**authorizedStakerInstance: **This variable holds the instance of the Authorized class which creates an object of staker and withdrawer. This instance will be used in the staking account creation instruction to tag the correct staker and withdrawer of the staking account.

File: src/stakeSOL.js


```
import { Authorized, Keypair, PublicKey, StakeProgram, Transaction } from "@solana/web3.js"

export const stakeSOL = async (totalSolToStake, provider,connection) => {
   totalSolToStake = totalSolToStake || 1 * 1000000000  //1 SOL in lamports
   if (!provider || (provider && !provider.isConnected)) {
     return "Wallet is not connected, please connect the wallet"

   }

   //TODO: hardcoded validator's voting account from solanaBeach
   const votingAccountToDelegate = new PublicKey('BXKwE3p8gmwwnepGxpgo1bUSU1pLzGZoNUC1dFUcbG3t')

   const newStakingAccount = Keypair.generate();
     const staker = provider.publicKey;
     const withdrawer = staker;
     const authorizedStakerInstance = new Authorized(staker, withdrawer);
     const transaction = new Transaction().add(
       // createAccount
         StakeProgram.createAccount({
           fromPubkey: provider.publicKey,
           stakePubkey: newStakingAccount.publicKey,
           authorized: authorizedStakerInstance,
           lamports:totalSolToStake
         })
     );
     transaction.recentBlockhash = (
         await connection.getRecentBlockhash()
       ).blockhash;
   transaction.feePayer = provider.publicKey;
   transaction.add(
       StakeProgram.delegate({
           stakePubkey: newStakingAccount.publicKey,
           authorizedPubkey:staker,
           votePubkey: votingAccountToDelegate
         })
   );
   transaction.partialSign(newStakingAccount);
   try{
     let signed = await provider.signTransaction(transaction);
     console.log('Got signature, submitting transaction', signed);
     let signature = await connection.sendRawTransaction(signed.serialize());
     console.log(
       'Submitted transaction ' + signature + ', awaiting confirmation'
     );
     await connection.confirmTransaction(signature);
     console.log('Transaction ' + signature + ' confirmed');


     return {newStakingAccountPubKey: newStakingAccount.publicKey, transactionId: signature}
   }catch(err){
     console.log(err,'----err----')
   }

}
```


## Subquest : How to run this quest ##
1. Install phantom wallet chrome extension and add some SOL to the wallet.
2. Create a react app 
    1. npx create-react-app stake-solana
    2. cd stake-solana
3. Copy paste the below initiator code in the App.js to invoke stakeSol.js

```
import './App.css';
import { Connection } from "@solana/web3.js";
import * as web3 from '@solana/web3.js';
import { useEffect, useState } from 'react';
import { stakeSOL } from './utils/stakeSOL';

const NETWORK = web3.clusterApiUrl("devnet");
const connection = new Connection(NETWORK);

function App() {

 const [provider, setProvider] = useState()
 const [providerPubKey, setProviderPub] = useState()
 const [stakeSOLDetails, setStakeSOLDetails ] = useState({})
const stakeSOLHandler = async () => {
 try{
     const totalSolToStake = 1 * web3.LAMPORTS_PER_SOL; // in SOL
     const result = await stakeSOL(totalSolToStake, provider, connection)
     setStakeSOLDetails(result)
 }catch(err){
   console.log(err,'---stake error---')
 }
}

const connectToWallet = () =>{
 if(!provider && window.solana){
   setProvider(window.solana)
 }
 if(!provider){
   console.log("No provider found")
   return
 }
 if(provider && !provider.isConnected){
   provider.connect()
 }
}


 useEffect(() => {
   if (provider) {
       provider.on("connect", async() => {
         console.log("wallet got connected")
         setProviderPub(provider.publicKey)

       });
       provider.on("disconnect", () => {
         console.log("Disconnected from wallet");
       });
   }
 }, [provider]);

 useEffect(() => {
   if ("solana" in window && !provider) {
     console.log("Phantom wallet present")
     setProvider(window.solana)
   }
 },[])

 return (
   <div className="App">
     <header className="App-header">
        
          <button onClick={connectToWallet}> {providerPubKey ? 'Connected' : 'Connect'} to wallet {providerPubKey ? (providerPubKey).toBase58() : ""}</button>
         <button onClick={stakeSOLHandler}> {stakeSOLDetails && stakeSOLDetails.newStakingAccountPubKey? `Staked SOL acccount: ${stakeSOLDetails.newStakingAccountPubKey}` : `Stake SOL` } </button>
     </header>
   </div>
 );
}

export default App;

```
4. To install the required dependencies:  “npm install” from the root folder of the project.

5. To run the project : “npm run start” from the root folder of the project.

6. Navigate to[ http://localhost:3000](http://localhost:3000) 

7. Click on the 'Connect to wallet' button to connect with the wallet.

8. Once connected, click on "Stake SOL" to stake SOL token to a validator wallet

9. Once the transaction is confirmed, you can view the stake account in the Phantom Wallet.


![alt_text](./learn_src/learn_assets/wallet-staking.png "Staking Wallet")


## Subquest : What next? ##

** What can you build taking this quest as a base ? **
1. You can create an application which can take users' SOL token and stake it to a validator.
2. You can build an application where based on the different algorithms and logic, you can recommend best validator to the user like step.finance