# Installation Guide

### Install Dependencies
```bash
# Update system package and install build tools

sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

### Configure Moniker
```bash
# Replace <your-moniker-name> with your own validator name

MONIKER="<your-moniker-name>"
```

### Install Go
```bash
# install go version 1.20.10

sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.20.10.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

### Build Binaries
```bash
# Cloning project repository & Compile binaries

cd $HOME
rm -rf lava
git clone https://github.com/lavanet/lava.git
cd lava
git checkout v2.0.0
```

```bash
# Export binaries name and build

export LAVA_BINARY=lavad
make build
```

```bash
# Prepare binaries for cosmovisor

mkdir -p $HOME/.lava/cosmovisor/genesis/bin
mv build/lavad $HOME/.lava/cosmovisor/genesis/bin/
rm -rf build
```

```bash
# Create symlinks

sudo ln -s $HOME/.lava/cosmovisor/genesis $HOME/.lava/cosmovisor/current -f
sudo ln -s $HOME/.lava/cosmovisor/current/bin/lavad /usr/local/bin/lavad -f
```

### Cosmovisor Setup
```bash
# Install cosmovisor

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

### Create Service
```bash
# Create a systemd service

sudo tee /etc/systemd/system/lava.service > /dev/null << EOF
[Unit]
Description=lava node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.lava"
Environment="DAEMON_NAME=lavad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.lava/cosmovisor/current/bin"
 
[Install]
WantedBy=multi-user.target
EOF
```

### Enable Service
```bash
# Enable lava systemd service

sudo systemctl daemon-reload
sudo systemctl enable lava
```

### Initialize Node
```bash
#Setting node configuration

lavad config chain-id lava-testnet-2
lavad config keyring-backend test
lavad config node tcp://localhost:20457
```
```bash
# Initialize node

lavad init $MONIKER --chain-id lava-testnet-2
```

### Download Genesis & Addrbook
```bash
# Download genesis & addrbook file

curl -Ls "you can get url on official discord channel" > $HOME/.lava/config/genesis.json
curl -Ls "you can get url on official discord channel" > $HOME/.lava/config/addrbook.json
```

### Configure Seeds
```bash
# Setting up a seed peers

# sed -i -e "s|^seeds *=.*|seeds = \"you can get url on official discord channel\"|" $HOME/.lava/config/config.toml
```

### Configure Gas Prices
```bash
# Setting up a gas prices

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ulava\"|" $HOME/.lava/config/app.toml
```

### Pruning Setting
```bash
# Configure pruning setting

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.lava/config/app.toml
```

### Start Service
```bash
sudo systemctl start lava
```
