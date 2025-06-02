---
description: >-
  This section explains how you can operate a Mainnet validator node on Titan
  Chain.
---

# 🚔 Become a Mainnet Validator

## Prepare your account

* Create an account by [Keplr](https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap)
* Go to the [Explorer website](https://tkxscan.io) click `Sign in` button and use Keplr method. It then help add titan testnet network into Keplr
* **NOTE: because min self delegate is 2048 TKX, to order to become validator you need have at least 2048 TKX**

## Create your validator

After creating a wallet with Keplr, you will now have a mnemonic (12 words/24 words) and a passphrase. We will proceed to import it into the node you created at step [integrate-on-mainnet.md](../../developers/integrate-on-mainnet.md "mention").

Access to your node and import your wallet

```sh
titand keys add my_wallet --recover
```

Enter your 12-word mnemonic and passphrase. The output will appear as follows:

```sh
- address: titan198knyvkecazg4snsa5gt08ygg46fv7ma2g33v2
  name: my_wallet
  pubkey: '{"@type":"/ethermint.crypto.v1.ethsecp256k1.PubKey","key":"AkwSg37hZTtEODSYiMR5GGx/KCMHZymmguLKgOWeFCdc"}'
  type: local
```

**NOTE: Make sure that the address matches the one displayed in Keplr.**

### Now it is time to create the validator

Feel free to change parameters as yours will, or remove unnecessary infos (like, details, identity)

```
titand tx staking create-validator \
  --amount=2048tkx \
  --pubkey=$(titand tendermint show-validator) \
  --moniker="choose a moniker" \
  --website="Your website" \
  --details="Yours very detail" \
  --security-contact="your_email@email.com" \
  --identity="your id on keybase"
  --chain-id=titan_18888-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --gas="auto" \
  --gas-adjustment=1.5 \
  --gas-prices="101000000000atkx" \
  --min-self-delegation="2048000000000000000000" \
  --from=my_wallet
```

You will be asked for the passphrase again and confirm the transaction.

```
gas estimate: 263922
auth_info:
  fee:
    amount:
    - amount: "263922000000000"
      denom: atkx
    gas_limit: "263922"
    granter: ""
    payer: ""
  signer_infos: []
  tip: null
body:
  extension_options: []
  memo: ""
  messages:
  - '@type': /cosmos.staking.v1beta1.MsgCreateValidator
    commission:
      max_change_rate: "0.010000000000000000"
      max_rate: "0.200000000000000000"
      rate: "0.100000000000000000"
    delegator_address: titan1graakezx2rllgd53yzn2z30cukxrga2h7397sz
    description:
      details: ""
      identity: ""
      moniker: Beast
      security_contact: ""
      website: ""
    min_self_delegation: "2048000000000000000000"
    pubkey:
      '@type': /cosmos.crypto.ed25519.PubKey
      key: bug7PM4m8iefXf/g4JVdklDJmDvzsmchCvEgkfMlCrs=
    validator_address: titanvaloper1graakezx2rllgd53yzn2z30cukxrga2hq9qmgn
    value:
      amount: "2048000000000000000000"
      denom: atkx
  non_critical_extension_options: []
  timeout_height: "0"
signatures: []
confirm transaction before signing and broadcasting [y/N]: y
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
txhash: F348DF6D236C310A8D65234F73D51F3D84DE41DE511C417807A06407FB04A892
```

Check the status of your transaction and your new validator at [Explorer](https://tkxscan.io).



**NOTE: after success create your validator node, you may want to remove yours private key from machine for security**

```
titand keys delete my_wallet
```
