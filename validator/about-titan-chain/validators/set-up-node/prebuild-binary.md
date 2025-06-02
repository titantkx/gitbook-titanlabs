---
description: You can also choose to use our prebuild binary.
---

# Prebuild binary

{% hint style="warning" %}
This guild uses Ubuntu. You will need knowledge about the linux shell.
{% endhint %}

Instead of installing `titand` from the source code, the validator can use our prebuild binary to use it instantly.

## System specification recommended

* For validator node (no index, no expose api, only keep 100 recent block):
  * 4 cores/threads
  * 8 GB ram
  * 512 GB SSD
* For api node/full node:
  * 4 cores/threads
  * more than 16 GB ram
  * 1 TB SSD

{% hint style="info" %}
This hardware specification for the startup node reflects the current state of the blockchain. Over time, these requirements will increase based on the amount of data stored in the chain and the transaction load.
{% endhint %}

## Download prebuild from github

Find the latest release at [https://github.com/titantkx/titan/releases](https://github.com/titantkx/titan/releases) and download a suitable version for your machine. Please choose correct version from [Broken link](broken-reference "mention")

Example: download v2.0.1 version for Linux with AMD processor

```sh
https://github.com/titantkx/titan/releases/download/v2.0.1/titan_2.0.1_Linux_amd64.tar.gz
```

Extract to the folder you wanted

```sh
mkdir ~/titan
tar -xzf titan_2.0.1_Darwin_amd64.tar.gz -C ~/titan
```

After extraction, the folder structure will be like this. It will contain binary `titand` and shared library `libwasmvm`

```sh
.
├── LICENSE.md
├── README.md
├── bin
│   └── titand
└── lib
    ├── libwasmvm.aarch64.so
    ├── libwasmvm.dylib
    └── libwasmvm.x86_64.so
```

At this point, you can easily run the program

```
~/titan/bin/titand version --long
```

You should get output like this

```
build_tags: netgo,ledger,cgo
commit: 
cosmos_sdk_version: v0.47.6
go: go version go1.20.6 darwin/amd64
name: titan
server_name: titand
version: 2.0.1
```
