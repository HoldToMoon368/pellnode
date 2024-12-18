# pellnode
Guilde for run PELL Validator

🟩
PELL NETWORK
Official Document: https://docs.pell.network/

# Hardware requirements
We recommend the following hardware specifications:

Ubuntu 22.04

Number of CPUs: 4
RAM: 8GB
NVMe SSD, SSD: 500GB

1. Update & install dependencies

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 aria2 pv gcc unzip -y
```
2. Install Go

```
cd $HOME
VER="1.22.2"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
go version
```

3. Set Vars

```
MONIKER="Moniker_name"
echo "export MONIKER=$MONIKER" >> $HOME/.bash_profile
echo "export PELL_CHAIN_ID="ignite_186-1"" >> $HOME/.bash_profile
echo "export PELL_PORT="22"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

4. Download Binary

```
cd $HOME
wget -O $HOME/pellcored https://github.com/0xPellNetwork/network-config/releases/download/v1.0.20-ignite/pellcored-v1.0.20-linux-amd64
export LD_LIBRARY_PATH=~/.pellcored/lib
mkdir -p $LD_LIBRARY_PATH
wget -O "$LD_LIBRARY_PATH/libwasmvm.x86_64.so" https://github.com/CosmWasm/wasmvm/releases/download/v2.1.2/libwasmvm.x86_64.so
echo "export LD_LIBRARY_PATH=$HOME/.pellcored/lib:$LD_LIBRARY_PATH" >> $HOME/.bash_profile
chmod +x pellcored
cp pellcored $HOME/go/bin/pellcored
source $HOME/.bash_profile
pellcored version
```

5. Config and Init app

```
pellcored init $MONIKER --chain-id $PELL_CHAIN_ID
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${PELL_PORT}657\"|" $HOME/.pellcored/config/client.toml
sed -i -e "s|^keyring-backend *=.*|keyring-backend = \"os\"|" $HOME/.pellcored/config/client.toml
sed -i -e "s|^chain-id *=.*|chain-id = \"ignite_186-1\"|" $HOME/.pellcored/config/client.toml
```

6. Download Genesis file and Addrbook

```
wget -O $HOME/.pellcored/config/genesis.json https://pell.j-node.net/genesis.json
wget -O $HOME/.pellcored/config/addrbook.json https://pell.j-node.net/addrbook.json
```

7. Set custom port (Optional)

```
sed -i.bak -e "s%:1317%:${PELL_PORT}317%g;
s%:8080%:${PELL_PORT}080%g;
s%:9090%:${PELL_PORT}090%g;
s%:9091%:${PELL_PORT}091%g;
s%:8545%:${PELL_PORT}545%g;
s%:8546%:${PELL_PORT}546%g;
s%:6065%:${PELL_PORT}065%g" $HOME/.pellcored/config/app.toml
sed -i.bak -e "s%:26658%:${PELL_PORT}658%g;
s%:26657%:${PELL_PORT}657%g;
s%:6060%:${PELL_PORT}060%g;
s%:26656%:${PELL_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${PELL_PORT}656\"%;
s%:26660%:${PELL_PORT}660%g" $HOME/.pellcored/config/config.toml
```

8. Config Seeds and Peers

```
SEEDS="d52c32a6a8510bdf0d33909008041b96d95c8408@34.87.39.12:26656,9b955d07f05b02b3d622f9cb7a0e6cfecd719985@34.87.47.193:26656"
PEERS="d52c32a6a8510bdf0d33909008041b96d95c8408@34.87.39.12:26656,9b955d07f05b02b3d622f9cb7a0e6cfecd719985@34.87.47.193:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.pellcored/config/config.toml
```

9. Config Pruning & indexer

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.pellcored/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.pellcored/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pellcored/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.pellcored/config/app.toml
```
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00001apell\"/" $HOME/.pellcored/config/app.toml
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.pellcored/config/config.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.pellcored/config/config.toml
```

