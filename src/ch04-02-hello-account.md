# Hello World Account Contract

> **NOTE:**
THIS CHAPTER NEEDS TO BE UPDATED TO REFLECT THE NEW SYNTAX FOR ACCOUNT CONTRACTS. PLEASE DO NOT USE THIS CHAPTER AS A REFERENCE UNTIL THIS NOTE IS REMOVED.

**CONTRIBUTE: This subchapter is missing an example of declaration, deployment and interaction with the contract. We would love to see your contribution! Please submit a PR.**

In this chapter, we will explore the fundamentals of account contracts
in Starknet using an example "Hello World" account contract written in
Cairo language. You can find it in the contracts directory of this
chapter in the Book’s repository (TODO: add link).

```rust
use starknet::account::Call;

// IERC6 obtained from Open Zeppelin's cairo-contracts/src/account/interface.cairo
#[starknet::interface]
trait ISRC6<TState> {
    fn __execute__(self: @TState, calls: Array<Call>) -> Array<Span<felt252>>;
    fn __validate__(self: @TState, calls: Array<Call>) -> felt252;
    fn is_valid_signature(self: @TState, hash: felt252, signature: Array<felt252>) -> felt252;
}

#[starknet::contract]
mod HelloAccount {
    use starknet::VALIDATED;
    use starknet::account::Call;
    use starknet::get_caller_address;

    #[storage]
    struct Storage {} // Empty storage. No public key is stored.

    #[external(v0)]
    impl SRC6Impl of super::ISRC6<ContractState> {
        fn is_valid_signature(
            self: @ContractState, hash: felt252, signature: Array<felt252>
        ) -> felt252 {
            // No signature is required so any signature is valid.
            VALIDATED
        }
        fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
            let hash = 0;
            let mut signature: Array<felt252> = ArrayTrait::new();
            signature.append(0);
            self.is_valid_signature(hash, signature)
        }
        fn __execute__(self: @ContractState, calls: Array<Call>) -> Array<Span<felt252>> {
            let sender = get_caller_address();
            assert(sender.is_zero(), 'Account: invalid caller');
            let Call{to, selector, calldata } = calls.at(0);
            let _res = starknet::call_contract_syscall(*to, *selector, calldata.span()).unwrap();
            let mut res = ArrayTrait::new();
            res.append(_res);
            res
        }
    }
}
```

Compile your project to ensure the setup is correct:

```bash
scarb build
```

## SNIP-6 Standard

To define an account contract, implement the `ISRC6` trait:

```rust
#[starknet::interface]
trait ISRC6<TState> {
    fn __execute__(self: @TState, calls: Array<Call>) -> Array<Span<felt252>>;
    fn __validate__(self: @TState, calls: Array<Call>) -> felt252;
    fn is_valid_signature(self: @TState, hash: felt252, signature: Array<felt252>) -> felt252;
}
```

The `__execute__` and `__validate__` functions are designed for exclusive use by the Starknet protocol to enhance account security. Despite their public accessibility, only the Starknet protocol can invoke these functions, identified by using the zero address. In this minimal account contract we will not enforce this restriction, but we will do it in the next examples.

## Validating Transactions

The `is_valid_signature` function is responsible for this validation, returning `VALIDATED` if the signature is valid. The `VALIDATED` constant is imported from the `starknet` module.

```rust
use starknet::VALIDATED;
```

Notice that the `is_valid_signature` function accepts all the transactions as valid. We are not storing a public key in the contract, so we cannot validate the signature. We will add this functionality in the next examples.

```rust
fn is_valid_signature(
            self: @ContractState, hash: felt252, signature: Array<felt252>
        ) -> felt252 {
            // No signature is required so any signature is valid.
            VALIDATED
        }
```

The `__validate__` function calls the `is_valid_signature` function with a dummy hash and signature. The `__validate__` function is called by the Starknet protocol to validate the transaction. If the transaction is not valid, the execution of the transaction is aborted.

```rust
fn __validate__(self: @ContractState, calls: Array<Call>) -> felt252 {
            let hash = 0;
            let mut signature: Array<felt252> = ArrayTrait::new();
            signature.append(0);
            self.is_valid_signature(hash, signature)
        }
```

In other words we have implemented a contract that accepts all the transactions as valid. We will add the signature validation in the next examples.

## Executing Transactions

The `__execute__` function is responsible for executing the transaction. In this minimal account contract we will only execute a single call. We will add the multicall functionality in the next examples.

```rust
fn __execute__(self: @ContractState, calls: Array<Call>) -> Array<Span<felt252>> {
            let Call{to, selector, calldata } = calls.at(0);
            let _res = starknet::call_contract_syscall(*to, *selector, calldata.span()).unwrap();
            let mut res = ArrayTrait::new();
            res.append(_res);
            res
        }
```

The `__execute__` function calls the `call_contract_syscall` function from the `starknet` module. This function executes the call and returns the result. The `call_contract_syscall` function is a Starknet syscall, which means that it is executed by the Starknet protocol. The Starknet protocol is responsible for executing the call and returning the result. The Starknet protocol will also validate the call, so we do not need to validate the call in the `__execute__` function.

## Deploying the Contract

[TODO]
