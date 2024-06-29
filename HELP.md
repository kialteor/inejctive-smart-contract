
# Injective Chain Smart Contract Deployment Guide

This guide outlines the steps to install `injectived`, upload an Injective smart contract, deploy it on the testnet, and interact with its functions.

## Prerequisites

- Linux environment (commands assume Linux syntax)
- `wget` and `unzip` installed
- `cargo` installed for JSON schema generation

## Installation Steps

### 1. Install `injectived`

```bash
wget https://github.com/InjectiveLabs/injective-chain-releases/releases/download/v1.12.1-1705909076/linux-amd64.zip
unzip linux-amd64.zip
sudo mv injectived peggo /usr/bin
sudo mv libwasmvm.x86_64.so /usr/lib
```

### 2. Set Up Wallet and Get Testnet Tokens

- Create a new key for your wallet 

```bash
# inside the "injective-core-staging" container
injectived keys add testuser
```
- Add key to env

```bash
# add address to env
export INJ_ADDRESS= <your_inj_address>
```

- Obtain test tokens from [Injective Testnet Faucet](https://bwarelabs.com/faucets/injective-testnet).

### 3. Check current balance

```bash
# check the current balance of my wallet
curl -X GET "https://k8s.testnet.lcd.injective.network/cosmos/bank/v1beta1/balances/$INJ_ADDRESS" -H "accept: application/json"
```

### 4. Upload the Wasm Contract and get code id

### 4.1. Upload the Wasm Contract
```bash
# Store the Wasm contract
yes 12345678 | injectived tx wasm store artifacts/my_first_contract.wasm \
--from=$INJ_ADDRESS \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj --gas=2000000 \
--node=https://k8s.testnet.tm.injective.network:443
```

### 4.2. Find code id
  - Check your address on the [Injective Testnet Explorer](https://testnet.explorer.injective.network/), and look for a transaction with the txhash returned from storing the code on chain. The transaction type should be MsgStoreCode.

  - Or you can check the txhash using following query command 

```bash
# check the detail tx hash
injectived query tx < txhash > --node=https://k8s.testnet.tm.injective.network:443
```

### 4.3. Register code id
find the code id and then register to env

```bash
# add code id to env
export CODE_ID= <code_id>
```

### 5. Instantiate the Contract and get contract address

### 5.1. Instantiate the Contract
```bash
INIT='{"count":99}'
yes 12345678 | injectived tx wasm instantiate $CODE_ID $INIT \
--label="CounterTestInstance" \
--from=$INJ_ADDRESS \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj \
--gas=2000000 \
--no-admin \
--node=https://k8s.testnet.tm.injective.network:443
```

### 5.2. Get contract address
You can find the contract address and metadata by:

- Looking on the Testnet Explorer
- Querying the ContractsByCode and ContractInfo APIs
- Querying through the CLI

### 5.3. Add contract address to env

```bash
# add contract address to env
export CONTRACT_ADDR= <contract_address>
```

### 6. Interact with the Contract

#### Check Current State

```bash
injectived query wasm contract-state smart $CONTRACT_ADDR '{"get_count":{}}' \
--node=https://k8s.testnet.tm.injective.network:443 \
--output json
```

#### Execute Functions

```bash
# Increment function
yes 12345678 | injectived tx wasm execute $CONTRACT_ADDR '{"increment":{}}' \
--from=$INJ_ADDRESS \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj --gas=2000000 \
--node=https://k8s.testnet.tm.injective.network:443 \
--output json

# Reset function
yes 12345678 | injectived tx wasm execute $CONTRACT_ADDR '{"reset":{"count":999}}' \
--from=$INJ_ADDRESS \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj --gas=2000000 \
--node=https://k8s.testnet.tm.injective.network:443 \
--output json
```
