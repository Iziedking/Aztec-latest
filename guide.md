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

if you face command not found, it means export dint work try this manually
```bash
export PATH="/root/.aztec/bin:$PATH"
source /root/.bashrc
aztec-up
aztec --version 
aztec validator-keys new --help
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
i made a script to help you confirm things are right before procedding copy and paste this script 

```bash
#!/bin/bash

clear

##############################################
# Aztec Node Upgrade to v2.1.2
# With Smart Approval Checking & PATH Fixes
##############################################

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Official Aztec v2.1.2 Contract Addresses (Sepolia)
STAKE_TOKEN="0x139d2a7a0881e16332d7D1F8DB383A4507E1Ea7A"
GSE_CONTRACT="0xebd99ff0ff6677205509ae73f93d0ca52ac85d67"
REQUIRED_ALLOWANCE="200000000000000000000000"  # 200k STAKE

##############################################
# Fix PATH for Foundry and Aztec
##############################################

# Add common binary paths
export PATH="$HOME/.foundry/bin:$PATH"
export PATH="$HOME/.aztec/bin:$PATH"
export PATH="/root/.foundry/bin:$PATH"
export PATH="/root/.aztec/bin:$PATH"

# Source shell configs
[ -f ~/.bashrc ] && source ~/.bashrc
[ -f ~/.profile ] && source ~/.profile

##############################################
# Install Dependencies
##############################################

if ! command -v jq &> /dev/null; then
    echo "Installing jq..."
    sudo apt update && sudo apt install jq -y
fi

if ! command -v bc &> /dev/null; then
    echo "Installing bc..."
    sudo apt update && sudo apt install bc -y
fi

##############################################
# Validate Prerequisites
##############################################

echo "=========================================="
echo "Checking Prerequisites"
echo "=========================================="
echo ""

# Check Aztec CLI
if ! command -v aztec &> /dev/null; then
    echo -e "${RED}Error: aztec CLI not found${NC}"
    echo ""
    echo "Searching for aztec installation..."
    AZTEC_BIN=$(find ~ -name "aztec" -type f 2>/dev/null | head -n 1)
    
    if [ -n "$AZTEC_BIN" ]; then
        AZTEC_DIR=$(dirname "$AZTEC_BIN")
        echo "Found aztec at: $AZTEC_BIN"
        export PATH="$AZTEC_DIR:$PATH"
        echo "Added to PATH: $AZTEC_DIR"
        
        if ! command -v aztec &> /dev/null; then
            echo -e "${RED}Still can't run aztec. Please install:${NC}"
            echo "  bash -i <(curl -s https://install.aztec.network)"
            echo "  echo 'export PATH=\"\$HOME/.aztec/bin:\$PATH\"' >> ~/.bashrc"
            echo "  source ~/.bashrc && aztec-up"
            exit 1
        fi
    else
        echo -e "${RED}Aztec CLI not found. Install it:${NC}"
        echo "  bash -i <(curl -s https://install.aztec.network)"
        echo "  echo 'export PATH=\"\$HOME/.aztec/bin:\$PATH\"' >> ~/.bashrc"
        echo "  source ~/.bashrc && aztec-up"
        exit 1
    fi
fi

echo -e "${GREEN}✓ Aztec CLI found${NC}"
echo "  Version: $(aztec --version)"

# Check Foundry (cast)
if ! command -v cast &> /dev/null; then
    echo -e "${RED}Error: cast (Foundry) not found${NC}"
    echo ""
    echo "Searching for Foundry installation..."
    CAST_BIN=$(find ~ -name "cast" -type f 2>/dev/null | head -n 1)
    
    if [ -n "$CAST_BIN" ]; then
        FOUNDRY_DIR=$(dirname "$CAST_BIN")
        echo "Found cast at: $CAST_BIN"
        export PATH="$FOUNDRY_DIR:$PATH"
        echo "Added to PATH: $FOUNDRY_DIR"
        
        if ! command -v cast &> /dev/null; then
            echo -e "${RED}Still can't run cast. Reinstalling Foundry...${NC}"
            curl -L https://foundry.paradigm.xyz | bash
            source ~/.bashrc
            foundryup
            
            if ! command -v cast &> /dev/null; then
                echo -e "${RED}Foundry installation failed${NC}"
                exit 1
            fi
        fi
    else
        echo -e "${YELLOW}Foundry not found. Installing...${NC}"
        curl -L https://foundry.paradigm.xyz | bash
        source ~/.bashrc
        
        # Add to PATH
        export PATH="$HOME/.foundry/bin:$PATH"
        
        # Run foundryup
        if command -v foundryup &> /dev/null; then
            foundryup
        else
            source "$HOME/.foundry/env"
            foundryup
        fi
        
        if ! command -v cast &> /dev/null; then
            echo -e "${RED}Foundry installation failed${NC}"
            echo "Please install manually:"
            echo "  curl -L https://foundry.paradigm.xyz | bash"
            echo "  source ~/.bashrc"
            echo "  foundryup"
            exit 1
        fi
    fi
