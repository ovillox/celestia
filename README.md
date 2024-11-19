Manual Installation
Official Documentation
Recommended Hardware: 4 Cores, 8GB RAM, 250GB of storage (NVME)

**install dependencies, if needed**
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**install go, if needed**
```
cd $HOME
VER="1.22.6"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

**set vars**
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export CELESTIA_CHAIN_ID="mocha-4"" >> $HOME/.bash_profile
echo "export CELESTIA_PORT="11"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**download binary**
```
cd $HOME 
rm -rf celestia-app 
git clone https://github.com/celestiaorg/celestia-app.git 
cd celestia-app/ 
APP_VERSION=v2.3.1-mocha
git checkout tags/$APP_VERSION -b $APP_VERSION 
make install
```

**config and init app**
```
celestia-appd config node tcp://localhost:${CELESTIA_PORT}657
celestia-appd config keyring-backend os
celestia-appd config chain-id mocha-4
celestia-appd init $MONIKER --chain-id mocha-4
celestia-appd download-genesis mocha-4
```

**download genesis and addrbook**
```
wget -O $HOME/.celestia-app/config/genesis.json https://server-4.itrocket.net/testnet/celestia/genesis.json
wget -O $HOME/.celestia-app/config/addrbook.json  https://server-4.itrocket.net/testnet/celestia/addrbook.json
```

**set seeds and peers**
```
SEEDS="5d0bf034d6e6a8b5ee31a2f42f753f1107b3a00e@celestia-testnet-seed.itrocket.net:11656"
PEERS="daf2cecee2bd7f1b3bf94839f993f807c6b15fbf@celestia-testnet-peer.itrocket.net:11656,6ec29df7709624d7deb589e0c5b0334be92bfa6b@37.59.57.210:26656,782a9828116f70a879d91e57156bf76c73333e53@38.46.221.159:26656,2da156fc133e1ecccb49edc67ffd0684b1811385@148.113.189.152:26656,a831cf42d79aded9d25efd71b1a6629311c2f644@95.217.120.205:11656,ee9f90974f85c59d3861fc7f7edb10894f6ac3c8@65.109.16.220:26656,6fde8d9cffe2c2fd5c6e4555dde41901a7d63540@65.108.234.28:36656,5a7566aa030f7e5e7114dc9764f944b2b1324bcd@65.109.23.114:11656,71db31cd8db312afa6f2cf48971c8430d612a5ad@206.189.143.243:26656,6cabdecd60b320c9481df4e63678623026283fab@136.243.94.113:26656,cee58e7a8724fea3022be98898d7346d12a0ef80@164.152.162.119:36656,048aa4dd677c7b49e7fed942a94aa82f8b5c0b43@38.101.149.243:56656,5f22818fcf0d8cab31a490ae038b4f06a5632685@95.217.225.107:26656,04d51161e4431b8e5f4d6d8b14655d041b3ea041@51.178.74.112:11056,aea85cf7e03258e9b02cdd8854f64857e9046d73@89.187.156.100:26698,174f976b84674e69322d61917629494d595dd847@144.24.165.142:26656,60016ce0be12c7361493e254e5c99c1cf4b6dcc2@205.178.183.14:26656,2abbf1892ce9d91acbbc55b112f3561b01fc3465@162.62.126.26:26656,c758100ed28cbc8bb657352b049b452ddad71247@141.98.217.188:26656,c2870ce12cfb08c4ff66c9ad7c49533d2bd8d412@185.183.33.109:26656,85aef6d15d0197baff696b6e31c88e0f21073c59@162.55.245.144:2400"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.celestia-app/config/config.toml
```

**set custom ports in app.toml**
```
sed -i.bak -e "s%:1317%:${CELESTIA_PORT}317%g;
s%:8080%:${CELESTIA_PORT}080%g;
s%:9090%:${CELESTIA_PORT}090%g;
s%:9091%:${CELESTIA_PORT}091%g;
s%:8545%:${CELESTIA_PORT}545%g;
s%:8546%:${CELESTIA_PORT}546%g;
s%:6065%:${CELESTIA_PORT}065%g" $HOME/.celestia-app/config/app.toml
```

**set custom ports in config.toml file**
```
sed -i.bak -e "s%:26658%:${CELESTIA_PORT}658%g;
s%:26657%:${CELESTIA_PORT}657%g;
s%:6060%:${CELESTIA_PORT}060%g;
s%:26656%:${CELESTIA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CELESTIA_PORT}656\"%;
s%:26660%:${CELESTIA_PORT}660%g" $HOME/.celestia-app/config/config.toml
```

**config pruning**
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.celestia-app/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.celestia-app/config/app.toml
```

