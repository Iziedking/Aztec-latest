# Aztec Node Guide (Updated)

This guide helps you run or upgrade your **Aztec node**.  
It covers both **new node runners** and those performing an **upgrade**.  


## 🖥️ Hardware Requirements

| **OS**              | **RAM**   | **CPU**     | **Disk**                          |
|---------------------|-----------|-------------|-----------------------------------|
| Ubuntu 20.04 or later | 32 GB | 6-8 cores   | 1TB - 1.5TB SSD (Nmve) |


---

##  Recommended VPS Providers

For the most stable performance and disk I/O:  
- **Savarica**  
- **Netcup**  
  

---

## Fresh Installation

If you are running an Aztec node for the first time:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y

# Clone Aztec repo directory
mkdir -p ~/aztec && cd ~/aztec

```
```bash
# install docker
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker
sudo docker run hello-world

sudo systemctl enable docker
sudo systemctl restart docker

```

---

## host your RPC (only do if your machine meets minimum requirement)

else get from reliable RPC providers or join discord and meet friends to give RPC

see full guide here for hosting your own RPC: https://github.com/Iziedking/geth-prysm_guide

create a directory and copy the one linear code from the guide and run 

```bash
# create a directory for your rpc node
mkdir -p ~/rpc-node && cd ~/rpc-node

```
copy code from rpc guide above and run inside this directory 
wait 8-12 hours for rpc node to sync then proceed to next step
done !

---

go back to aztec directory to start your node

```bash
cd aztec
```

### Create a file called .env and edit with your own values:

```bash
nano .env
```
```bash
ETHEREUM_RPC_URL=http://localhost:8545
CONSENSUS_BEACON_URL=http://localhost:3500
VALIDATOR_PRIVATE_KEYS=0xyour_validator_key_here
COINBASE=0xyour_coinbase_address
P2P_IP=your_server_ip

```
hint: if you are hosting your rpc leave the RPC and BEACON variables the way they are otherwise, change to your values



### Now create a docker-compose.yml file:

```bash
nano docker-compose.yml
```

```bash
services:
  aztec-node:
    container_name: aztec-sequencer
    image: aztecprotocol/aztec:2.0.2
    restart: unless-stopped
    environment:
      ETHEREUM_HOSTS: ${ETHEREUM_RPC_URL}
      L1_CONSENSUS_HOST_URLS: ${CONSENSUS_BEACON_URL}
      DATA_DIRECTORY: /data
      VALIDATOR_PRIVATE_KEYS: ${VALIDATOR_PRIVATE_KEYS}
      COINBASE: ${COINBASE}
      P2P_IP: ${P2P_IP}
      LOG_LEVEL: info
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network testnet --node --archiver --sequencer'
    ports:
      - 40400:40400/tcp
      - 40400:40400/udp
      - 8080:8080
    volumes:
      - /root/.aztec/testnet/data/:/data

```

### Start the node:

```bash
docker compose up -d
docker compose logs -f --tail=1000
```

add screen for persistent logs 

```bash
sudo apt install screen

screen -S aztecNode
```
```bash
docker compose logs -f --tail=1000
```

use ctrl + A + D to minimize screen 

to enter screen again 

```bash
screen -r aztecNode
```

---

# Upgrade Existing Node

### If you already have an older Aztec node running in Docker, follow these steps:

```bash
cd ~/aztec
docker compose down -v
rm -rf ~/.aztec/alpha-testnet/data/
nano docker-compose.yml
```

edit your docker-compose.yml file 

```bash
services:
  aztec-node:
    container_name: aztec-sequencer
    image: aztecprotocol/aztec:2.0.2
    restart: unless-stopped
    environment:
      ETHEREUM_HOSTS: ${ETHEREUM_RPC_URL}
      L1_CONSENSUS_HOST_URLS: ${CONSENSUS_BEACON_URL}
      DATA_DIRECTORY: /data
      VALIDATOR_PRIVATE_KEYS: ${VALIDATOR_PRIVATE_KEYS}
      COINBASE: ${COINBASE}
      P2P_IP: ${P2P_IP}
      LOG_LEVEL: info
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network testnet --node --archiver --sequencer'
    ports:
      - 40400:40400/tcp
      - 40400:40400/udp
      - 8080:8080
    volumes:
      - /root/.aztec/testnet/data/:/data
