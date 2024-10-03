# polkadot_dezentralised_nodes
# Introduction

Decentralized Nodes (the “Programme") is an initiative by the Web 3.0 Technologies Foundation (the “Foundation”). The purpose of the Programme is to utilise tokens held by the Foundation to nominate a validator operator (the “Validator Operator”) within the community. A Validator Operator shall be understood as an individual or business entity responsible for validating transactions and maintaining the integrity of the network, together with an underlying infrastructure (the “Validator Node”).

The primary objective of the Programme is to provide a structured on-ramp for the Validator Operator to join the active set on Kusama and/or Polkadot networks and further decentralising the validator active set. These terms and conditions (the “Agreement”), shall constitute a binding agreement between the Validator Operator and the Foundation in relation to the Programme.


# Requirements

- Familiarize yourself with the general Terms and Conditions and Rules criteria.
- It is necessary to complete KYC or KYB. **Important** - check the sanction lists in the requirements. Russia and some other countries are prohibited.
- Early participants in 1KV will be given an advantage in the selection for the first cohort.
- Launch a node and configure the validator.
- Fill out the form very carefully. The email in the form must match the one used for KYC.
- Identify the main wallet that will act as the stash. At a minimum, you must provide an email and Matrix handle. Contact [hello@polkassembly.io](mailto:hello@polkassembly.io) to complete the verification of your Matrix descriptor.

### Eligibility
- A self-bond of **7500 DOT** for Polkadot is required.
- A validator can charge a commission of up to **5%** on Polkadot.
- Validator rewards must be set as "staked" (increasing their own stake).
- Nodes must connect to public telemetry [https://telemetry.polkadot.io/](https://telemetry.polkadot.io/).
- No data should be hidden from telemetry. Specifically, nodes must at least exchange the following information: (CPU, RAM, number of cores, and whether the machine is virtual).
- Servers must meet the minimum hardware requirements described [here](#).
- Each node must run on a separate server. Exceptions may be made, but they are not recommended and must be justified.
- Validators should never be slashed. The only exception is if slashing occurs due to a client error or a general network problem.
- Nodes must be updated to the latest client version within **24 hours** of its release.
- Validators must pay staking rewards no later than **24 hours** after the end of the era.
- **Hetzner and Contabo cannot be used.**
- Validators must be available for a call (in English) upon request, during which they may be asked questions or asked to provide evidence of their operations.
- If a participant decides to leave the program, they must notify the Web3 Foundation by email at [validators@web3.foundation](mailto:validators@web3.foundation) at least one week before disconnecting their nodes.

### Ports Used

#### For the first node
```
--port 30333
--rpc-port 9933
--prometheus-port 9615 # Prometheus
```
This guide uses the RocksDB database by default.

In the future, it is recommended to switch to the faster and more efficient ParityDB option. **Note** that ParityDB is still experimental and should not be used in a production environment. If you wish to test ParityDB, you can add the flag `--database paritydb`.

Switching between database backends will require re-synchronization.

## Server Preparation

### Update repositories
```bash
apt update && apt upgrade -y
```

### Install necessary utilities
```bash
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### Check the operation of hard drives
```bash
curl -sL yabs.sh | bash -s — -ig
```

**Disk Test**
```bash
echo ; nvme smart-log /dev/nvme1 | grep -e Temperature -e Warning -e available -e percentage -e Critical ; echo ; nvme smart-log /dev/nvme1 | grep -e Temperature -e Warning -e available -e percentage -e Critical ; echo
```

**Check disk info**
```bash
inxi -D
inxi -Dxx
```

### Check internet connection
```bash
curl -sL yabs.sh | bash -s — -fg
```

### Network diagnostics (tracing)
```bash
apt install mtr -y
mtr 65.108.101.50
mtr 217.13.223.167
```

### Monitor real-time traffic consumption
```bash
apt-get install nethogs -y
nethogs
```

## Install Docker
```bash
. <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/installers/docker.sh)
```

## New Node Installation

**IMPORTANT** — In the commands below, replace everything in `<>` with your own value and remove the `<>`.

### Create a directory
```bash
mkdir -p $HOME/.kusama
```

### Set necessary permissions
```bash
chown -R $(id -u):$(id -g) $HOME/.kusama
```

### Open the necessary ports. Works without opening the ports as well
```bash
ufw allow 30333
```

Polkadot/Kusama versions - [https://hub.docker.com/r/parity/polkadot/tags](https://hub.docker.com/r/parity/polkadot/tags)

**Note** that when running polkadot in Docker, the process listens to `localhost` only by default. If you wish to connect to your node's services (RPC, WebSockets, and Prometheus), make sure to start your node using `--rpc-external --ws-external` and `--prometheus-external`.

### Ports
```
--port <PORT> - to select P2P port
--prometheus-port <PORT> - to select Prometheus port
--rpc-port <PORT> - to select RPC port
```

### RPC
```
--rpc-external - to open the RPC port
--ws-max-connections <COUNT> - max number of connections for RPC. Default is 100
--rpc-methods <Safe> - to provide a safe subset of RPC methods. Default is auto
--rpc-cors <all> - to allow access to the RPC server from anywhere. Default is localhost
```

### Pruning
```
--state-pruning 128 - saving block "states", default is 256
                      For an archive node, use --state-pruning archive
--blocks-pruning 128 - pruning saved "states", default is archive-canonical
                      For an archive node, use --blocks-pruning archive
```

### BEEFY
**IMPORTANT - ⚠️ BEEFY is enabled on Westend and Kusama ⚠️**
[https://github.com/paritytech/polkadot/pull/7661](https://github.com/paritytech/polkadot/pull/7661)

BEEFY is a consensus protocol that helps connect Ethereum or Kusama <> Polkadot bridging. BEEFY is currently enabled by default and conflicts with `sync warp` when the `--validator` flag is enabled.

Validators using `sync warp` for Kusama and Westend must disable BEEFY by adding the `--no-beefy` flag or removing the `--sync=warp` flag.

### Run Docker with the validator name `<moniker>`
```bash
docker run -dit \
--name kusama_node \
--restart always \
--network host \
-v $HOME/.kusama:/data -u $(id -u ${USER}):$(id -g ${USER}) \
parity/polkadot:v1.9.0 --base-path /data --chain kusama \
--validator --name "<moniker>" \
--public-addr /ip4/$(wget -qO- eth0.me)/tcp/30333 \
--port 30333 --rpc-port 9933 --prometheus-port 9615 \
--telemetry-url 'wss://telemetry.polkadot.io/submit/ 1' \
--telemetry-url 'wss://telemetry-backend.w3f.community/submit 1'
```

Now, the node should appear in telemetry - [telemetry W3F](https://telemetry.w3f.community/#list/0x91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3)
![image](https://github.com/user-attachments/assets/41f3ca57-1a0d-408c-8851-9d776b24c6db)


## Validator Setup

After the node is synchronized, extract the key from the node by entering the command below. If the node is not on standard ports, change the RPC port at the end of the command.
```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

If you receive a result similar to:
```json
{"jsonrpc":"2.0","result":"0xa0very0long0hex0string","id":1}
```
Copy the key (highlighted in bold), it will be needed soon.

### Check if keys were created
```bash
ls -a $HOME/.kusama/chains/ksmcc3/keystore/
```

Go to the website and select `Network - Staking - Accounts - Validator`. Select the stash account, staking amount, and click next. Then paste the key obtained from the validator node, choose the commission percentage, and sign the transaction.

## Validator Transfer

- Start the node on the new server as usual and synchronize fully.
- After synchronizing on the new server, run the

 command and copy the new key:
```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```
- Go to `Network - Accounts - Change session keys` and change your key. Sign the transaction.
- Wait for the end of the era.
- After the end of the era, stop the old node.

## Updating (Manually)

### Update image
```bash
docker pull parity/polkadot
```

### Stop the node
```bash
docker stop kusama_node
```

### Remove the container
```bash
docker rm kusama_node
```

### Start with the standard command

## Snapshot

By default, the node performs `--sync full` synchronization, which downloads and verifies the full history of the blockchain.

### Fast sync
Another sync option:
```bash
--sync fast
```
Downloads the history of block headers and verifies changes in the authority set to obtain a specific (usually the latest) header. After completing this process, the node can continue full synchronization.

### Warp sync
Recently, the `warp sync` option has become more developed and popular:
```bash
--sync=warp
```
If you run a node with an empty database and the parameter `--sync=warp`, the node will first download the final proofs, after which it will be ready to validate the network and will load the remaining blocks in the background.

---

## Useful Commands

### View logs
```bash
docker logs kusama_node -fn 100
```

### Restart the node
```bash
docker restart kusama_node
```

## Node Removal

```bash
docker stop kusama_node
docker rm kusama_node

cd $HOME
rm -rf .kusama/
```
