# MultiSig Wallet
 
 This walkthrough is based on [this project](./Multisig_wallet_info.md) by Nate Rush.
The solution is based on [this MultiSignature Wallet](https://github.com/ConsenSys/MultiSigWallet) found in the ConsenSys github repository.

# What is a Multisignature wallet? 

A multisignature wallet is an account that requires some m-of-n quorum of approved private keys to approve a transaction before it is executed. 

Ethereum implements multisignature wallets slightly differently than Bitcoin does. In Ethereum, multisignature wallets are implemented as a smart contract, that each of the approved external accounts sends a transaction to in order to "sign" a group transaction. 

Following this project spec designed by the UPenn Blockchain Club, you will now create your own multisignature wallet contract. 

**Note: It is not suggested that you use this multisignature wallet with any real funds, but rather use a far more deeply audited one such as the [Gnosis multisignature wallet.](https://wallet.gnosis.pm/)**

Interacting with the Contract
============

Now that we have a basic MultiSignature Wallet, let’s interact with the Multisig Wallet and see how it works.


Copy the contract that we developed in Remix into the truffle project directory provided. 


You can see that the project directory comes with a SimpleStorage.sol contract. This is the contract that we are going to be calling from the Multisig contract.


If you look in the migrations directory you will see the deployment script that truffle will use to deploy the SimpleStorage contract as well as the MultiSig Wallet.


The truffle deployer allows us to access accounts, which is useful given that the MultiSig contract constructor requires an array of owner addresses as as well as the number of required confirmations to execute a transaction.


The owners array and the deployment scripts are already in the file.

```javascript
    const owners = [accounts[0], accounts[1]]
    deployer.deploy(MultiSig, owners, 2)
```

We are only going to require 2 confirmations for the sake of simplicity.


To deploy the contracts, start the development environment by running `truffle develop` in a terminal window at the project directory. The truffle command line will appear

```
truffle(develop)> 
```

Deploy the contracts
```
truffle(develop)> migrate 
```
If `migrate` does not work, try `migrate --reset`.

And then get the deployed instances of the SimpleStorage.sol and MultiSignatureWallet.sol contracts.

```
truffle(develop)> var ss = await SimpleStorage.at(SimpleStorage.address)
truffle(develop)> var ms = await MultiSignatureWallet.at(MultiSignatureWallet.address)
```

Check the state of the the SimpleStorage contract

```
truffle(develop)> ss.storedData.call()
<BN: 0>
```

This means that it is 0. You can verify by waiting for the promise to resolve and converting the answer to a string. Try it with:

```
ss.storedData.call().then(res => { console.log( res.toString(10) )} )
0
```

Let’s submit a transaction to update the state of the SimpleStorage contract to the MultiSig contract. `SumbitTransaction` takes the address of the destination contract, the value to send with the transaction and the transaction data, which includes the encoded function signature and input parameters.


If we want to update the SimpleStorage contract data to be 5, the encoded function signature and input parameters would look like this:

```javascript
var encoded = '0x60fe47b10000000000000000000000000000000000000000000000000000000000000005'
```

Let's get the available accounts and then make a call to the MultiSig contract:

```
truffle(develop)> var accounts = await web3.eth.getAccounts()
truffle(develop)> ms.submitTransaction(ss.address, 0, encoded, {from: accounts[0]})
```

And we see the transaction information printed in the terminal window. In the logs, we can see that a “Submission” event was fired, as well as a “Confirmation” event, which is what we expect. 


The current state of the MultiSig has one transaction that has not been executed and has one confirmation (from the address that submitted it). One more confirmation should cause the transaction to execute. Let’s use the second account to confirm it. The `confirmTransaction` function takes one input, the index of the Transaction to confirm.

```
truffle(develop)> ms.confirmTransaction(0, {from: accounts[1]})
```

The transaction information is printed in the terminal. You should see two log events this time as well. A “Confirmation” event as well as an “Execution” event. This indicates that the call to SimpleStorage executed successfully. If it didn’t execute successfully, we would see an “ExecutionFailure” event there instead.


We can verify that the state of the contract was updated by running

```
truffle(develop)> ss.storedData.call()
<BN: 5>
```

The `storedData` is now 5. And we can check that the address that updated the SimpleStorage contract was the MultiSig Wallet.

```
truffle(develop)> ss.caller.call()
'0x855d1c79ad3fb086d516554dc7187e3fdfc1c79a'
truffle(develop)> ms.address
'0x855d1c79ad3fb086d516554dc7187e3fdfc1c79a'
```

The two addresses are the same!

Further Reading
============
- [ConsenSys MultiSig Repo](https://medium.com/hellogold/ethereum-multi-signature-wallets-77ab926ab63b)
- [List of MultiSig Contracts on Mainnet](https://medium.com/talo-protocol/list-of-multisig-wallet-smart-contracts-on-ethereum-3824d528b95e)
- [What is a MultiSig Wallet?](https://medium.com/hellogold/ethereum-multi-signature-wallets-77ab926ab63b)
- [Unchained Capital's Multi-Sig](https://blog.unchained-capital.com/a-simple-safe-multisig-ethereum-smart-contract-for-hardware-wallets-a107bd90bb52)
- [Grid+: Toward an Ethereum MultiSig Standard](https://blog.gridplus.io/toward-an-ethereum-multisig-standard-c566c7b7a3f6)