**set minimum gas price, enable prometheus and disable indexing**
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.002utia"|g' $HOME/.celestia-app/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.celestia-app/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.celestia-app/config/config.toml
```

**create service file**
```
sudo tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
[Unit]
Description=Celestia node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.celestia-app
ExecStart=$(which celestia-appd) start --home $HOME/.celestia-app
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

**reset and download snapshot**
```
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
if curl -s --head curl https://server-4.itrocket.net/testnet/celestia/celestia_2024-11-04_3069552_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-4.itrocket.net/testnet/celestia/celestia_2024-11-04_3069552_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.celestia-app
    else
  echo "no snapshot found"
fi
```

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f
Automatic Installation
pruning: custom: 100/0/19 | indexer: null

source <(curl -s https://itrocket.net/api/testnet/celestia/autoinstall/)
Create wallet
# to create a new wallet, use the following command. don’t forget to save the mnemonic
celestia-appd keys add $WALLET

# to restore exexuting wallet, use the following command
celestia-appd keys add $WALLET --recover

# save wallet and validator address
WALLET_ADDRESS=$(celestia-appd keys show $WALLET -a)
VALOPER_ADDRESS=$(celestia-appd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile

# check sync status, once your node is fully synced, the output from above will print "false"
celestia-appd status 2>&1 | jq 

# before creating a validator, you need to fund your wallet and check balance
celestia-appd query bank balances $WALLET_ADDRESS 
Node Sync Status Checker
#!/bin/bash
rpc_port=$(grep -m 1 -oP '^laddr = "\K[^"]+' "$HOME/.celestia-app/config/config.toml" | cut -d ':' -f 3)
while true; do
  local_height=$(curl -s localhost:$rpc_port/status | jq -r '.result.sync_info.latest_block_height')
  network_height=$(curl -s https://celestia-testnet-rpc.itrocket.net/status | jq -r '.result.sync_info.latest_block_height')

  if ! [[ "$local_height" =~ ^[0-9]+$ ]] || ! [[ "$network_height" =~ ^[0-9]+$ ]]; then
    echo -e "\033[1;31mError: Invalid block height data. Retrying...\033[0m"
    sleep 5
    continue
  fi

  blocks_left=$((network_height - local_height))
  if [ "$blocks_left" -lt 0 ]; then
    blocks_left=0
  fi

  echo -e "\033[1;33mNode Height:\033[1;34m $local_height\033[0m \033[1;33m| Network Height:\033[1;36m $network_height\033[0m \033[1;33m| Blocks Left:\033[1;31m $blocks_left\033[0m"

  sleep 5
done
Create validator
Moniker
Identity
Details
I love blockchain ❤️
Amount, utia
1000000
Commission rate
0.1
Commission max rate
0.2
Commission max change rate
0.01
Website
celestia-appd tx staking create-validator \
--amount 1000000utia \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(celestia-appd tendermint show-validator) \
--moniker "test" \
--identity "" \
--website "" \
--details "I love blockchain ❤️" \
--chain-id mocha-4 \
--gas auto --gas-adjustment 1.5 --gas-prices 0.005utia \
-y
Monitoring
If you want to have set up a monitoring and alert system use our cosmos nodes monitoring guide with tenderduty

Security
To protect you keys please don`t share your privkey, mnemonic and follow basic security rules

Set up ssh keys for authentication
You can use this guide to configure ssh authentication and disable password authentication on your server

Firewall security
Set the default to allow outgoing connections, deny all incoming, allow ssh and node p2p port

sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${CELESTIA_PORT}656/tcp
sudo ufw enable
Delete node
sudo systemctl stop celestia-appd
sudo systemctl disable celestia-appd
sudo rm -rf /etc/systemd/system/celestia-appd.service
sudo rm $(which celestia-appd)
sudo rm -rf $HOME/.celestia-app
sed -i "/CELESTIA_/d" $HOME/.bash_profile