fi

echo -e "${GREEN}✓ Foundry (cast) found${NC}"
echo "  Version: $(cast --version | head -n 1)"
echo ""

##############################################
# Collect User Input
##############################################

echo "=========================================="
echo "Aztec Node Upgrade to v2.1.2"
echo "=========================================="
echo ""

read -sp "Enter your OLD Sequencer Private Key: " OLD_PRIVATE_KEY
echo ""

if [ -z "$OLD_PRIVATE_KEY" ]; then
    echo -e "${RED}Error: Private key cannot be empty${NC}"
    exit 1
fi

read -p "Enter your Sepolia RPC URL: " ETH_RPC
echo ""

if [ -z "$ETH_RPC" ]; then
    echo -e "${RED}Error: RPC URL cannot be empty${NC}"
    exit 1
fi

##############################################
# Verify Old Address
##############################################

echo "Validating old sequencer address..."
OLD_ADDRESS=$(cast wallet address --private-key "$OLD_PRIVATE_KEY" 2>/dev/null)

if [ -z "$OLD_ADDRESS" ]; then
    echo -e "${RED}Error: Invalid private key${NC}"
    exit 1
fi

echo -e "${GREEN}✓ Old sequencer address: $OLD_ADDRESS${NC}"
echo ""

##############################################
# Verify RPC Connection
##############################################

echo "Testing RPC connection..."
BLOCK_NUMBER=$(cast block-number --rpc-url "$ETH_RPC" 2>/dev/null)

if [ -z "$BLOCK_NUMBER" ]; then
    echo -e "${RED}Error: Cannot connect to RPC endpoint${NC}"
    echo "Please check your RPC URL"
    exit 1
fi

echo -e "${GREEN}✓ RPC connected (Block: $BLOCK_NUMBER)${NC}"
echo ""

##############################################
# Verify Smart Contracts Exist
##############################################

echo "Verifying contract addresses..."

STAKE_CODE=$(cast code "$STAKE_TOKEN" --rpc-url "$ETH_RPC" 2>/dev/null)
if [ -z "$STAKE_CODE" ] || [ "$STAKE_CODE" == "0x" ]; then
    echo -e "${RED}Error: STAKE token contract not found${NC}"
    echo "Address: $STAKE_TOKEN"
    echo "Your RPC may not be synced to Sepolia"
    exit 1
fi
echo -e "${GREEN}✓ STAKE token verified${NC}"

GSE_CODE=$(cast code "$GSE_CONTRACT" --rpc-url "$ETH_RPC" 2>/dev/null)
if [ -z "$GSE_CODE" ] || [ "$GSE_CODE" == "0x" ]; then
    echo -e "${RED}Error: GSE contract not found${NC}"
    echo "Address: $GSE_CONTRACT"
    exit 1
fi
echo -e "${GREEN}✓ GSE contract verified${NC}"
echo ""

##############################################
# Check Account Balances
##############################################

echo "Checking account balances..."

# ETH Balance
ETH_BALANCE=$(cast balance "$OLD_ADDRESS" --rpc-url "$ETH_RPC" 2>/dev/null)
ETH_BALANCE_READABLE=$(cast --to-unit "$ETH_BALANCE" ether 2>/dev/null)
echo "ETH Balance: ${ETH_BALANCE_READABLE} ETH"

if (( $(echo "$ETH_BALANCE_READABLE < 0.05" | bc -l) )); then
    echo -e "${YELLOW}⚠ Warning: Low ETH balance, you may need more for gas${NC}"
fi

# STAKE Balance
STAKE_BALANCE=$(cast call "$STAKE_TOKEN" \
  "balanceOf(address)(uint256)" \
  "$OLD_ADDRESS" \
  --rpc-url "$ETH_RPC" 2>/dev/null)

if [ -n "$STAKE_BALANCE" ] && [ "$STAKE_BALANCE" != "0" ]; then
    STAKE_BALANCE_READABLE=$(cast --to-unit "$STAKE_BALANCE" ether 2>/dev/null)
    echo "STAKE Balance: ${STAKE_BALANCE_READABLE} STAKE"
else
    echo -e "${RED}STAKE Balance: 0 STAKE${NC}"
    echo -e "${RED}Error: You need STAKE tokens to join the testnet${NC}"
    exit 1
fi

echo ""

##############################################
# Check Existing Approval
##############################################

