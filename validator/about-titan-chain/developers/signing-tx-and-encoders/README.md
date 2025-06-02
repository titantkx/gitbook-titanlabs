---
description: >-
  The signing client is used to sign and broadcast transactions. Before using
  it, you need to add the appropriate encoders for the message types you want to
  use.
---

# 🖊️ Signing TX & Encoders

### Basic Usage

```
import { useChainWallet } from '@titan-kit/react';
import { MsgSend } from '@titanlabjs/titan-types/cosmos/bank/v1beta1/tx';
import { untitledWallet } from '@titan-kit/untitled-wallet';

function YourComponent() {
  const { signingClient } = useChainWallet(
    'titantestnet',  // chain name
    untitledWallet.info.name // wallet name from untitled wallet
  );

  // Add encoder
  signingClient.addEncoders([MsgSend]);

  // Now you can use the signing client to sign and broadcast transactions
  const handleTransaction = async () => {
    try {
      const res = await signingClient.signAndBroadcastSync(
        address,
        messages,
        fee,
        memo
      );
      console.log(res);
    } catch (error) {
      console.error(error);
    }
  };
}
```

### Send Transactions

```
import { useChainWallet } from '@titan-kit/react';
import { MsgSend } from '@titanlabjs/titan-types/cosmos/bank/v1beta1/tx';
import { MessageComposer } from '@titanlabjs/titan-types/cosmos/bank/v1beta1/tx.registry';
import { untitledWallet } from '@titan-kit/untitled-wallet';

function SendTokens() {
  const { signingClient } = useChainWallet(
    'titantestnet',
    untitledWallet.info.name
  );

  // Add encoder
  signingClient.addEncoders([MsgSend]);

  const handleSend = async () => {
    // Create message
    const message = MessageComposer.fromPartial.send({
      fromAddress: senderAddress,
      toAddress: recipientAddress,
      amount: [{
        denom: 'atkx',
        amount: '1000000'
      }]
    });

    // Sign and broadcast
    const res = await signingClient.signAndBroadcastSync(
      senderAddress,
      [message],
      fee,
      'Send tokens'
    );
  };
}
```

### Execute CosmWasm Contracts

```
import { useChainWallet } from '@titan-kit/react';
import { MsgExecuteContract } from '@titanlabjs/titan-types/cosmwasm/wasm/v1/tx';
import { MessageComposer } from '@titanlabjs/titan-types/cosmwasm/wasm/v1/tx.registry';
import { untitledWallet } from '@titan-kit/untitled-wallet';

function ExecuteContract() {
  const { signingClient } = useChainWallet(
    'titantestnet',
    untitledWallet.info.name
  );

  // Add encoder
  signingClient.addEncoders([MsgExecuteContract]);

  const handleExecute = async () => {
    // Create message
    const message = MessageComposer.fromPartial.executeContract({
      sender: senderAddress,
      contract: contractAddress,
      msg: Buffer.from(JSON.stringify({
        transfer: {
          recipient: recipientAddress,
          amount: '1000000'
        }
      })),
      funds: [{
        denom: 'atkx',
        amount: '1000000'
      }]
    });

    // Sign and broadcast
    const res = await signingClient.signAndBroadcastSync(
      senderAddress,
      [message],
      fee,
      'Execute contract'
    );
  };
} 
```







