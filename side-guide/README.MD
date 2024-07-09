## **Preparing the server**

### **Preparing the server**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### **GO 1.21.6**

```bash
ver="1.21.6"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### **Build**

```bash
cd $HOME && mkdir -p go/bin/
git clone https://github.com/sideprotocol/side.git
cd side
git checkout v0.8.1
make install
```

### **Create/recover wallet**

```bash
# **Create wallet**
lavad keys add <walletname>

# OR

# Recover wallet
lavad keys add <walletname> --recover
```

Download Genesis From Somewhere

### **Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers**

```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uside\"/;" ~/.side/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.side/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.side/config/config.toml
seeds="582dedd866dd77f25ac0575118cf32df1ee50f98@202.182.119.24:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.side/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.side/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.side/config/config.toml
```

### **Pruning (optional)**

```bash
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.side/config/app.toml
```

### **Indexer (optional)**

```bash
indexer="null" && sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.side/config/config.toml
```

Download addrbook whom somewhere

### **Create a service file**

```bash
sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
[Unit]
Description=sided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### **Start**

```bash
sudo systemctl daemon-reload
sudo systemctl enable sided
sudo systemctl restart sided && sudo journalctl -fu sided -o cat
```

### **Create validator**

```bash
sided tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.1 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000uside \
--pubkey $(sided tendermint show-validator) \
--from <wallet> \
--moniker="YOUR_MONIKER_NAME" \
--chain-id S2-testnet-2 \
--gas 350000 \
--identity="" \
--website="" \
--details="" -y
```