echo "=========================================="
echo "Checking STAKE Approval Status"
echo "=========================================="
echo ""

CURRENT_ALLOWANCE=$(cast call "$STAKE_TOKEN" \
  "allowance(address,address)(uint256)" \
  "$OLD_ADDRESS" \
  "$GSE_CONTRACT" \
  --rpc-url "$ETH_RPC" 2>/dev/null)

if [ -z "$CURRENT_ALLOWANCE" ]; then
    echo -e "${RED}Error: Could not check allowance${NC}"
    exit 1
fi

echo "Current Allowance: $CURRENT_ALLOWANCE"
echo "Required Allowance: $REQUIRED_ALLOWANCE"
echo ""

if [ "$CURRENT_ALLOWANCE" == "$REQUIRED_ALLOWANCE" ]; then
    echo -e "${GREEN}✓ Approval already exists! (200k STAKE approved)${NC}"
    echo -e "${GREEN}✓ Skipping approval transaction${NC}"
    APPROVAL_NEEDED=false
elif [ "$CURRENT_ALLOWANCE" == "0" ]; then
    echo -e "${YELLOW}⚠ No approval found${NC}"
    echo "Need to approve 200k STAKE for GSE contract"
    APPROVAL_NEEDED=true
else
    CURRENT_READABLE=$(cast --to-unit "$CURRENT_ALLOWANCE" ether 2>/dev/null)
    echo -e "${YELLOW}⚠ Partial approval found: ${CURRENT_READABLE} STAKE${NC}"
    echo "Need to update approval to 200k STAKE"
    APPROVAL_NEEDED=true
fi

echo ""

##############################################
# Send Approval Transaction (if needed)
##############################################

if [ "$APPROVAL_NEEDED" = true ]; then
    echo "=========================================="
    echo "Sending Approval Transaction"
    echo "=========================================="
    echo ""
    
    read -p "Press [Enter] to approve 200k STAKE for GSE contract..."
    
    echo "Submitting approval transaction..."
    echo "This may take 1-2 minutes on Sepolia..."
    echo ""
    
    set +e
    APPROVAL_OUTPUT=$(cast send "$STAKE_TOKEN" \
      "approve(address,uint256)" \
      "$GSE_CONTRACT" \
      "$REQUIRED_ALLOWANCE" \
      --private-key "$OLD_PRIVATE_KEY" \
      --rpc-url "$ETH_RPC" \
      --legacy \
      --gas-price 30gwei \
      --confirmations 2 2>&1)
    
    APPROVAL_STATUS=$?
    set -e
    
    echo "$APPROVAL_OUTPUT"
    echo ""
    
    if [ $APPROVAL_STATUS -eq 0 ]; then
        echo -e "${GREEN}✓ Transaction submitted${NC}"
    else
        # Check if it's "already known" error (transaction already in mempool)
        if echo "$APPROVAL_OUTPUT" | grep -q "already known"; then
            echo -e "${YELLOW}⚠ Transaction already submitted (in mempool)${NC}"
        else
            echo -e "${YELLOW}⚠ Transaction command returned error${NC}"
        fi
    fi
    
    echo ""
    echo "Waiting 30 seconds for confirmation..."
    sleep 30
    
    # Verify approval succeeded
    echo "Verifying approval on-chain..."
    FINAL_ALLOWANCE=$(cast call "$STAKE_TOKEN" \
      "allowance(address,address)(uint256)" \
      "$OLD_ADDRESS" \
      "$GSE_CONTRACT" \
      --rpc-url "$ETH_RPC" 2>/dev/null)
    
    if [ "$FINAL_ALLOWANCE" == "$REQUIRED_ALLOWANCE" ]; then
        echo -e "${GREEN}✓ SUCCESS! Approval verified (200k STAKE)${NC}"
    else
        echo -e "${RED}✗ Approval verification failed${NC}"
        echo "Expected: $REQUIRED_ALLOWANCE"
        echo "Got: $FINAL_ALLOWANCE"
        echo ""
        echo "Please check transaction on Etherscan:"
        echo "https://sepolia.etherscan.io/address/$OLD_ADDRESS"
        echo ""
        read -p "Press [Enter] if transaction succeeded, or Ctrl+C to abort..."
    fi
    
    echo ""
fi

##############################################
# Generate New Keys
##############################################

echo "=========================================="
echo "Generating New Validator Keys"
echo "=========================================="
echo ""

mkdir -p ~/.aztec/keystore
rm -f ~/.aztec/keystore/key1.json

echo "IMPORTANT: Save the keys that will be generated!"
read -p "Press [Enter] to generate new ETH and BLS keys..."
echo ""

