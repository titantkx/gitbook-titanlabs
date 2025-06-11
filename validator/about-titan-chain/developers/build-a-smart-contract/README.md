---
description: >-
  Within this section, we'll explain how to setup your environment for CosmWasm
  Smart Contracts Development for deployment on Titan Chain.
---

# 👨‍💻 Build a Smart Contract

#### Prerequisites <a href="#prerequisites" id="prerequisites"></a>

Before starting, make sure you have [`rustup`](https://rustup.rs/) along with recent versions of `rustc` and `cargo` installed. Currently, we are testing on Rust v1.58.1+.

You also need to have the `wasm32-unknown-unknown` target installed as well as the `cargo-generate` Rust crate.

You can check versions via the following commands:

```
rustc --version
cargo --version
rustup target list --installed
# if wasm32 is not listed above, run this
rustup target add wasm32-unknown-unknown
# to install cargo-generate, run this
cargo install cargo-generate
```

#### Objectives <a href="#objectives" id="objectives"></a>

* Create and interact with a smart contract that increases and resets a counter to a given value
* Understand the basics of a CosmWasm smart contract, learn how to deploy it on Titan, and interact with it using Titan tools

#### CosmWasm Contract Basics <a href="#cosmwasm-contract-basics" id="cosmwasm-contract-basics"></a>

A smart contract can be considered an instance of a [singleton object](https://en.wikipedia.org/wiki/Singleton_pattern) whose internal state is persisted on the blockchain. Users can trigger state changes by sending the smart contract JSON messages, and users can also query its state by sending a request formatted as a JSON message. These JSON messages are different than Titan blockchain messages such as `MsgSend` and `MsgExecuteContract`.

As a smart contract writer, your job is to define 3 functions that compose your smart contract's interface:

* `instantiate()`: a constructor which is called during contract instantiation to provide initial state
* `execute()`: gets called when a user wants to invoke a method on the smart contract
* `query()`: gets called when a user wants to get data out of a smart contract

We will implement one `instantiate`, one `query`, and two `execute` methods.

In your working directory, quickly launch your smart contract with the recommended folder structure and build options by running the following commands

```
cargo generate --git https://github.com/CosmWasm/cw-template.git --branch 1.0 --name my-first-contract
cd my-first-contract
```

#### State <a href="#state" id="state"></a>

You can learn more about CosmWasm State on their [documentation](https://book.cosmwasm.com/basics/state.html).

`State` handles the state of the database where smart contract data is stored and accessed.

#### Building the Contract <a href="#building-the-contract" id="building-the-contract"></a>

Now that we understand and have tested the contract, we can run the following command to build the contract. This will check for any preliminary errors before we optimize the contract in the next step.

```
cargo wasm
```

Next, we must optimize the contract in order to ready the code for upload to the chain.

CosmWasm has [rust-optimizer](https://github.com/CosmWasm/rust-optimizer), an optimizing compiler that can produce a small and consistent build output. The easiest method to use the tool is to use a published Docker image—check [here](https://hub.docker.com/r/cosmwasm/rust-optimizer/tags) for the latest x86 version, or [here](https://hub.docker.com/r/cosmwasm/rust-optimizer-arm64/tags) for the latest ARM version. With Docker running, run the following command to mount the contract code to `/code` and optimize the output (you can use an absolute path instead of `$(pwd)` if you don't want to `cd` to the directory first):

```
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer:0.12.12
```

If you're on an ARM64 machine, you should use a docker image built for ARM64:

```
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer-arm64:0.12.12
```

CosmWasm does not recommend using the ARM64 version of the compiler because it produces different Wasm artifacts from the Intel/AMD version. For release/production, only contracts built with Intel/AMD optimizers are recommended for use. See [here](https://github.com/CosmWasm/rust-optimizer#notice) for the note from CosmWasm.

You may receive an `` Unable to update registry `crates-io` `` error while running the command. Try adding the following lines to the `Cargo.toml` file located within the contract directory and running the command again:

```
[net]
git-fetch-with-cli = true
```

See [The Cargo Book](https://doc.rust-lang.org/cargo/reference/config.html#netgit-fetch-with-cli) for more information.

This produces an `artifacts` directory with a `PROJECT_NAME.wasm`, as well as `checksums.txt`, containing the Sha256 hash of the Wasm file. The Wasm file is compiled deterministically (anyone else running the same docker on the same git commit should get the identical file with the same Sha256 hash).

#### Install `titand` <a href="#install-injectived" id="install-injectived"></a>

`titand` is the command-line interface and daemon that connects to titand and enables you to interact with the Titand blockchain.

If you want to interact with your Smart Contract locally using CLI, you have to have `titand` installed. To do so, you can follow the installation guidelines here [Install titand](../../validators/set-up-node/from-source.md#install-titan-binary).

Alternatively, a Docker image has been prepared to make this tutorial easier.

Executing this command will make the docker container execute indefinitely.

```
docker run --name="titan-core-staging" \
-v=<directory_to_which_you_cloned_cw-template>/artifacts:/var/artifacts \
--entrypoint=sh public.ecr.aws/l9h3g6c6/titan-core:staging \
-c "tail -F anything"
```

Note: `directory_to_which_you_cloned_cw-template` must be an absolute path. The absolute path can easily be found by running the `pwd` command from inside the CosmWasm/cw-counter directory.

Open a new terminal and go into the Docker container to initialize the chain:

```
docker exec -it titan-core-staging sh
```

Let’s start by adding `jq` dependency, which will be needed later on:

```
# inside the "titan-core-staging" container
apk add jq
```

Now we can proceed to local chain initialization and add a test user called `testuser` (when prompted use 12345678 as password). We will only use the test user to generate a new private key that will later on be used to sign messages on the testnet:

```
# inside the "titan-core-staging" container
titand keys add testuser
```

**OUTPUT**

```
- name: testuser
  type: local
  address: titan16nqhl5df9uyy899q3kq8nc2qndmpf2z4rqey0q
  pubkey: '{"@type":"/ethermint.crypto.v1.ethsecp256k1.PubKey","key":"A3UgyJ7AMzECx+jD/vJ694O+ZY2Kn1TB7w9STYsowvBJ"}'
  mnemonic: ""

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

wash wise evil buffalo fiction quantum planet dial grape slam title salt dry and some more words that should be here
```

Take a moment to write down the address or export it as an env variable, as you will need it to proceed:

```
# inside the "titan-core-staging" container
export TKX_ADDRESS= <your tkx address>
```

You can request testnet funds for your recently generated test address using the [Titan Testnet Faucet](https://titan-testnet-faucet.titanlab.io) .

Now you have successfully created `testuser` an Titan Testnet. It should also hold some funds after requesting `testnet` funds from the faucet.

To confirm, search for your address on the [Titan Testnet Explorer](https://testnet.tkxscan.io/Titan%20Testnet) to check your balance.

Alternatively, you can verify it by [querying the bank balance](https://titan-testnet-rpc.titanlab.io/) or with curl:

```
curl -X GET " https://titan-testnet-rpc.titanlab.io/cosmos/bank/v1beta1/balances/<your_TKX_address>" -H "accept: application/json"
```

### Upload the Wasm Contract <a href="#upload-the-wasm-contract" id="upload-the-wasm-contract"></a>

Now it's time to upload the `.wasm` file that you compiled in the previous steps to the Titan Testnet. Please note that the procedure for mainnet is different and [requires a governance proposal.](mainnet-deployment/)

```
# inside the "titan-core-staging" container, or from the contract directory if running titand locally
yes 12345678 | titand tx wasm store artifacts/my_first_contract.wasm \
--from=$(echo $TKX_ADDRESS) \
--chain-id="titan_18889-1" \
--yes --fees=1000000000000000atkx --gas=2000000 \
--node=https://titan-testnet-rpc.titanlab.io:443
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
txhash: 912458AA8E0D50A479C8CF0DD26196C49A65FCFBEEB67DF8A2EA22317B130E2C
```

Check your address on the Titan [Testnet Explorer](https://testnet.tkxscan.io/), and look for a transaction with the `txhash` returned from storing the code on chain. The transaction type should be `MsgStoreCode`.

There are different ways to find the code that you just stored:

* Look for the TxHash on the Titan Explorer [codes list](https://testnet.explorer.injective.network/codes/); it is most likely the most recent.
* Use `titand` to query transaction info.

To query the transaction use the `txhash` and verify the contract was deployed.

{% code fullWidth="false" %}
```
titand query tx 912458AA8E0D50A479C8CF0DD26196C49A65FCFBEEB67DF8A2EA22317B130E2C --node= https://titan-testnet-rpc.titanlab.io:443
```
{% endcode %}

Inspecting the output more closely, we can see the `code_id` of `290` for the uploaded contract:

```
- events:
  - attributes:
    - key: access_config
      value: '{"permission":"Everybody","address":""}'
    - key: checksum
      value: '"+OdoniOsDJ1T9EqP2YxobCCwFAqNdtYA4sVGv7undY0="'
    - key: code_id
      value: '"290"'
    - key: creator
      value: '"titan1h3gepa4tszh66ee67he53jzmprsqc2l9npq3ty"'
    type: cosmwasm.wasm.v1.EventCodeStored
  - attributes:
    - key: action
      value: /cosmwasm.wasm.v1.MsgStoreCode
    - key: module
      value: wasm
    - key: sender
      value: titan1h3gepa4tszh66ee67he53jzmprsqc2l9npq3ty
    type: message
  - attributes:
    - key: code_id
      value: "290"
    type: store_code
```

Let's export your `code_id` as an ENV variable—we'll need it to instantiate the contract. You can skip this step and manually add it later, but keep note of the ID.

```
export CODE_ID= <code_id of your stored contract>
```

### Generating JSON Schema <a href="#generating-json-schema" id="generating-json-schema"></a>

While the Wasm calls `instantiate`, `execute`, and `query` accept JSON, this is not enough information to use them. We need to expose the schema for the expected messages to the clients.

In order to make use of JSON-schema auto-generation, you should register each of the data structures that you need schemas for.

```
// examples/schema.rs

use std::env::current_dir;
use std::fs::create_dir_all;

use cosmwasm_schema::{export_schema, remove_schemas, schema_for};

use my_first_contract::msg::{CountResponse, HandleMsg, InitMsg, QueryMsg};
use my_first_contract::state::State;

fn main() {
    let mut out_dir = current_dir().unwrap();
    out_dir.push("schema");
    create_dir_all(&out_dir).unwrap();
    remove_schemas(&out_dir).unwrap();

    export_schema(&schema_for!(InstantiateMsg), &out_dir);
    export_schema(&schema_for!(ExecuteMsg), &out_dir);
    export_schema(&schema_for!(QueryMsg), &out_dir);
    export_schema(&schema_for!(State), &out_dir);
    export_schema(&schema_for!(CountResponse), &out_dir);
}
```

The schemas can then be generated with

```
cargo schema
```

which will output 5 files in `./schema`, corresponding to the 3 message types the contract accepts, the query response message, and the internal `State`.

These files are in standard JSON Schema format, which should be usable by various client side tools, either to auto-generate codecs, or just to validate incoming JSON with respect to the defined schema.

Take a minute to generate the schema and get familiar with it, as you will need to it for the next steps.

### Instantiate the Contract <a href="#instantiate-the-contract" id="instantiate-the-contract"></a>

Now that we have the code on Titan, it is time to instantiate the contract to interact with it.

Reminder On CosmWasm, the upload of a contract's code and the instantiation of a contract are regarded as separate events

To instantiate the contract, run the following CLI command with the code\_id you got in the previous step, along with the JSON encoded initialization arguments and a label (a human-readable name for this contract in lists).

```
INIT='{"count":99}'
yes 12345678 | titand tx wasm instantiate $CODE_ID $INIT \
--label="CounterTestInstance" \
--from=$(echo $TKX_ADDRESS) \
--chain-id="titan_18889-1" \
--yes --fees=1000000000000000tkx \
--gas=2000000 \
--no-admin \
--node=https://titan-testnet-rpc.titanlab.io:443
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
txhash: 01804F525FE336A5502E3C84C7AE00269C7E0B3DC9AA1AB0DDE3BA62CF93BE1D
```

You can find the contract address and metadata by:

* Looking on the [Testnet Explorer](https://testnet.tkxscan.io/Titan%20Testnet/cosmwasm)
* Querying the [ContractsByCode](https://titan-testnet-lcd-1.titanlab.io/#/Query/CosmwasmWasmV1ContractsByCode) and [ContractInfo](https://titan-testnet-lcd-1.titanlab.io/#/Query/CosmwasmWasmV1ContractInfo) APIs
* Querying through the CLI

{% code fullWidth="false" %}
```
CONTRACT=$(titand query wasm list-contract-by-code "44" --output json | jq -r '.contracts[-1]')
titand query wasm contract $CONTRACT --node=https://titan-testnet-rpc.titanlab.io:443
```
{% endcode %}

#### Querying the Contract <a href="#querying-the-contract" id="querying-the-contract"></a>

As we know from earlier, the only QueryMsg we have is `get_count`.

```
GET_COUNT_QUERY='{"get_count":{}}'
titand query wasm contract-state smart $CONTRACT "$GET_COUNT_QUERY" \
--node=https://titan-testnet-rpc.titanlab.io:443 \
--output json
```

**Output:**

```
{"data":{"count":99}}
```

We see that `count` is 99, as set when we instantiated the contract.

If you query the same contract, you may receive a different response as others may have interacted with the contract and incremented or reset the count.

#### Execute the Contract <a href="#execute-the-contract" id="execute-the-contract"></a>

Let's now interact with the contract by incrementing the counter.

```
INCREMENT='{"increment":{}}'
yes 12345678 | titand tx wasm execute $CONTRACT "$INCREMENT" --from=$(echo $TKX_ADDRESS) \
--chain-id="titan_18889-1" \
--yes --fees=1000000000000000atkx --gas=2000000 \
--node=https://titan-testnet-rpc.titanlab.io:443 \
--output json
```

If we query the contract for the count, we see:

```
{"data":{"count":100}}
```

**yes 12345678 |** automatically pipes (passes) the passphrase to the input of **titand tx wasm execute** so you do not need to enter it manually.

To reset the counter:

```
RESET='{"reset":{"count":999}}'
yes 12345678 | titand tx wasm execute $CONTRACT "$RESET" \
--from=$(echo $TKX_ADDRESS) \
--chain-id="titan_18889-1" \
--yes --fees=1000000000000000atkx --gas=2000000 \
--node=https://titan-testnet-rpc.titanlab.io:443 \
--output json
```

Now, if we query the contract again, we see the count has been reset to the provided value:

```
{"data":{"count":999}}
```

### Cosmos Messages <a href="#cosmos-messages" id="cosmos-messages"></a>

In addition to defining custom smart contract logic, CosmWasm also allows contracts to interact with the underlying Cosmos SDK functionalities. One common use case is to send tokens from the contract to a specified address using the Cosmos SDK's bank module.

#### Example: Bank Send <a href="#example-bank-send" id="example-bank-send"></a>

The `BankMsg::Send` message allows the contract to transfer tokens to another address. This can be useful in various scenarios, such as distributing rewards or returning funds to users.

**Note:** If you want to send funds and execute a function on another contract at the same time, don't use BankMsg::Send. Instead, use WasmMsg::Execute and set the respective funds field.

#### Constructing the Message <a href="#constructing-the-message" id="constructing-the-message"></a>

You can construct a `BankMsg::Send` message within your contract's `execute` function. This message requires specifying the recipient address and the amount to send. Here's an example of how to construct this message:

```
use cosmwasm_std::{BankMsg, Coin, Response, MessageInfo};

pub fn try_send(
    info: MessageInfo,
    recipient_address: String,
    amount: Vec<Coin>,
) -> Result<Response, ContractError> {
    let send_message = BankMsg::Send {
        to_address: recipient_address,
        amount,
    };

    let response = Response::new().add_message(send_message);
    Ok(response)
}
```

#### Usage in a Smart Contract <a href="#usage-in-a-smart-contract" id="usage-in-a-smart-contract"></a>

In your contract, you can add a new variant to your ExecuteMsg enum to handle this bank send functionality. For example:

```
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum ExecuteMsg {
    // ... other messages ...
    SendTokens { recipient: String, amount: Vec<Coin> },
}
```

Then, in the `execute` function, you can add a case to handle this message:

```
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response, ContractError> {
    match msg {
        // ... other message handling ...
        ExecuteMsg::SendTokens { recipient, amount } => try_send(info, recipient, amount),
    }
}
```

### Testing <a href="#testing" id="testing"></a>

As with other smart contract functions, you should add unit tests to ensure your bank send functionality works as expected. This includes testing different scenarios, such as sending various token amounts and handling errors correctly.

You may use test-tube for running integration tests that include a local Titan chain.

Congratulations! You've created and interacted with your first Titan smart contract and now know how to get started with CosmWasm development on Titan. Continue to Creating a Frontend for Your Contract for a guide on creating a web UI.