```
start node 

```bash
docker compose up -d
```

---

# Upgrading Aztec Node to Version 2.0.4 and Configuring Governance Proposal
Aztec Labs has released version 2.0.4. It is recommended to update your node as soon as possible (within 24 hours) to avoid potential slashing.
Additionally, a new governance proposal is being put forward. You need to signal your support by configuring the proposer payload.

## For Docker Users
Use this one-line command to upgrade your node and configure the governance payload:

```bash
cd aztec && \
docker compose down && \
echo 'GOVERNANCE_PROPOSER_PAYLOAD_ADDRESS=0x0CD09dDEabef70108Ce02576DF1eb333C4244c66' >> .env && \
sed -i 's|image: aztecprotocol/aztec:.*|image: aztecprotocol/aztec:2.0.4|' docker-compose.yml && \
sed -i '/environment:/a \      GOVERNANCE_PROPOSER_PAYLOAD_ADDRESS: ${GOVERNANCE_PROPOSER_PAYLOAD_ADDRESS}' docker-compose.yml && \
docker compose pull && docker compose up -d
```

### After sucessful update your node logs should look like this:
<img width="2079" height="1513" alt="image" src="https://github.com/user-attachments/assets/44e968ea-0c2b-4f7d-a96a-3d2304b2cbd7" />



### If you experience snapshot error (see Image for error logs)
<img width="1833" height="1214" alt="image" src="https://github.com/user-attachments/assets/7b5a952d-1d16-4da9-95e1-9a6685bd6dca" />

### use this one linear command to solve this issue:

```bash
cd ~/aztec && \
docker compose down && \
sed -i "s|--sequencer'|--sequencer --snapshots-url https://snapshots.aztec.graphops.xyz/files/'|" docker-compose.yml && \
docker compose pull && \
docker compose up -d
```

---

# For CLI users 

### Dependencies: Install Node.js and Yarn if not already
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
npm install -g yarn
```

### Install/Update Aztec CLI to 2.0.4
```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

aztec-up 2.0.4
```

### Stop Any Existing Node (If Running)
```bash
pkill -f aztec
```

### clear old data for a clean upgrade (optional)
```bash
rm -rf ~/.aztec/testnet/data/
```

### Start the Node with Governance Configuration
```bash
aztec start --node --archiver --sequencer \
  --network testnet \
  --l1-rpc-urls your_rpc_url \
  --l1-consensus-host-urls your_beacon_url \
  --sequencer.validatorPrivateKey your_private_key \
  --sequencer.coinbase your_node_walletAddress \
  --p2p.p2pIp your_vps_ip \
  --sequencer.governanceProposerPayload 0x0CD09dDEabef70108Ce02576DF1eb333C4244c66
```

### Run in Background: Wrap in screen -S aztec for detachment:
```bash
screen -S aztec
```
Paste the aztec start command above
Detach with Ctrl+A, D; reattach with ``screen -r aztec``

### If Snapshot Sync Issues Occur
```bash
aztec start --node --archiver --sequencer \
  --network testnet \
  --l1-rpc-urls your_rpc_url \
  --l1-consensus-host-urls your_beacon_url \
  --sequencer.validatorPrivateKeys your_private_key \
  --sequencer.coinbase your_node_walletAddress \
  --sequencer.governanceProposerPayload 0xDcD9DdeABEF70108ce02576Df1Eb333c4244c666 \
  --p2p.p2pIp your_vps_ip \
  --snapshots-url https://snapshots.aztec.graphops.xyz/files/
```
ensure to replace all the flags by your actual detals

