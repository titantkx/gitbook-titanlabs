---
description: >-
  This guide will explain how to build and install the Titand binary from source
  code. Once installed on the server, you can now interact with the Titan Chain
  testnet as a Full node or a Validator.
---

# From source

{% hint style="warning" %}
This guide uses Ubuntu. You will need knowledge about how golang and linux shell.
{% endhint %}

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

## Build Requirements

At present, we support installation on Linux distributions.

1. Update and install build tools
2. Install Go
3. Clone the source code of the Titan node, then proceed to build and install it.

## Build Tools

Install make and gcc

```
sudo apt-get update
sudo apt-get install -y make gcc
```

## Install Go

You can follow [Go official docs](https://go.dev/doc/install) to download and install Go.&#x20;

Or follow our guide below to install Go in Ubuntu (at time of writing Go version is 1.21.0):

{% code lineNumbers="true" %}
```sh
# Download tarball into current folder
curl -OL https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
# Extract it to /usr/local
sudo tar -xvf go1.21.0.linux-amd64.tar.gz -C /usr/local
```
{% endcode %}

Add /usr/local/go/bin to the PATH environment variable. **Note:** This change only applies to the current shell session. If you want it to become persistent, you can add this line into $HOME/.profile.

```sh
export PATH=$PATH:/usr/local/go/bin
```

Add your `GOPATH` to `PATH` environment variable. You will also want to add below path into $HOME/.profile to persistent. `titand` binary will be installed into GOPATH

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

For now, you can check the current go version by command

```sh
go version
```

## Install Git

```sh
sudo apt-get install git-all
```

## Install Titan binary

Let's clone Titan source code from [github](https://github.com/tokenize-titan/titan). Please choose correct version tag via [Broken link](broken-reference "mention").

```sh
git clone -b <version_tag> https://github.com/titantkx/titan.git
```

Build and install titand binary

```
cd titan && make install
```

Now verify everything installed successfully by running

```
titand version --long
```

You should get output like this:

```
build_tags: netgo ledger cgo,
commit: 
cosmos_sdk_version: v0.47.6
go: go version go1.21.0 linux/amd64
name: titan
server_name: titand
version: 2.0.1
```
