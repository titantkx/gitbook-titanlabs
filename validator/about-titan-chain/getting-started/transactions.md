# Transactions

When users want to interact with Titan Chain and make state changes, they create transactions. Once the transaction is created, it requires a signature from the private key linked to the account initiating the particular state change. Following the signature, the transaction is broadcasted to Titan Chain.

After being broadcasted and passing all validations (including signature validation, values validations, etc.), the transaction is included in a block that undergoes network approval via the consensus process.

### Messages <a href="#messages" id="messages"></a>

In simpler terms, messages are the instructions given to TItan Chain about the desired state change. Messages are module-specific objects that trigger state transitions within the scope of the module they belong to. Every transaction must have at least one message.

**Additionally, multiple messages can be packed within the same transaction.** Available Messages from each module can be found in the [Modules](../developers/modules/) section.

### Transaction Context <a href="#transaction-context" id="transaction-context"></a>

Besides Messages, every transaction has a context. The context includes `fee`, `accountDetails`, `memo`, `signatures`, etc.

### Transaction Flow <a href="#transaction-flow" id="transaction-flow"></a>

Every transaction we want to broadcast to Titan Chain has the same flow. The flow consists of three steps: preparing, signing, and broadcasting the transaction. When the transaction is included in a block, the state change that was specified using the Message is applied on Titan Chain.

### Gas and Fees <a href="#gas-and-fees" id="gas-and-fees"></a>

Learn about the differences between `Gas` and `Fees` on Titan Chain.

Pre-requisite Readings -> [Cosmos SDK Gas](https://docs.cosmos.network/main/build/modules/auth#gas--fees)

Gas represents the amount of computational effort required to execute specific operations on the state machine.

Titan Chain utilizes the concept of gas to track the resource usage of operations during execution. Operations on Injective are represented as read or writes done to the chain's store.

A fee is calculated and charged to the user during a message execution. This fee is calculated from the sum of all gas consumed in a message execution:

```
fee = gas * gas price
```

Gas is used to make sure that operations do not require an excess amount of computational power to complete, and to deter bad-acting users from spamming the network.

**Minimum gas price:** The minimum gas price set by validators is currently `160,000,000tkx`. To find the amount paid in `tkx`, multiply the gas price by the gas amount and divide by 1e18 (TKX has 18 decimals).

**For example:** if `gasWanted` is 104,519, then `gasFees` = 160,000,000 \* 104,519 / 1e18 = 0.000016723`tkx`&#x20;

#### Cosmos SDK `Gas` <a href="#cosmos-sdk-gas" id="cosmos-sdk-gas"></a>

In the Cosmos SDK, gas is tracked in the main `GasMeter` and the `BlockGasMeter`:

* `GasMeter`: keeps track of the gas consumed during executions that lead to state transitions. It is reset on every transaction execution.
* `BlockGasMeter`: keeps track of the gas consumed in a block and enforces that the gas does not go over a predefined limit. This limit is defined in the Tendermint consensus parameters and can be changed via governance parameter change proposals.

More information regarding gas in Cosmos SDK can be found [here](https://docs.cosmos.network/main/learn/beginner/gas-fees).

In Cosmos, there are types of operations that are not triggered by transactions that can also result in state transitions. Concrete examples are the `BeginBlock` and `EndBlock` operations and the `AnteHandler` checks, which might also read and write to the store before running the state transition from a transaction.

**`BeginBlock` and `EndBlock`**

These operations are defined by the Tendermint Core's Application Blockchain Interface (ABCI) and are defined by each Cosmos SDK module. As their name suggest, they are executed at the beginning and at the end of each block processing respectively (i.e., pre- and post-transaction execution).

**`AnteHandler`**

The Cosmos SDK [`AnteHandler`](https://docs.cosmos.network/v0.45/modules/auth/03_antehandlers.html) performs basic checks prior to transaction execution. These checks are usually signature verification, transaction field validation, transaction fees, etc.