---


---

# Aztec Node Upgrade to v2.1.2


Aztec Labs has released version 2.1.2 for the testnet, introducing a new rollout contract and GSE. This upgrade requires rejoining the testnet by approving the rollout to spend 200k STAKE and generating new BLS keys. The process involves updating your node and registering as a sequencer with the new keys.

**Prerequisites:**
- Ubuntu server with Docker and Docker Compose installed.
- Existing Aztec node setup (from previous versions).
- Sepolia ETH for gas fees (0.2–0.5 ETH recommended for the new address).
- Your old sequencer private key and ETH RPC URL.


## Step 1: Stop the Node
Shut down your current node to prepare for the upgrade.

```bash
cd ~/aztec && \
docker compose down -v
```


## Step 2: Install Aztec CLI
Install or update the Aztec CLI for key generation and registration.

```bash
bash -i <(curl -s https://install.aztec.network)
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
aztec-up
```

## Step 3: Install Foundry
Foundry is used in the script for key generation (cast wallet).

```bash
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup
```

Verify: `cast --version`.

## Step 4: Run the Key Generation and Registration Script
```bash
nano aztec-upgrade.sh
```
copy and paste this script thanks to community mod @Hendrix for making provisions

```bash
#!/bin/bash

clear

# Install jq if not present
if ! command -v jq &> /dev/null; then
    echo "jq not found. Installing..."
    sudo apt update && sudo apt install jq -y
fi

# Source bashrc to ensure PATH includes Aztec CLI
source ~/.bashrc

# Check if aztec CLI is installed
if ! command -v aztec &> /dev/null; then
    echo "Error: aztec command not found. Please install Aztec CLI first:"
    echo "bash -i <(curl -s https://install.aztec.network)"
    echo "Then run: echo 'export PATH=\"\$HOME/.aztec/bin:\$PATH\"' >> ~/.bashrc"
    echo "source ~/.bashrc"
    echo "aztec-up"
    exit 1
fi

# Check if cast (from Foundry) is installed
if ! command -v cast &> /dev/null; then
    echo "Error: cast command not found. Please install Foundry:"
    echo "curl -L https://foundry.paradigm.xyz | bash"
    echo "source ~/.bashrc"
    echo "foundryup"
    exit 1
fi

# Output CLI version for debugging
echo "Aztec CLI version: $(aztec --version)"

echo "pls provide your OLD validator info."
read -sp " put your old Sequencer Private Key (will not be shown): " OLD_PRIVATE_KEY
echo ""
echo "Good. Starting..." && echo ""
read -p " put your sepolia rpc only: " ETH_RPC
echo "Good. Starting..." && echo ""

# Create keystore directory if not exists
mkdir -p ~/.aztec/keystore

rm -f ~/.aztec/keystore/key1.json
echo "BE READY to write down your private key both eth and BLS and your eth address.."
read -p "Press [Enter] to generate your new keys...."

aztec validator-keys new --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 && echo "" || { echo "Key generation failed."; exit 1; }
KEYSTORE_FILE=~/.aztec/keystore/key1.json

if [ ! -f "$KEYSTORE_FILE" ]; then
    echo "Error: Keystore file not generated. Check aztec CLI output above."
    echo "Possible fix: Run 'aztec-up' to update CLI, or check installation."
    exit 1
fi

NEW_ETH_PRIVATE_KEY=$(jq -r '.validators[0].attester.eth' $KEYSTORE_FILE)
NEW_BLS_PRIVATE_KEY=$(jq -r '.validators[0].attester.bls' $KEYSTORE_FILE)

if [ -z "$NEW_ETH_PRIVATE_KEY" ] || [ -z "$NEW_BLS_PRIVATE_KEY" ]; then
    echo "Error: Failed to extract keys from keystore. Check jq output or file contents."
    cat $KEYSTORE_FILE
    exit 1
fi

NEW_PUBLIC_ADDRESS=$(cast wallet address --private-key $NEW_ETH_PRIVATE_KEY)

echo "good! Your new keys are below. SAVE THIS INFO SECURELY!"
echo " - NEW ETH Private Key: $NEW_ETH_PRIVATE_KEY"
echo " - NEW BLS Private Key: $NEW_BLS_PRIVATE_KEY"
echo " - NEW Public Address: $NEW_PUBLIC_ADDRESS"
echo ""

echo "You need to send 0.2 to 0.5 Sepolia eth to this new address:"
echo " $NEW_PUBLIC_ADDRESS"
read -p " After the funding txn is confirmed, press [Enter] to continue..." && echo ""

echo "Approving STAKE spending..."
cast send 0x139d2a7a0881e16332a7d1f8db383a4507e1ea7a "approve(address,uint256)" 0xebd99ff6ff6677205509ae7f3f93d0ca52ac85d6 200000ether --private-key "$OLD_PRIVATE_KEY" --rpc-url "$ETH_RPC" && echo "" || { echo "Approval failed. Check address, key, or RPC."; exit 1; }

echo "joining the testnet yey..."
aztec add-l1-validator \
  --l1-rpc-urls "$ETH_RPC" \
  --network testnet \
  --private-key "$OLD_PRIVATE_KEY" \
  --attester "$NEW_PUBLIC_ADDRESS" \
  --withdrawer "$NEW_PUBLIC_ADDRESS" \
  --bls-secret-key "$NEW_BLS_PRIVATE_KEY" \
  --rollup 0xebd99ff6ff6677205509ae7f3f93d0ca52ac85d6 && echo "" || { echo "Registration failed. Check addresses or CLI version."; exit 1; }

echo "All done! u have successfully joined the new testnet now rerun your node with your new pvt and address"
```
Save and exit (Ctrl+O, Enter, Ctrl+X in nano)

