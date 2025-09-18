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

ensure to join discord and ask for help where you find errors 
