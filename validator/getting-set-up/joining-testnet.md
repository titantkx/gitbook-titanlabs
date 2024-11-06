# 🤼‍♂️ Joining Testnet

{% hint style="warning" %}
Make sure you completed the [Installation](installation/from-source.md) step before following this session. You will also need knowledge about linux service file (systemd).
{% endhint %}

* Current Version: v2.0.1
* Chain ID: titan\_18889-1
* Denom: atkx, tkx (1e18 atkx)
* Endpoint: [testnet.md](../testnet.md "mention")
* Faucet: [https://titan-testnet-faucet.titanlab.io](https://titan-testnet-faucet.titanlab.io)
* Explorer: [https://testnet.tkxscan.io](https://testnet.tkxscan.io/)

## Information

The Titan testnet is currently running **titand** `v2.0.1`. Visit the [testnet explorer](https://testnet.tkxscan.io) to view real-time on-chain activity

We recommend running public testnet nodes on a machine with at least two cores, 4GB of RAM, and 50GB of disk space.

## Installation & Configuration

### Init config file and data

```sh
titand init <custom_moniker> --chain-id titan_18889-1
```

Now, a folder will be created by default in your HOME directory as `$HOME/.titand`. This folder will store all the configuration and data files of the node during operation.

### Retrieve the genesis file of the Titan Chain testnet and store it in the config directory.

```sh
cd $HOME
curl -LO https://github.com/titantkx/titan-testnets/raw/main/public/genesis.json.gz
gzip -d genesis.json.gz
mv genesis.json $HOME/.titand/config/genesis.json
```

### Config your node.

#### Set peers

<pre class="language-sh"><code class="lang-sh">cd $HOME/.titand/config
<strong>sed -i 's/seeds =.*/seeds = "1f61a190809e4413079174b6236bc00a502722b6@titan-testnet-node-1.titanlab.io:26656,c580270d0741f08d8ed88eda5d7de272622e7c02@titan-testnet-node-2.titanlab.io:26656,acb90d29636059abd5c4ca36f3731a69de73cf5b@titan-testnet-seed-1.titanlab.io:26656"/' config.toml
</strong></code></pre>

#### Config external address

You will need to configure `external_address` in file `config.toml` is the external address of your node. Correct this config will allow other nodes in P2P network connect to your node.

#### Index:

{% hint style="info" %}
In default, we disable all index and only keep 100 recent blocks to optimize the diskspace used by node.
{% endhint %}

If you want to enable index, you need to change config in file `app.toml` and `config.toml`

* Ethereum transaction  (in `app.toml`)

```toml
[json-rpc]

# EnableIndexer enables the custom transaction indexer for the EVM (ethereum transactions).
enable-indexer = true
```

* TX index, for query tx (in `config.toml`)

```toml
[tx_index]

# What indexer to use for transactions
#
# The application will set which txs to index. In some cases a node operator will be able
# to decide which txs to index based on configuration set in the application.
#
# Options:
#   1) "null"
#   2) "kv" (default) - the simplest possible indexer, backed by key-value storage (defaults to levelDB; see DBBackend).
# 		- When "kv" is chosen "tx.height" and "tx.hash" will always be indexed.
#   3) "psql" - the indexer services backed by PostgreSQL.
# When "kv" or "psql" is chosen "tx.height" and "tx.hash" will always be indexed.
indexer = "kv"
```

### Config state sync

{% hint style="warning" %}
If you use the latest binary version, this is required to sync data from blockchain. If you want to sync data from the genesis block, you must start syncing from genesis binary and follow [every upgrade in history](../upgrade/upgrade-list.md#testnet).
{% endhint %}

Without this, your node will have to synchronize data from the genesis, which could take a considerable amount of time. Instead, we will search for snapshots from trusted peers at a specific height and then verify a minimal set of snapshot chunks against the network.

Get the block height and block hash. First, you need to access the [explorer website](https://testnet.tkxscan.io) to obtain the nearest block height and its corresponding hash. We configure a node to take a snapshot every 1000 blocks and keep the 10 most recent snapshots. Therefore, the snapshot corresponding to the nearest block height will have the highest thousand value. For example, at the time of writing this guide testnet block height is `986922` so the nearest block height will be `986000`. Enter the block height in the search bar at the top of the website to retrieve its hash. The hash of Block `986000` is:&#x20;

```
BA8F33B927BE01087C66257EAEE302C3940F7C619A18F72C0EB7789A009B6839
```

After having block height and block hash run the following command in order (replace `<BLOCK_HEIGHT>` and  `<BLOCK_HASH>` by the value you found in the above step)

{% hint style="info" %}
`rpc_servers` need at least 2 servers.
{% endhint %}

```sh
cd $HOME/.titand/config
sed -i 's/enable =.*/enable = true/' config.toml
sed -i 's/trust_height =.*/trust_height = <BLOCK_HEIGHT>/' config.toml
sed -i 's/trust_hash =.*/trust_hash = "<BLOCK_HASH>"/' config.toml
sed -i 's/rpc_servers =.*/rpc_servers = "https:\/\/titan-testnet-rpc-1.titanlab.io:443,https:\/\/titan-testnet-rpc-2.titanlab.io:443"/' config.toml
```

### Create Service File

{% hint style="warning" %}
To make the upgrade process as convenient as possible, instead of using `titand` binary directly we recommend you follow [this guide](../upgrade/automatic-upgrades.md) to use [`cosmovisor`](https://docs.cosmos.network/main/build/tooling/cosmovisor)
{% endhint %}

Get your absolute path of `titand`  binary and your HOME directory that contains `.titan` folder

```sh
which titand
echo $HOME
```

Create a service file at `/etc/systemd/system/titand.service`. Replace `<titand_absolute_path>` and `<home_directory>` to your.

{% hint style="info" %}
To find out your `<titand_absolute_path>` you can use command `which titand`. And your `<home_directory>` by command `echo $HOME`
{% endhint %}

```
[Unit]
Description=Titan service
After=network-online.target

[Service]
User=root
ExecStart=<titand_absolute_path> start --x-crisis-skip-assert-invariants --home <home_directory>/.titand
Restart=always
RestartSec=6
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

### Start the Service

<pre class="language-sh"><code class="lang-sh">systemctl daemon-reload
<strong>systemctl restart systemd-journald
</strong>systemctl enable titand.service
systemctl start titand.service
</code></pre>

Now that your node has started up and the synchronization process has begun, you can check the sync status of the node using the following command. If the node is syncing, it will show `true`. Please wait until it show `false`, which means your node has caught up with the network and is ready for querying.

```sh
titand status | grep -o '"catching_up":[^,}]*'
```

If have any issue, you can check error log by

```sh
journalctl -fu titand
```
