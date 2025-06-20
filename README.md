# drt-chain-simulator-examples-py
Example scripts for chain simulator

## 1. Move balance script `./01-move-balance/main.py`

This script does the following:
* generates a new key: the sender key
* calls the special `send-user-funds` REST API endpoint route so the provided sender address will be minted with native tokens. 
This is a mandatory step in almost all tests and examples as we need native tokens to succeed in issuing transactions.
* generates a new key: the receiver key
* creates a transaction from sender -> receiver having the amount of 1 rEWA (1*10^18 = 1000000000000000000)
* tests & prints the amount remaining in sender's account & the balance of the receiver account.


## 2. Issue and transfer a fungible token script `./02-fungible-dcdt-interaction/main.py`

This script does the following:
* generates a new key: the sender key
* calls the special `send-user-funds` REST API endpoint route so the provided sender address will be minted with native tokens.
  This is a mandatory step in almost all tests and examples as we need native tokens to succeed in issuing transactions.
* creates a special issuing DCDT transaction to create a `TTKN-xxyyzz` token identifier. Note that the sequence `xxyyzztt` will 
be automatically generated by the blockchain
* generates a new key: the receiver key
* creates a transaction from sender -> receiver transferring the amount of 500 TTKN-xxyyzz
* tests & prints the DCDT amount remaining in sender's account & the DCDT balance of the receiver account.


## 3. Deploy and interact with a smartcontract `./03-smartcontract-interaction/main.py`

This script does the following:
* generates a new key: the sender key
* calls the special `send-user-funds` REST API endpoint route so the provided sender address will be minted with native tokens.
  This is a mandatory step in almost all tests and examples as we need native tokens to succeed in issuing transactions.
* opens and loads the file `./adder.wasm`
* deploys the adder contract. The owner of the contract is the sender address.
* performs a vm-query operation that will execute (in a sandbox environment) a specified contract endpoint. The endpoint is 
`getSum` and returns an integer value corresponding to a variable in the contract instance. After the contract deployment
the value returned will be `0` (up until we will change it through another endpoint call)
* creates a new transaction that will call the endpoint `add` and provide a dummy value of `10`. The contract's endpoint is 
written in such a way that the internal contract's variable will be incremented with the provided value.
* performs another vm-query operation that will execute the `getSum` endpoint, 
returning the integer value of `10`.

For reference, this is how the adder contract can be written in Rust:
```rust
#![no_std]

use dharitri_sc::imports::*;

pub mod adder_proxy;

/// One of the simplest smart contracts possible,
/// it holds a single variable in storage, which anyone can increment.
#[dharitri_sc::contract]
pub trait Adder {
    #[view(getSum)]
    #[storage_mapper("sum")]
    fn sum(&self) -> SingleValueMapper<BigUint>;

    #[init]
    fn init(&self, initial_value: BigUint) {
        self.sum().set(initial_value);
    }

    #[upgrade]
    fn upgrade(&self, initial_value: BigUint) {
        self.init(initial_value);
    }

    /// Add desired amount to the storage variable.
    #[endpoint]
    fn add(&self, value: BigUint) {
        self.sum().update(|sum| *sum += value);
    }
}
```
