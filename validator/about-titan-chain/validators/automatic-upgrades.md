# 🤖 Automatic Upgrades

Cosmovisor is a process manager for Cosmos SDK applications, providing a mechanism for automatic node upgrades. It monitors the governance process for signals indicating that a software upgrade is required and automatically downloads and applies the necessary updates. This tool helps to facilitate smoother and more efficient network upgrades, thereby ensuring a high level of continuity and reliability of blockchain services.

## 1. Install Titand

[Install `titand` ](../developers/)

## 2. Install Cosmovisor

{% hint style="info" %}
To install Cosmovisor you also need to install Go ([guide](set-up-node/from-source.md#install-go))
{% endhint %}

Follow [the document of cosmos](https://docs.cosmos.network/main/build/tooling/cosmovisor) to install Cosmovisor

Ensure add go bin `/usr/local/go/bin` and your `GOPATH` to `PATH` environment variable. You will also want to add below path into $HOME/.profile to persistent.

```
# add go binary
export PATH=$PATH:/usr/local/go/bin
# add go package binary
export PATH=$PATH:$(go env GOPATH)/bin
```

## 3. Init and config Node

Follow [Joining Testnet](../developers/integrate-on-testnet.md) until step [Create service file](../developers/integrate-on-testnet.md#create-service-file).

## 4. Config Cosmovisor

1. Add environment variable to `~/.profile`

Correct `DAEMON_HOME` if you do not use default home location for node

```
# Setup Cosmovisor
export DAEMON_NAME=titand
export DAEMON_HOME=$HOME/.titand
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=true
```

2. Reload your `.profile`

```sh
source ~/.profile
```

3. Create folders for Cosmovisor

```sh
mkdir -p ~/.titand/cosmovisor
mkdir -p ~/.titand/cosmovisor/genesis
mkdir -p ~/.titand/cosmovisor/upgrades
```

4. Init Cosmvisor with `titand` binary

* If you install `titand` from source. Run follow command to init your genesis binary

```sh
cosmovisor init $(which titand)
```

* If you download prebuild binary `titand` from our github. Copy prebuild binary and lib into cosmovior

```sh
cp -r ~/titan/* ~/.titand/cosmovisor/genesis/
cosmovisor init ~/.titand/cosmovisor/genesis/bin/titand
```

After that your `.titand` folder will be like this (will not contain `lib` folder if you install from source)

```
.
├── config
│   ├── addrbook.json
│   ├── app.toml
│   ├── client.toml
│   ├── config.toml
│   ├── genesis.json
│   ├── node_key.json
│   └── priv_validator_key.json
├── cosmovisor
│   ├── current -> /Users/mac/.titand/cosmovisor/genesis
│   └── genesis
│       ├── LICENSE.md
│       ├── README.md
│       ├── bin
│       │   └── titand
│       └── lib
│           ├── libwasmvm.aarch64.so
│           ├── libwasmvm.dylib
│           └── libwasmvm.x86_64.so
├── data
│   └── priv_validator_state.json
└── wasm
    └── wasm
```

Verify your setup with command (ensure version is same as your `titand`)

```
cosmovisor run version --long
```

5. Update service file `/etc/systemd/system/titand.service` to use cosmovisor

Replace `<cosmovisor_absoule_path>` by output of command `which cosmovisor`.&#x20;

Replace `<home_directory>` by your home directory that contain folder `.titand` . Or other directory if you chose your own folder for titan node.

{% hint style="info" %}
By default, Cosmovisor will back up the current data before upgrading the chain process (this means your disk space should always have more than 40% free). If your node is configured to keep only a small amount of historical state, the backup process may be fast. However, if your node is a full node that stores a large amount of historical data, you may want to set UNSAFE\_SKIP\_BACKUP=true to help speed up the upgrade process and avoid consuming disk space.
{% endhint %}

```
[Unit]
Description=Titan service
After=network-online.target

[Service]
User=root
ExecStart=<cosmovisor_absoule_path> run start --x-crisis-skip-assert-invariants --home <home_directory>/.titand
Restart=always
RestartSec=6
LimitNOFILE=4096

Environment="DAEMON_HOME=<home_directory>/.titand"
Environment="DAEMON_NAME=titand"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=false"

[Install]
WantedBy=multi-user.target
```

6. Start service

```sh
systemctl daemon-reload
systemctl restart systemd-journald
systemctl enable titand.service
systemctl start titand.service
```

7. Check node status

```sh
cosmovisor run status
```

You should get something like this:

{% code overflow="wrap" %}
```sh
7:12AM INF running app args=["status"] module=cosmovisor path=/home/ubuntu/.titand/cosmovisor/genesis/bin/titand
{"NodeInfo":{"protocol_version":{"p2p":"8","block":"11","app":"0"},"id":"0e05f4f0c57ed26089e1d10dba6a1ac24e1eaa68","listen_addr":"tcp://0.0.0.0:26656","network":"titan_18889-1","version":"0.37.4","channels":"40202122233038606100","moniker":"Full","other":{"tx_index":"on","rpc_address":"tcp://0.0.0.0:26657"}},"SyncInfo":{"latest_block_hash":"2524BFFE965E2779C63A8F279090C765306194ED58E583C0CC51D2BB51AEFCD5","latest_app_hash":"48886A8524132719FEAC0250A777AE2D52BD6C118378722655514095E7756E5E","latest_block_height":"208339","latest_block_time":"2024-02-06T07:12:20.352972672Z","earliest_block_hash":"1E13F1C9B29A262BED941DEB6545D2081B563D81C55A570608C6AE955492CE3E","earliest_app_hash":"E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855","earliest_block_height":"1","earliest_block_time":"2024-01-24T00:00:00Z","catching_up":false},"ValidatorInfo":{"Address":"0BCCB16848B6C9DAA1E889CC1F43C2B9344B6636","PubKey":{"type":"tendermint/PubKeyEd25519","value":"0ujE587gKbF3YdFdsvLhbKsAPmWQSOF+BJVNj8tU+bg="},"VotingPower":"0"}}
```
{% endcode %}

If have any issue, you can check error log by

```sh
journalctl -fu titand
```
