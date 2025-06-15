# OG-Labs
A step by step guide on how to Run Og Labs Storage node using Google Cloud Platform on Testnet

## System Requirements

| Component     | Recommended |
|---------------|-------------|
| CPU           | 4+ cores    |
| RAM           | 16 GB+      |
| SSD           | 500 GB+     |
| OS            | Ubuntu 24.04|
| Bandwidth     | 500 Mbps    |

## Step 1: Create a GCP VM

1. Go to Google Cloud Console → Compute Engine → VM Instances → Create Instance
2. Choose:
   - Machine: e2-standard-4 (or higher)
   - Boot Disk: Ubuntu 24.04 LTS (500GB+)
   - Allow HTTP/HTTPS
3. Click “Create”

## Step 2: Open Required Firewall Ports

Open these TCP/UDP ports:

- 30333, 30334, 9000

From VPC > Firewall > Create rule:

```
Name:       og-storage-node
Target:     All instances
Source:     0.0.0.0/0
Protocols:  tcp:30333,30334,9000
            udp:30333,30334,9000
```
Add chain claim faucets and get RPC:

Add 0G-Galileo-Testnet chain from here: https://hub.0g.ai/portfolio/token

Take faucet: https://faucet.0g.ai/

Get RPC: https://www.astrostake.xyz/0g-status

---

##  Step 3: Install Dependencies

SSH into the VM and run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential pkg-config libssl-dev curl clang cmake screen git unzip
```
Install Rust:

```bash
curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
```

Install Go:

```bash
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz && \
rm go1.24.3.linux-amd64.tar.gz && \
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc && \
source ~/.bashrc
```
---

##  Step 4: Clone and Build 0G Storage Node

```bash
git clone -b v0.8.7 https://github.com/0glabs/0g-storage-node.git
```
```bash
cd 0g-storage-node && git checkout v1.0.0 && git submodule update --init
```
```bash
cargo build --release
```
---

##  Step 5: Configure the Node

First Install screen 
```bash
sudo apt-get install screen
```

Create new Screen session 
```bash
screen -S ognode
``` 

```bash
rm -rf $HOME/0g-storage-node/run/config.toml
```

```bash
curl -o $HOME/0g-storage-node/run/config.toml https://raw.githubusercontent.com/Shahzuby/0glab-storage-node-guide/main/config.toml
```

```bash
nano $HOME/0g-storage-node/run/config.toml
```

Edit the following fields:

- `blockchain_rpc_endpoint = "your_rpc_endpoint"`
- `miner_key = "<64-char private key (no 0x)>"`

And save with cntl+x-y-enter 

---
Create a System Service File
```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=ZGS Node
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/0g-storage-node/run
ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config.toml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable zgs
sudo systemctl start zgs
```
---

##  Monitoring

```bash
sudo systemctl status zgs
```

complete logs
```bash
tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
```

Check block & Sync process
```bash
 while true; do     response=$(curl -s -X POST http://localhost:5678 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}');     logSyncHeight=$(echo $response | jq '.result.logSyncHeight');     connectedPeers=$(echo $response | jq '.result.connectedPeers');     echo -e "logSyncHeight: \033[32m$logSyncHeight\033[0m, connectedPeers: \033[34m$connectedPeers\033[0m";     sleep 5; done
```


check node status: https://chainscan-galileo.bangcode.id/

View Miner Details: https://storagescan-galileo.0g.ai/miner/YOUREVMADDRESS

---
## To Stop & Delete your Node
stop 
```bash
sudo systemctl stop zgs
```
Delete
```bash
sudo systemctl disable zgs
sudo rm /etc/systemd/system/zgs.service
rm -rf $HOME/0g-storage-node
```
---
## Restarting node Next day
reload
```bash
sudo systemctl daemon-reload
```
Enable 
```bash
sudo systemctl enable zgs
```
Start Service
```bash
sudo systemctl start zgs
```
---
## To check ram and storage usage of your vps 
Storage
```bash
df -h
```
Ram
```bash
free -h
```

## Join TG For Updates : https://t.me/Web3Brothers


