# Local Development

This guide will get you started deploying `cw20` smart contracts on a local Titan network running on your computer.

We'll use the `cw20-base` contract from [CosmWasm's collection of specifications and contracts](https://github.com/CosmWasm/cw-plus) designed for production use on real networks. `cw20-base` is a basic implementation of a `cw20` compatible contract that can be imported in any custom contract you want to build on. It contains a straightforward but complete implementation of the cw20 spec along with all extensions. `cw20-base` can be deployed as-is or imported by other contracts.

### Prerequisites

Install Go, Rust, and other Cosmwasm dependencies by following the instructions:

1. [Go](https://docs.cosmwasm.com/docs/getting-started/installation#go)
2. [Rust](https://docs.cosmwasm.com/docs/getting-started/installation#rust)

Before starting, make sure you have [`rustup`](https://rustup.rs/) along with recent versions of `rustc` and `cargo` installed. Currently, we are testing on Rust v1.58.1+.

You also need to have the `wasm32-unknown-unknown` target installed as well as the `cargo-generate` Rust crate.

You can check versions via the following commands:

```bash
rustc --version
cargo --version
rustup target list --installed
# if wasm32 is not listed above, run this
rustup target add wasm32-unknown-unknown
# to install cargo-generate, run this
cargo install cargo-generate
```

### Titand

Make sure you have `titand` installed locally. You can follow the guide to get `titand` and other prerequisites running locally.

Once you have `titand` installed, you should also start a local chain instance.

### Compile CosmWasm Contracts

In this step, we will get all CW production template contracts and compile them using the [CosmWasm Rust Optimizer](https://github.com/CosmWasm/rust-optimizer) Docker image for compiling multiple contracts (called `workspace-optimizer`)—see [here](https://hub.docker.com/r/cosmwasm/workspace-optimizer/tags) (x86) or [here](https://hub.docker.com/r/cosmwasm/workspace-optimizer-arm64/tags) (ARM) for latest versions. This process may take a bit of time and CPU power.

<pre class="language-bash"><code class="lang-bash">git clone https://github.com/CosmWasm/cw-plus
<strong>cd cw-plus
</strong></code></pre>

Non ARM (Non-Apple silicon) devices:

```bash
docker run --rm -v "$(pwd)":/code \
--mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
--mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
cosmwasm/workspace-optimizer:0.12.12
```

Alternatively for Apple Silicon devices (M1, M2, etc.) please use:

```bash
docker run --rm -v "$(pwd)":/code \
--mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
--mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
cosmwasm/workspace-optimizer-arm64:0.12.12
```

The docker script builds and optimizes all the CW contracts in the repo, with the compiled contracts located under the `artifacts` directory. Now we can deploy the `cw20_base.wasm` contract (`cw20_base-aarch64.wasm` if compiled on an ARM device).

### Upload the CosmWasm Contract to the Chain

```bash
# inside the CosmWasm/cw-plus repo
yes 12345678 | titand tx wasm store artifacts/cw20_base.wasm --from=genesis --chain-id="titan_18889-1" --yes --gas-prices=500000000atkx --gas=20000000
```

**Output:**

```
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: F50155DDEF7D9AF2A0A1280603B8572DBDAA9E29B9CB00429872386BF4CA500B
```

Then query the transaction by the `txhash` to verify the contract was indeed deployed.

{% code fullWidth="false" %}
```bash
titand query tx F50155DDEF7D9AF2A0A1280603B8572DBDAA9E29B9CB00429872386BF4CA500B
```
{% endcode %}

**Output:**

{% code fullWidth="false" %}
```bash
code: 0
codespace: ""
data: 124E0A262F636F736D7761736D2E7761736D2E76312E4D736753746F7265436F6465526573706F6E73651224082C1220CDB1C2AAAADC5A62623B0B7AACE0F572817A605F3B69311692832325BE11C07A
events:
- attributes:
  - index: true
    key: spender
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  - index: true
    key: amount
    value: 656017230000000000atkx
  type: coin_spent
- attributes:
  - index: true
    key: receiver
    value: titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs
  - index: true
    key: amount
    value: 656017230000000000atkx
  type: coin_received
- attributes:
  - index: true
    key: recipient
    value: titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs
  - index: true
    key: sender
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  - index: true
    key: amount
    value: 656017230000000000atkx
  type: transfer
- attributes:
  - index: true
    key: sender
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  type: message
- attributes:
  - index: true
    key: fee
    value: 656017230000000000atkx
  - index: true
    key: fee_payer
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  type: tx
- attributes:
  - index: true
    key: acc_seq
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q/2
  type: tx
- attributes:
  - index: true
    key: signature
    value: pPmr9UpCem4w0Pv2Q16d9OE9ArXSXTAQxgnFpPH5FjMrUOcALW7xf+TPZU2pYLcRDd56aQ8VHOyv9f5B195jWQ==
  type: tx
- attributes:
  - index: true
    key: action
    value: /cosmwasm.wasm.v1.MsgStoreCode
  - index: true
    key: sender
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  - index: true
    key: module
    value: wasm
  type: message
- attributes:
  - index: true
    key: code_checksum
    value: cdb1c2aaaadc5a62623b0b7aace0f572817a605f3b69311692832325be11c07a
  - index: true
    key: code_id
    value: "44"
  type: store_code
- attributes:
  - index: true
    key: spender
    value: titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs
  - index: true
    key: amount
    value: 149059130000000000atkx
  type: coin_spent
- attributes:
  - index: true
    key: receiver
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  - index: true
    key: amount
    value: 149059130000000000atkx
  type: coin_received
- attributes:
  - index: true
    key: recipient
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  - index: true
    key: sender
    value: titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs
  - index: true
    key: amount
    value: 149059130000000000atkx
  type: transfer
- attributes:
  - index: true
    key: sender
    value: titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs
  type: message
- attributes:
  - index: true
    key: amount
    value: 149059130000000000atkx
  - index: true
    key: fee_payer
    value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  type: refund
gas_used: "4608710"
gas_wanted: "5963793"
height: "7826945"
info: ""
logs:
- events:
  - attributes:
    - key: action
      value: /cosmwasm.wasm.v1.MsgStoreCode
    - key: sender
      value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
    - key: module
      value: wasm
    type: message
  - attributes:
    - key: code_checksum
      value: cdb1c2aaaadc5a62623b0b7aace0f572817a605f3b69311692832325be11c07a
    - key: code_id
      value: "44"
    type: store_code
  log: ""
  msg_index: 0
raw_log: '[{"msg_index":0,"events":[{"type":"message","attributes":[{"key":"action","value":"/cosmwasm.wasm.v1.MsgStoreCode"},{"key":"sender","value":"titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q"},{"key":"module","value":"wasm"}]},{"type":"store_code","attributes":[{"key":"code_checksum","value":"cdb1c2aaaadc5a62623b0b7aace0f572817a605f3b69311692832325be11c07a"},{"key":"code_id","value":"44"}]}]}]'
timestamp: "2025-05-22T07:36:46Z"
tx:
  '@type': /cosmos.tx.v1beta1.Tx
  auth_info:
    fee:
      amount:
      - amount: "656017230000000000"
        denom: atkx
      gas_limit: "5963793"
      granter: ""
      payer: ""
    signer_infos:
    - mode_info:
        single:
          mode: SIGN_MODE_DIRECT
      public_key:
        '@type': /ethermint.crypto.v1.ethsecp256k1.PubKey
        key: A3UgyJ7AMzECx+jD/vJ694O+ZY2Kn1TB7w9STYsowvBJ
      sequence: "2"
    tip: null
  body:
    extension_options: []
    memo: titanlab.io
    messages:
    - '@type': /cosmwasm.wasm.v1.MsgStoreCode
      instantiate_permission:
        addresses:
        - ""
        permission: Everybody
      sender: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
      wasm_byte_code: <— wasm-by-code-here —>
    non_critical_extension_options: []
    timeout_height: "0"
  signatures:
  - pPmr9UpCem4w0Pv2Q16d9OE9ArXSXTAQxgnFpPH5FjMrUOcALW7xf+TPZU2pYLcRDd56aQ8VHOyv9f5B195jWQ==
txhash: F50155DDEF7D9AF2A0A1280603B8572DBDAA9E29B9CB00429872386BF4CA500B

```
{% endcode %}

Inspecting the output more closely, we can see the `code_id` of `44` for the contract

```bash
logs:
- events:
  - attributes:
    - key: action
      value: /cosmwasm.wasm.v1.MsgStoreCode
    - key: sender
      value: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
    - key: module
      value: wasm
    type: message
  - attributes:
    - key: code_checksum
      value: cdb1c2aaaadc5a62623b0b7aace0f572817a605f3b69311692832325be11c07a
    - key: code_id
      value: "44"
    type: store_code
  log: ""
  msg_index: 0
```

We’ve uploaded the contract code, but we still need to instantiate the contract.

### Instantiate the Contract

Before instantiating the contract, let's take a look at the CW-20 contract function signature for `instantiate`.

```rust
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    mut deps: DepsMut,
    _env: Env,
    _info: MessageInfo,
    msg: InstantiateMsg,
) -> Result<Response, ContractError> {
```

Notably, it contains the `InstantiateMsg` parameter which contains the token name, symbol, decimals, and other details.

```rust
#[derive(Serialize, Deserialize, JsonSchema, Debug, Clone, PartialEq)]
pub struct InstantiateMsg {
    pub name: String,
    pub symbol: String,
    pub decimals: u8,
    pub initial_balances: Vec<Cw20Coin>,
    pub mint: Option<MinterResponse>,
    pub marketing: Option<InstantiateMarketingInfo>,
}
```

The first step of instantiating the contract is to select an address to supply with our initial CW20 token allocation. In our case, we can just use the genesis address since we have the keys already set up, but feel free to generate new addresses and keys.

{% hint style="warning" %}
Make sure you have the private keys for the address you choose—you won't be able to test token transfers from the address otherwise. In addition, the chosen address must be a valid address on the chain (the address must have received funds at some point in the past) and must have balances to pay for gas when executing the contract.
{% endhint %}

To find the genesis address, run:

```bash
yes 12345678 | titand keys show genesis
```

**Output:**

```bash
- name: genesis
  type: local
  address: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  pubkey: '{"@type":"/ethermint.crypto.v1.ethsecp256k1.PubKey","key":"A3UgyJ7AMzECx+jD/vJ694O+ZY2Kn1TB7w9STYsowvBJ"}'
  mnemonic: ""
```

Run the CLI command with `code_id` `44` along with the JSON encoded initialization arguments (with your selected address) and a label (a human-readable name for this contract in lists) to instantiate the contract:

```bash
CODE_ID=44
INIT='{"name":"Albcoin","symbol":"ALB","decimals":6,"initial_balances":[{"address":"titan10cfy5e6qt2zy55q2w2ux2vuq862zcyf4fmfpj3","amount":"69420"}],"mint":{"minter":"titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q"},"marketing":{}}'
yes 12345678 | titand tx wasm instantiate $CODE_ID $INIT --label="Albcoin Token" --from=genesis --chain-id="titan_18889-1" --yes --gas-prices=500000000atkx --gas=20000000 --no-admin
```

Now the address of the instantiated contract can be obtained on [http://localhost:26657/swagger/#/Query/ContractsByCode](http://localhost:10337/swagger/#/Query/ContractsByCode)

And the contract info metadata can be obtained on [http://localhost:26657/swagger/#/Query/ContractInfo](http://localhost:10337/swagger/#/Query/ContractInfo) or by CLI query

```bash
CONTRACT=$(titand query wasm list-contract-by-code $CODE_ID --output json | jq -r '.contracts[-1]')
titand query wasm contract $CONTRACT
```

**Output:**

```bash
titand query wasm contract $CONTRACT
address: titan16awrdehvla6kqq2dk5v4m6ze83qfg8trpw55qc8rvfrg9qdmfvhq8zaw94
contract_info:
  admin: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  code_id: "44"
  created:
    block_height: "7827043"
    tx_index: "0"
  creator: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  extension: null
  ibc_port_id: ""
  label: meow-nft
```

### Querying the contract

The entire contract state can be queried with:

```bash
titand query wasm contract-state all $CONTRACT
```

**Output:**

```bash
models:
- key: 636F6E666967
  value: eyJvd25lciI6InRpdGFuMTZucWhsNWRmOXV5eTg5OXEza3E4bmMycW5kbXBmMno0cnFleTBxIiwiYmFzZV90b2tlbl91cmkiOiJpcGZzOi8vbWVvdyIsIm1pbnRfZmVlIjp7ImRlbm9tIjoiZmFjdG9yeS90aXRhbjE2bnFobDVkZjl1eXk4OTlxM2txOG5jMnFuZG1wZjJ6NHJxZXkwcS9tZW93IiwiYW1vdW50IjoiMjAwMDAwMDAwIn0sInZlc3RpbmdfYW1vdW50IjoiMjAwMDAwMDAiLCJ2ZXN0aW5nX2NvdW50IjoiMTIiLCJ2ZXN0aW5nX2ludGVydmFsIjoiMjU5MjAwMCIsInRva2VuX2xpbWl0IjoiMTAwIiwid2l0aGRyYXdfYWRkcmVzcyI6InRpdGFuMTZucWhsNWRmOXV5eTg5OXEza3E4bmMycW5kbXBmMno0cnFleTBxIn0=
- key: 636F6E74726163745F696E666F
  value: eyJjb250cmFjdCI6ImN3NzIxLXZlc3RpbmciLCJ2ZXJzaW9uIjoiMC4xOC4xIn0=
- key: 6E66745F696E666F
  value: eyJuYW1lIjoiTWVvdyBDb25uY2lsIFBhc3MiLCJzeW1ib2wiOiJNQ1AifQ==
pagination:
  next_key: null
  total: "0"
```

The individual user’s token balance can also be queried with:

{% code fullWidth="false" %}
```bash
BALANCE_QUERY='{"balance": {"address": "titan10cfy5e6qt2zy55q2w2ux2vuq862zcyf4fmfpj3"}}'
titand query wasm contract-state smart $CONTRACT "$BALANCE_QUERY" --output json
```
{% endcode %}

**Output:**

```bash
{"data":{"balance":"69420"}}
```

### Transferring Tokens

```bash
TRANSFER='{"transfer":{"recipient":"titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs","amount":"420"}}'
yes 12345678 | titand tx wasm execute $CONTRACT "$TRANSFER" --from genesis --chain-id="titan_18889-1" --yes --gas-prices=500000000atkx --gas=20000000
```

Then confirm the balance transfer occurred successfully with:

```bash
# first address balance query
BALANCE_QUERY='{"balance": {"address": "titan10cfy5e6qt2zy55q2w2ux2vuq862zcyf4fmfpj3"}}'
titand query wasm contract-state smart $CONTRACT "$BALANCE_QUERY" --output json
```

**Output:**

```bash
{"data":{"balance":"69000"}}
```

And confirm the recipient received the funds:

```bash
# recipient's address balance query
BALANCE_QUERY='{"balance": {"address": "titan17xpfvakm2amg962yls6f84z3kell8c5l6sf5cs"}}'
titand query wasm contract-state smart $CONTRACT "$BALANCE_QUERY" --output json
```

**Output:**

```bash
{"data":{"balance":"420"}}
```

## Testnet Development

Here are the main differences between a `local` and `testnet` development/deployment

* You can use our [Titan Testnet Faucet](https://titan-testnet-faucet.titanlab.io) to get testnet funds to your address,
* You can use the [Titan Testnet Explorer](https://testnet.tkxscan.io/Titan%20Testnet) to query your transactions and get more details,
* When you are using `titand` you have to specify the `testnet` rpc using the `node` flag `--node=https://titan-testnet-rpc.titanlab.io:443`
* Instead of using `titan_18889-1` as a `chainId` you should use `titan_18889-1` i.e the `chain-id` flag should now be `--chain-id="titan_18889-1"`
* You can use the [Titan Testnet Explorer](https://testnet.tkxscan.io/Titan%20Testnet/cosmwasm) to find information about the `codeId` of the uploaded smart contracts OR find your instantiated smart contract

You can read more on the `titand` and how to use it to query/send transactions against `testnet` .
