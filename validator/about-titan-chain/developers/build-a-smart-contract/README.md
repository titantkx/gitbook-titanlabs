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

In our [sample counter contract](https://github.com/InjectiveLabs/cw-counter/blob/59b9fed82864103eb704a58d20ddb4bf94c69787/src/msg.rs), we will implement one `instantiate`, one `query`, and two `execute` methods.

#### Start with a Template <a href="#start-with-a-template" id="start-with-a-template"></a>

In your working directory, quickly launch your smart contract with the recommended folder structure and build options by running the following commands

```
cargo generate --git https://github.com/CosmWasm/cw-template.git --branch 1.0 --name my-first-contract
cd my-first-contract
```

This helps get you started by providing the basic boilerplate and structure for a smart contract. In the [`src/contract.rs`](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/src/contract.rs) file you will find that the standard CosmWasm entrypoints [`instantiate()`](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/src/contract.rs#L15), [`execute()`](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/src/contract.rs#L35), and [`query()`](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/src/contract.rs#L72) are properly exposed and hooked up.

#### State <a href="#state" id="state"></a>

You can learn more about CosmWasm State on their [documentation](https://book.cosmwasm.com/basics/state.html).

`State` handles the state of the database where smart contract data is stored and accessed.

The [starting template](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/src/state.rs) has the following basic state, a singleton struct `State` containing:

* `count`, a 32-bit integer with which `execute()` messages will interact by increasing or resetting it.
* `owner`, the sender `address` of the `MsgInstantiateContract`, which will determine if certain execution messages are permitted.

```
// src/state.rs
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};

use cosmwasm_std::Addr;
use cw_storage_plus::Item;

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct State {
    pub count: i32,
    pub owner: Addr,
}

pub const STATE: Item<State> = Item::new("state");
```

Titan smart contracts have the ability to keep persistent state through Titan's native LevelDB, a bytes-based key-value store. As such, any data you wish to persist should be assigned a unique key, which may be used to index and retrieve the data.

Data can only be persisted as raw bytes, so any notion of structure or data type must be expressed as a pair of serializing and deserializing functions. For instance, objects must be stored as bytes, so you must supply both the function that encodes the object into bytes to save it on the blockchain, as well as the function that decodes the bytes back into data types that your contract logic can understand. The choice of byte representation is up to you, so long as it provides a clean, bi-directional mapping.

Fortunately, CosmWasm provides utility crates, such as [`cosmwasm_storage`](https://crates.io/crates/cosmwasm-storage), which provides convenient high-level abstractions for data containers such as a "singleton" and "bucket", which automatically provide serialization and deserialization for commonly-used types, such as structs and Rust numbers. Additionally, the[`cw-storage-plus`](https://docs.cosmwasm.com/docs/smart-contracts/state/cw-plus/) crate can be used for a more efficient storage mechanism.

Notice how the `State` struct holds both `count` and `owner`. In addition, the `derive` attribute is applied to auto-implement some useful traits:

* `Serialize`: provides serialization
* `Deserialize`: provides deserialization
* `Clone`: makes the struct copyable
* `Debug`: enables the struct to be printed to string
* `PartialEq`: provides equality comparison
* `JsonSchema`: auto-generates a JSON schema

`Addr` refers to a human-readable Titan address prefixed with `inj`, e.g. `inj1clw20s2uxeyxtam6f7m84vgae92s9eh7vygagt`.

#### InstantiateMsg <a href="#instantiatemsg" id="instantiatemsg"></a>

You can learn more about CosmWasm [InstantiateMsg on their documentation](https://github.com/CosmWasm/docs/blob/archive/dev-academy/develop-smart-contract/01-intro.md#instantiatemsg)

The `InstantiateMsg` is provided to the contract when a user instantiates a contract on the blockchain through a `MsgInstantiateContract`. This provides the contract with its configuration as well as its initial state.

On the Titan blockchain, the uploading of a contract's code and the instantiation of a contract are regarded as separate events, unlike on Ethereum. This is to allow a small set of vetted contract archetypes to exist as multiple instances sharing the same base code, but be configured with different parameters (imagine one canonical ERC20, and multiple tokens that use its code).

**Example**

For your contract, the contract creator is expected to supply the initial state in a JSON message. We can see in the message definition below that the message holds one parameter `count`, which represents the initial count.

```
{
  "count": 100
}
```

**Message Definition**

```
// src/msg.rs

use schemars::JsonSchema;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct InstantiateMsg {
    pub count: i32,
}
```

**Contract Logic**

In `contract.rs`, you will define your first entry-point, `instantiate()`, or where the contract is instantiated and passed its `InstantiateMsg`. Extract the count from the message and set up your initial state where:

* `count` is assigned the count from the message
* `owner` is assigned to the sender of the `MsgInstantiateContract`

```
// src/contract.rs
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    deps: DepsMut,
    _env: Env,
    info: MessageInfo,
    msg: InstantiateMsg,
) -> Result<Response, ContractError> {
    let state = State {
        count: msg.count,
        owner: info.sender.clone(),
    };
    set_contract_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;
    STATE.save(deps.storage, &state)?;

    Ok(Response::new()
        .add_attribute("method", "instantiate")
        .add_attribute("owner", info.sender)
        .add_attribute("count", msg.count.to_string()))
}
```

#### ExecuteMsg <a href="#executemsg" id="executemsg"></a>

You can learn more about CosmWasm ExecuteMsg in their [documentation](https://book.cosmwasm.com/basics/execute.html).

The `ExecuteMsg` is a JSON message passed to the `execute()` function through a `MsgExecuteContract`. Unlike the `InstantiateMsg`, the `ExecuteMsg` can exist as several different types of messages to account for the different types of functions that a smart contract can expose to a user. The [`execute()` function](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/src/contract.rs#L35) demultiplexes these different types of messages to its appropriate message handler logic.

We have [two ExecuteMsg](https://github.com/InjectiveLabs/cw-counter/blob/59b9fed82864103eb704a58d20ddb4bf94c69787/src/msg.rs#L9): `Increment` and `Reset`.

* `Increment` has no input parameter and increases the value of count by 1.
* `Reset` takes a 32-bit integer as a parameter and resets the value of `count` to the input parameter.

**Example**

**Increment**

Any user can increment the current count by 1.

```
{
  "increment": {}
}
```

**Reset**

Only the owner can reset the count to a specific number. See Logic below for the implementation details.

```
{
  "reset": {
    "count": 5
  }
}
```

**Message Definition**

For `ExecuteMsg`, an `enum` can be used to multiplex over the different types of messages that your contract can understand. The `serde` attribute rewrites your attribute keys in snake case and lower case, so you'll have `increment` and `reset` instead of `Increment` and `Reset` when serializing and deserializing across JSON.

```
// src/msg.rs

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum ExecuteMsg {
    Increment {},
    Reset { count: i32 },
}
```

**Logic**

```
// src/contract.rs

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    _env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response, ContractError> {
    match msg {
        ExecuteMsg::Increment {} => try_increment(deps),
        ExecuteMsg::Reset { count } => try_reset(deps, info, count),
    }
}
```

This is your `execute()` method, which uses Rust's pattern matching to route the received `ExecuteMsg` to the appropriate handling logic, either dispatching a `try_increment()` or a `try_reset()` call depending on the message received.

```
pub fn try_increment(deps: DepsMut) -> Result<Response, ContractError> {
    STATE.update(deps.storage, |mut state| -> Result<_, ContractError> {
        state.count += 1;
        Ok(state)
    })?;

    Ok(Response::new().add_attribute("method", "try_increment"))
}
```

First, it acquires a mutable reference to the storage to update the item located at key `state`. It then updates the state's count by returning an `Ok` result with the new state. Finally, it terminates the contract's execution with an acknowledgement of success by returning an `Ok` result with the `Response`.

```
// src/contract.rs

pub fn try_reset(deps: DepsMut, info: MessageInfo, count: i32) -> Result<Response, ContractError> {
    STATE.update(deps.storage, |mut state| -> Result<_, ContractError> {
        if info.sender != state.owner {
            return Err(ContractError::Unauthorized {});
        }
        state.count = count;
        Ok(state)
    })?;
    Ok(Response::new().add_attribute("method", "reset"))
}
```

The logic for reset is very similar to increment—except this time, it first checks that the message sender is permitted to invoke the reset function (in this case, it must be the contract owner).

#### QueryMsg <a href="#querymsg" id="querymsg"></a>

You can learn more about CosmWasm [QueryMsg in their documentation](https://docs.cosmwasm.com/docs/smart-contracts/query)

The `GetCount` [query message](https://github.com/InjectiveLabs/cw-counter/blob/59b9fed82864103eb704a58d20ddb4bf94c69787/src/msg.rs#L16) has no parameters and returns the `count` value.

See the implementation details in Logic below.

**Example**

The template contract only supports one type of `QueryMsg`:

**GetCount**

The request:

```
{
  "get_count": {}
}
```

Which should return:

```
{
  "count": 5
}
```

**Message Definition**

To support data queries in the contract, you'll have to define both a `QueryMsg` format (which represents requests), as well as provide the structure of the query's output—`CountResponse` in this case. You must do this because `query()` will send information back to the user through structured JSON, so you must make the shape of your response known. See Generating JSON Schema for more info.

Add the following to your `src/msg.rs`:

```
// src/msg.rs
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    // GetCount returns the current count as a json-encoded number
    GetCount {},
}

// Define a custom struct for each query response
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct CountResponse {
    pub count: i32,
}
```

**Logic**

The logic for `query()` is similar to that of `execute()`; however, since `query()` is called without the end-user making a transaction, the `env` argument is omitted as no information is necessary.

```
// src/contract.rs

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn query(deps: Deps, _env: Env, msg: QueryMsg) -> StdResult<Binary> {
    match msg {
        QueryMsg::GetCount {} => to_binary(&query_count(deps)?),
    }
}

fn query_count(deps: Deps) -> StdResult<CountResponse> {
    let state = STATE.load(deps.storage)?;
    Ok(CountResponse { count: state.count })
}
```

#### Unit test <a href="#unit-test" id="unit-test"></a>

Unit tests should be run as the first line of assurance before deploying the code on chain. They are quick to execute and can provide helpful backtraces on failures with the `RUST_BACKTRACE=1` flag:

```
cargo unit-test // run this with RUST_BACKTRACE=1 for helpful backtraces
```

You can find the [unit test implementation](https://github.com/InjectiveLabs/cw-counter/blob/59b9fed82864103eb704a58d20ddb4bf94c69787/src/contract.rs#L88) at `src/contract.rs`

#### Building the Contract <a href="#building-the-contract" id="building-the-contract"></a>

Now that we understand and have tested the contract, we can run the following command to build the contract. This will check for any preliminary errors before we optimize the contract in the next step.

```
cargo wasm
```

Next, we must optimize the contract in order to ready the code for upload to the chain.

Read more details on [preparing the Wasm bytecode for production](https://github.com/InjectiveLabs/cw-counter/blob/59b9fed82864103eb704a58d20ddb4bf94c69787/Developing.md#preparing-the-wasm-bytecode-for-production)

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

#### Install `injectived` <a href="#install-injectived" id="install-injectived"></a>

`injectived` is the command-line interface and daemon that connects to Injective and enables you to interact with the Injective blockchain.

If you want to interact with your Smart Contract locally using CLI, you have to have `injectived` installed. To do so, you can follow the installation guidelines here [Install injectived](https://docs.injective.network/developers/cosmwasm-developers/your-first-smart-contract#install-injectived).

Alternatively, a Docker image has been prepared to make this tutorial easier.

If you install `injectived` from the binary, ignore the docker commands. On the [public endpoints section](https://docs.injective.network/nodes/public-endpoints) you can find the right --node info to interact with Mainnet and Testnet.

Executing this command will make the docker container execute indefinitely.

```
docker run --name="injective-core-staging" \
-v=<directory_to_which_you_cloned_cw-template>/artifacts:/var/artifacts \
--entrypoint=sh public.ecr.aws/l9h3g6c6/injective-core:staging \
-c "tail -F anything"
```

Note: `directory_to_which_you_cloned_cw-template` must be an absolute path. The absolute path can easily be found by running the `pwd` command from inside the CosmWasm/cw-counter directory.

Open a new terminal and go into the Docker container to initialize the chain:

```
docker exec -it injective-core-staging sh
```

Let’s start by adding `jq` dependency, which will be needed later on:

```
# inside the "injective-core-staging" container
apk add jq
```

Now we can proceed to local chain initialization and add a test user called `testuser` (when prompted use 12345678 as password). We will only use the test user to generate a new private key that will later on be used to sign messages on the testnet:

```
# inside the "injective-core-staging" container
injectived keys add testuser
```

**OUTPUT**

```
- name: testuser
  type: local
  address: inj1exjcp8pkvzqzsnwkzte87fmzhfftr99kd36jat
  pubkey: '{"@type":"/injective.crypto.v1beta1.ethsecp256k1.PubKey","key":"Aqi010PsKkFe9KwA45ajvrr53vfPy+5vgc3aHWWGdW6X"}'
  mnemonic: ""

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

wash wise evil buffalo fiction quantum planet dial grape slam title salt dry and some more words that should be here
```

Take a moment to write down the address or export it as an env variable, as you will need it to proceed:

```
# inside the "injective-core-staging" container
export INJ_ADDRESS= <your inj address>
```

You can request testnet funds for your recently generated test address using the [Injective test faucet](https://faucet.injective.network/).

Now you have successfully created `testuser` an Injective Testnet. It should also hold some funds after requesting `testnet` funds from the faucet.

To confirm, search for your address on the [Injective Testnet Explorer](https://testnet.explorer.injective.network/) to check your balance.

Alternatively, you can verify it by [querying the bank balance](https://sentry.testnet.lcd.injective.network/swagger/#/Query/AllBalances) or with curl:

```
curl -X GET "https://sentry.testnet.lcd.injective.network/cosmos/bank/v1beta1/balances/<your_INJ_address>" -H "accept: application/json"
```

### Upload the Wasm Contract <a href="#upload-the-wasm-contract" id="upload-the-wasm-contract"></a>

Now it's time to upload the `.wasm` file that you compiled in the previous steps to the Injective Testnet. Please note that the procedure for mainnet is different and [requires a governance proposal.](https://docs.injective.network/developers/cosmwasm-developers/guides/mainnet-deployment)

```
# inside the "injective-core-staging" container, or from the contract directory if running injectived locally
yes 12345678 | injectived tx wasm store artifacts/my_first_contract.wasm \
--from=$(echo $INJ_ADDRESS) \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj --gas=2000000 \
--node=https://testnet.sentry.tm.injective.network:443
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

* Look for the TxHash on the Injective Explorer [codes list](https://testnet.explorer.injective.network/codes/); it is most likely the most recent.
* Use `injectived` to query transaction info.

To query the transaction use the `txhash` and verify the contract was deployed.

```
injectived query tx 912458AA8E0D50A479C8CF0DD26196C49A65FCFBEEB67DF8A2EA22317B130E2C --node=https://testnet.sentry.tm.injective.network:443
```

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
      value: '"inj1h3gepa4tszh66ee67he53jzmprsqc2l9npq3ty"'
    type: cosmwasm.wasm.v1.EventCodeStored
  - attributes:
    - key: action
      value: /cosmwasm.wasm.v1.MsgStoreCode
    - key: module
      value: wasm
    - key: sender
      value: inj1h3gepa4tszh66ee67he53jzmprsqc2l9npq3ty
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

Take a minute to generate the schema ([take a look at it here](https://github.com/InjectiveLabs/cw-counter/blob/master/schema/cw-counter.json)) and get familiar with it, as you will need to it for the next steps.

### Instantiate the Contract <a href="#instantiate-the-contract" id="instantiate-the-contract"></a>

Now that we have the code on Injective, it is time to instantiate the contract to interact with it.

Reminder On CosmWasm, the upload of a contract's code and the instantiation of a contract are regarded as separate events

To instantiate the contract, run the following CLI command with the code\_id you got in the previous step, along with the [JSON encoded initialization arguments](https://github.com/InjectiveLabs/cw-counter/blob/ea3b781447a87f052e4b8308d5c73a30481ed61f/schema/cw-counter.json#L7) and a label (a human-readable name for this contract in lists).

```
INIT='{"count":99}'
yes 12345678 | injectived tx wasm instantiate $CODE_ID $INIT \
--label="CounterTestInstance" \
--from=$(echo $INJ_ADDRESS) \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj \
--gas=2000000 \
--no-admin \
--node=https://testnet.sentry.tm.injective.network:443
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

* Looking on the [Testnet Explorer](https://www.injscan.com/smart-contracts/)
* Querying the [ContractsByCode](https://k8s.testnet.lcd.injective.network/swagger/#/Query/ContractsByCode) and [ContractInfo](https://k8s.testnet.lcd.injective.network/swagger/#/Query/ContractInfo) APIs
* Querying through the CLI

```
injectived query wasm contract inj1ady3s7whq30l4fx8sj3x6muv5mx4dfdlcpv8n7 --node=https://testnet.sentry.tm.injective.network:443
```

#### Querying the Contract <a href="#querying-the-contract" id="querying-the-contract"></a>

As we know from earlier, the only QueryMsg we have is `get_count`.

```
GET_COUNT_QUERY='{"get_count":{}}'
injectived query wasm contract-state smart inj1ady3s7whq30l4fx8sj3x6muv5mx4dfdlcpv8n7 "$GET_COUNT_QUERY" \
--node=https://testnet.sentry.tm.injective.network:443 \
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
yes 12345678 | injectived tx wasm execute inj1ady3s7whq30l4fx8sj3x6muv5mx4dfdlcpv8n7 "$INCREMENT" --from=$(echo $INJ_ADDRESS) \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj --gas=2000000 \
--node=https://testnet.sentry.tm.injective.network:443 \
--output json
```

If we query the contract for the count, we see:

```
{"data":{"count":100}}
```

**yes 12345678 |** automatically pipes (passes) the passphrase to the input of **injectived tx wasm execute** so you do not need to enter it manually.

To reset the counter:

```
RESET='{"reset":{"count":999}}'
yes 12345678 | injectived tx wasm execute inj1ady3s7whq30l4fx8sj3x6muv5mx4dfdlcpv8n7 "$RESET" \
--from=$(echo $INJ_ADDRESS) \
--chain-id="injective-888" \
--yes --fees=1000000000000000inj --gas=2000000 \
--node=https://testnet.sentry.tm.injective.network:443 \
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

You may use [test-tube](https://github.com/InjectiveLabs/test-tube) for running integration tests that include a local Titan chain.

Congratulations! You've created and interacted with your first Titan smart contract and now know how to get started with CosmWasm development on Titan. Continue to Creating a Frontend for Your Contract for a guide on creating a web UI.
