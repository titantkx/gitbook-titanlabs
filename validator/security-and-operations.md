# 🔐 Security and Operations

## Node liveness

The validator node must be kept running continuously, connect to other nodes and submit a vote for each block. If not validator will be slashed from staked balance and will be jailed (currently is 0.01% staked TKX if offline in around 14h)

#### Monitoring

Cometbft can export prometheus metrics to monitor node status ([https://docs.cometbft.com/v0.37/core/metrics](https://docs.cometbft.com/v0.37/core/metrics)). You should use it belong with [node exporter](https://github.com/prometheus/node\_exporter) to monitor node status.

#### Chain upgrade

Every chain upgrade will require the validator download new `titand` binary to replace the old one, some upgrades may require manual action like migrating config file or genesis (if that is a fork upgrade).&#x20;

You should always follow [titand github repo](https://github.com/titantkx/titan) for the latest release version. Main upgrades always be decided by [government proposal](https://tkxscan.io/Titan/gov).

For automatic handle upgrade please follow this [guide](upgrade/automatic-upgrades.md).

## Government responsibility

Titan chain has configuration factors that are determined by the government. Anyone can propose updates on configuration parameters (including chain upgrades). If the proposal is approved through government voting, that change will be implemented on the chain. Therefore, all validators have the right and responsibility to vote on every proposal and determine how the chain functions. [Gov page](https://tkxscan.io/Titan/gov).

## Sensitive information

After setting up your validator node, have some key files you need to keep secret and safe:

* `.titand/config/priv_validator_key.json` : This file contains the key that the validator node used to sign the block.. If this file is damaged or lost, your validator node may no longer sign new blocks, resulting in a loss of block rewards and potential jailing for missing signatures in too many blocks (being offline for too long).
* `.titand/config/node_key.json` : Not critical as `priv_validator_key` but this file contains identity information of your validator node, helping other node identify your node in P2P network.
* `.titand/data/priv_validator_state.json` : (updated every block) this file contains latest information about the singing process of your validator node, this helps your node prevent duplicate sign blocks (that may make slash large amounts of TKX). Move this file along with the 2 files above when transferring your node to a new machine or during a chain upgrade.

