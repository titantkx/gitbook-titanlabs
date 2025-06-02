---
description: >-
  This section helps node operators and validators to run, upgrade and maintain
  their validator nodes on Titan Chain.
---

# ⌨️ Set-up Node

To begin operating nodes either on Testnet or Mainnet, you must first:

| Build from source code                     | Build with prebuild binary                         |
| ------------------------------------------ | -------------------------------------------------- |
| [from-source.md](from-source.md "mention") | [prebuild-binary.md](prebuild-binary.md "mention") |

{% hint style="warning" %}
The following guides uses Ubuntu. You will need knowledge about how golang and linux shell.
{% endhint %}

***

### Upgrade Log

Refer to the following table for the most recent and prior chain releases.

#### Mainnet

| Version                                                             |  Breaking Changed | Manual Upgrade Only |  Height                                                     |
| ------------------------------------------------------------------- | ----------------- | ------------------- | ----------------------------------------------------------- |
| [**v2.0.1**](https://github.com/titantkx/titan/releases/tag/v2.0.1) | ❌                 | ❌                   | [1](https://titan-explorer-light.titanlab.io/Titan/block/1) |
| [**v3.0.0**](https://github.com/titantkx/titan/releases/tag/v3.0.0) | ✅                 | ❌                   | [4176482](https://tkxscan.io/Titan/gov/1)                   |

#### Testnet

| Version                                                                       |  Breaking Changed | Manual Upgrade Only |  Height                                                                                 |
| ----------------------------------------------------------------------------- | ----------------- | ------------------- | --------------------------------------------------------------------------------------- |
| [**v1.0.0**](https://github.com/titantkx/titan/releases/tag/v1.0.0)           | ❌                 | ❌                   | [1](https://titan-testnet-explorer-light.titanlab.io/Titan%20Testnet/block/1)           |
| [**v2.0.0**](https://github.com/titantkx/titan/releases/tag/v2.0.0)           | ✅                 | ❌                   | [727408](https://titan-testnet-explorer-light.titanlab.io/Titan%20Testnet/block/727408) |
| [**v2.0.1**](https://github.com/titantkx/titan/releases/tag/v2.0.1)           | ❌                 | ❌                   | [967678](https://titan-testnet-explorer-light.titanlab.io/Titan%20Testnet/block/967678) |
| [**v3.0.0-rc.0**](https://github.com/titantkx/titan/releases/tag/v3.0.0-rc.0) | ✅                 | ❌                   | [4416064](https://testnet.tkxscan.io/Titan%20Testnet/block/4416064)                     |