make it executable 
```bash
chmod +x aztec-upgrade.sh
```
run 
```bash
./aztec-upgrade.sh
```

**Script Notes:**
- It prompts for your old private key and ETH RPC.
- Generates new keys using `aztec validator-keys new`.
- Outputs new keys—save them securely!
- Funds the new address with 0.2–0.5 Sepolia ETH (from your wallet).
- Approves the rollout contract (`0x139d2a7a0881e16332a7d1fDBDB383AA4507E1eA7a`) to spend 200k STAKE to the GSE (`0xebd99fF6fF6677205509ae7F3F93d0ca52ac85d67`).
- Registers with `aztec add-l1-validator`.
- If errors occur (e.g., invalid address), check casing or use lowercase hex.

if you face command not found, it means export dint work try this manually
```bash
export PATH="/root/.aztec/bin:$PATH"
source /root/.bashrc
aztec-up
aztec --version 
aztec validator-keys new --help
```

then retry running script again
```bash
./aztec-upgrade.sh
```

## Step 5: Update Your .env File
Replace old values with the new ones from Step 4.

```bash
cd ~/aztec
nano .env
```

Update:
- `VALIDATOR_PRIVATE_KEYS` → New ETH private key (`$NEW_ETH_PRIVATE_KEY`)
- `COINBASE` → New public address (`$NEW_PUBLIC_ADDRESS`)

Save and exit (Ctrl+O, Enter, Ctrl+X in nano).

## Step 6: Update and Start Your Node
Update the Docker image to 2.1.2 and restart.

```bash
sed -i 's|image: aztecprotocol/aztec:.*|image: aztecprotocol/aztec:2.1.2|' docker-compose.yml && \
docker compose pull && \
docker compose up -d
```

## Verification
- Check logs: `docker compose logs -f`
- Node status: monitor from [dashtec](https://dashtec.xyz/)
- Sequencer status on-chain (using cast or explorer): Check if your new attester address is validating.
- Aztec Scan: [aztec scan](https://aztecscan.xyz/) (search your address).


## ensure to join discord and ask for help where you find errors 