aztec validator-keys new --fee-recipient 0x0000000000000000000000000000000000000000000000000000000000000000 && echo "" || {
    echo -e "${RED}Error: Key generation failed${NC}"
    exit 1
}

KEYSTORE_FILE=~/.aztec/keystore/key1.json

if [ ! -f "$KEYSTORE_FILE" ]; then
    echo -e "${RED}Error: Keystore file not generated${NC}"
    echo "Try running: aztec-up"
    exit 1
fi

NEW_ETH_PRIVATE_KEY=$(jq -r '.validators[0].attester.eth' "$KEYSTORE_FILE")
NEW_BLS_PRIVATE_KEY=$(jq -r '.validators[0].attester.bls' "$KEYSTORE_FILE")

if [ -z "$NEW_ETH_PRIVATE_KEY" ] || [ "$NEW_BLS_PRIVATE_KEY" == "null" ]; then
    echo -e "${RED}Error: Failed to extract keys from keystore${NC}"
    cat "$KEYSTORE_FILE"
    exit 1
fi

NEW_PUBLIC_ADDRESS=$(cast wallet address --private-key "$NEW_ETH_PRIVATE_KEY")

echo "=========================================="
echo "NEW KEYS GENERATED - SAVE THESE SECURELY!"
echo "=========================================="
echo ""
echo "NEW ETH Private Key: $NEW_ETH_PRIVATE_KEY"
echo "NEW BLS Private Key: $NEW_BLS_PRIVATE_KEY"
echo "NEW Public Address:  $NEW_PUBLIC_ADDRESS"
echo ""
echo "=========================================="
echo ""

##############################################
# Fund New Address
##############################################

echo "You must send 0.2 to 0.5 Sepolia ETH to your NEW address:"
echo "$NEW_PUBLIC_ADDRESS"
echo ""
read -p "After funding is confirmed, press [Enter] to continue..."
echo ""

# Verify new address has ETH
echo "Verifying new address has ETH..."
NEW_ETH_BALANCE=$(cast balance "$NEW_PUBLIC_ADDRESS" --rpc-url "$ETH_RPC" --ether 2>/dev/null)
echo "New address balance: $NEW_ETH_BALANCE ETH"

if (( $(echo "$NEW_ETH_BALANCE < 0.1" | bc -l) )); then
    echo -e "${YELLOW}⚠ Warning: Balance seems low, but continuing...${NC}"
fi
echo ""

##############################################
# Register as Validator
##############################################

echo "=========================================="
echo "Registering as Validator on Testnet"
echo "=========================================="
echo ""

aztec add-l1-validator \
  --l1-rpc-urls "$ETH_RPC" \
  --network testnet \
  --private-key "$OLD_PRIVATE_KEY" \
  --attester "$NEW_PUBLIC_ADDRESS" \
  --withdrawer "$NEW_PUBLIC_ADDRESS" \
  --bls-secret-key "$NEW_BLS_PRIVATE_KEY" \
  --rollup "$GSE_CONTRACT" && echo "" || {
    echo -e "${RED}Error: Registration failed${NC}"
    echo "Check CLI version, parameters, and network connectivity"
    exit 1
}

echo -e "${GREEN}✓ Successfully registered as validator!${NC}"
echo ""

##############################################
# Final Instructions
##############################################

echo "=========================================="
echo "✓ UPGRADE COMPLETE"
echo "=========================================="
echo ""
echo "Next Steps:"
echo ""
echo "1. Update your .env file:"
echo "   cd ~/aztec"
echo "   nano .env"
echo ""
echo "   Update these values:"
echo "   VALIDATOR_PRIVATE_KEYS=$NEW_ETH_PRIVATE_KEY"
echo "   COINBASE=$NEW_PUBLIC_ADDRESS"
echo ""
echo "2. Update Docker image to v2.1.2:"
echo "   sed -i 's|image: aztecprotocol/aztec:.*|image: aztecprotocol/aztec:2.1.2|' docker-compose.yml"
echo ""
echo "3. Restart your node:"
echo "   docker compose pull"
echo "   docker compose up -d"
echo ""
echo "4. Verify node is running:"
echo "   docker compose logs -f"
echo ""
echo "5. Check your validator status:"
echo "   - Aztec Scan: https://aztec-scan.io"
echo "   - Search for: $NEW_PUBLIC_ADDRESS"
echo ""
echo "=========================================="
echo ""
echo -e "${GREEN}Congratulations! You're now running v2.1.2${NC}"
echo ""
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

if you face any form of time out and script terminates 
check with new address generated from CLI (`$NEW_PUBLIC_ADDRESS`) on [dashtec](https://dashtec.xyz/) if you are on queue 
if you are on queue, just run the following commands below to spin your node

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