10. Create service file

```
sudo tee /etc/systemd/system/pellcored.service > /dev/null <<EOF
[Unit]
Description=Pell-network-testnet
After=network-online.target

[Service]
User=$USER
Environment="LD_LIBRARY_PATH=/root/.pellcored/lib"
ExecStart=$(which pellcored) start --home $HOME/.pellcored
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

LIVE PEES

```
PEERS=$(curl -s -X POST https://rpc-pell.j-node.net -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"net_info","params":[],"id":1}' | jq -r '.result.peers[] | select(.connection_status.SendMonitor.Active == true) | "\(.node_info.id)@\(if .node_info.listen_addr | contains("0.0.0.0") then .remote_ip + ":" + (.node_info.listen_addr | sub("tcp://0.0.0.0:"; "")) else .node_info.listen_addr | sub("tcp://"; "") end)"' | tr '\n' ',' | sed 's/,$//' | awk '{print "\"" $0 "\""}')
sed -i "s/^persistent_peers *=.*/persistent_peers = $PEERS/" "$HOME/.pellcored/config/config.toml"
    if [ $? -eq 0 ]; then
echo -e "Configuration file updated successfully with new peers"
    else
echo "Failed to update configuration file."
fi
```

```
sudo systemctl daemon-reload
sudo systemctl enable pellcored
sudo systemctl start pellcored && sudo journalctl -fu pellcored -o cat
```

Download snapshot

```
cd $HOME
rm -f pell_snapshot.lz4
aria2c -x 16 -s 16 -k 1M https://pell.j-node.net/pell_snapshot.lz4
Reset data
```

```
lz4 -dc pell_snapshot.lz4 | pv | tar -xf - -C $HOME/.pellcored
```

```
sudo systemctl restart pellcored && sudo systemctl status pellcored
```

Enable service and start node

```
sudo systemctl daemon-reload
sudo systemctl enable pellcored
sudo systemctl start pellcored && sudo journalctl -fu pellcored -o cat
```

# Check status
```
pellcored query status | jq
```

```
pellcored query status | jq '{ latest_block_height: .sync_info.latest_block_height, catching_up: .sync_info.catching_up }'
```

# Chekc Block sync left: (and waiting sync to 0)
```
while true; do
 local_height=$(pellcored query status | jq -r '.sync_info.latest_block_height');
  network_height=$(curl -s https://rpc-pell.j-node.net/status | jq -r '.result.sync_info.latest_block_height')
  blocks_left=$((network_height - local_height));

  echo -e "\033[1;38mYour node height:\033[0m \033[1;34m$local_height\033[0m | \033[1;35mNetwork height:\033[0m \033[1;36m$network_height\033[0m | \033[1;29mBlocks left:\033[0m \033[1;31m$blocks_left\033[0m";
  sleep 5;
done
```

Create validator.json file

```
cd $HOME
echo "{
    \"pubkey\":{
        \"@type\":\"/cosmos.crypto.ed25519.PubKey\",
        \"key\":\"$(pellcored query status | jq -r '.validator_info.pub_key.value')\"
    },
    \"amount\": \"49pell\",
    \"moniker\": \"YOUR_MONIKER\",
    \"identity\": \"YOUR_KEY_BASE\",
    \"website\": \"YOUR_WEBSITE\",
    \"security\": \"YOUR_CONTACT\", 
    \"details\": \"I love blockchain 🚀\",
    \"commission-rate\": \"0.05\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
```

Create a validator with .json file

```
pellcored tx staking create-validator validator.json \
    --from "wallet_name" \
    --chain-id ignite_186-1 \
    --gas auto \
    --gas-adjustment 1.5 \
    --gas-prices 33apell \
    -y
```

#Caution
Backup the validator key:

File location: 
/root/.pellcored/config/priv_validator_key.json

Copy this file to your local machine.

Store it carefully; this is the most crucial key for your validator.

DONE!
