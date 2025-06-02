---
description: >-
  Please find the relevant message types you may require for wallet integration
  below. The Module list is regularly updated.
---

# Wallet Module

#### Bank Module <a href="#bank-module" id="bank-module"></a>

```
import { 
  MsgSend,
  MsgMultiSend,
  MsgSetSendEnabled
} from '@titanlabjs/titan-types/cosmos/bank/v1beta1/tx';
```

#### Staking Module <a href="#staking-module" id="staking-module"></a>

```
import {
  MsgDelegate,
  MsgUndelegate,
  MsgBeginRedelegate,
  MsgCancelUnbondingDelegation
} from '@titanlabjs/titan-types/cosmos/staking/v1beta1/tx';
```

#### Distribution Module <a href="#distribution-module" id="distribution-module"></a>

```
import {
  MsgWithdrawDelegatorReward,
  MsgWithdrawValidatorCommission,
  MsgSetWithdrawAddress,
  MsgFundCommunityPool
} from '@titanlabjs/titan-types/cosmos/distribution/v1beta1/tx';
```

#### Governance Module <a href="#governance-module" id="governance-module"></a>

```
import {
  MsgSubmitProposal,
  MsgVote,
  MsgVoteWeighted,
  MsgDeposit
} from '@titanlabjs/titan-types/cosmos/gov/v1beta1/tx';
```

#### CosmWasm Module <a href="#cosmwasm-module" id="cosmwasm-module"></a>

```
import {
  MsgExecuteContract,
  MsgInstantiateContract,
  MsgStoreCode,
  MsgMigrateContract,
  MsgUpdateAdmin,
  MsgClearAdmin
} from '@titanlabjs/titan-types/cosmwasm/wasm/v1/tx';
```
