Paloma validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/palomachain/paloma)
##### Explorer:
> https://paloma.explorers.guru/

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
### Install go
```Bash
ver="1.18.2"
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export GOROOT=/usr/local/go" >> ~/.bash_profile && \
echo "export GOPATH=$HOME/go" >> ~/.bash_profile && \
echo "export GO111MODULE=on" >> ~/.bash_profile && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```
### Download and build binaries
```Bash
cd $HOME
wget -qO - https://github.com/palomachain/paloma/releases/download/v0.2.4-prealpha/paloma_0.2.4-prealpha_Linux_x86_64.tar.gz | \
sudo tar -C /usr/local/bin -xvzf - palomad
sudo chmod +x /usr/local/bin/palomad
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/raw/main/api/libwasmvm.x86_64.so
```
### Init app
```Bash
palomad init <moniker> --chain-id paloma-testnet-5
```
### Download genesis
```Bash
wget -qO $HOME/.paloma/config/genesis.json "https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-5/genesis.json"
```
### Download addrbook
```Bash
wget -qO $HOME/.paloma/config/addrbook.json "https://raw.githubusercontent.com/palomachain/testnet/master/paloma-testnet-5/addrbook.json"
```
### Set chain and keyring-backend to config
```Bash
palomad config chain-id paloma-testnet-5
palomad config keyring-backend file
```
### Config app
Setup seeds and peers
```Bash
seeds=""
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.paloma/config/config.toml
peers="b159364b4e6a3036c36ef6c7c690c5fbc81fa9c4@65.108.71.92:54056,e1efddf3b39f1953590f8264d30d71d1a1313061@164.90.134.139:26656,57bf3a71ea63bf5260bb00a8324c4567214b6f1f@38.242.199.201:36416,3a06e1d98f831963a09a16561c4125e4eec5ed06@195.3.223.33:30656,701d5f9826d33a15f34c50d76c69494027efbd16@137.184.32.223:36416,94d0adc7e711c0a9a6eb6479cd4e940dbf4be27d@65.21.181.135:36656,7e93f6409ade895fe301b502d6fb9dfb96343a34@135.125.5.34:54056,030b255fa1575f2b3b0c7f23babddb4e1daeafb1@134.209.254.198:28656"
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.paloma/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.paloma/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.paloma/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.paloma/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.paloma/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.paloma/config/config.toml
```
Set min gas
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0grain\"/" $HOME/.paloma/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.paloma/config/app.toml
```
Enable state sync (optional)
```Bash
SNAP_RPC=

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.paloma/config/config.toml
```
Change port (optional)
```Bash
PALOMA_PORT=
echo "export PALOMA_PORT=${PALOMA_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PALOMA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PALOMA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PALOMA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PALOMA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PALOMA_PORT}660\"%" $HOME/.paloma/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PALOMA_PORT}317\"%; s%^address = \":8080\"%address = \":${PALOMA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PALOMA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PALOMA_PORT}091\"%" $HOME/.paloma/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${PALOMA_PORT}657\"%" $HOME/.paloma/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${PALOMA_PORT}656\"/" $HOME/.paloma/config/config.toml
```
Change laddr adress 127.0.0.1 to 0.0.0.0 (optional)
```Bash
PALOMA_PRPC=$(grep -A 3 "\[rpc\]" ~/.paloma/config/config.toml | egrep -o ":[0-9]+") && \
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:$(PALOMA_PRPC)\"%laddr = \"tcp://0.0.0.0:$(PALOMA_PRPC)\"%" $HOME/.paloma/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.paloma/config/config.toml
```
### Reset chain data
```Bash
palomad tendermint unsafe-reset-all --home $HOME/.paloma
```
### Create service
```Bash
sudo tee /etc/systemd/system/palomad.service > /dev/null <<EOF
[Unit]
Description=PalomaNode
After=network-online.target

[Service]
User=$USER
ExecStart=$(which palomad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```Bash
sudo systemctl restart systemd-journald && \
sudo systemctl daemon-reload && \
sudo systemctl enable palomad && \
sudo systemctl restart palomad && sudo journalctl -u palomad -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
palomad status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
palomad keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
palomad keys add <wallet_name> --recover
```
### Check balance
```Bash
palomad query bank balances <wallet_address>
```
### Create validator
```Bash
palomad tx staking create-validator \
  --amount 100000000000ugrain \
  --pubkey  $(palomad tendermint show-validator) \
  --from <wallet_name> \
  --commission-max-change-rate "0.01" \
  --gas 10000000 \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --moniker <moniker> \
  --node "tcp://164.90.134.139:26656" \
  --chain-id paloma-testnet-5 \
  --yes \
  -b block
```
### Show address validator
```Bash
palomad  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
palomad  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
palomad status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
palomad status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
palomad tendermint show-node-id
```
##### Get wallet balance
```Bash
palomad query bank balances <wallet_name>
```
##### Transfer funds
```Bash
palomad tx bank send <wallet_address> <to_wallet_address> 10000000ugrain
```
##### Voting
```Bash
palomad tx gov vote 1 yes --from <wallet_name> --chain-id=paloma-testnet-5
```
##### Delegate stake
```Bash
palomad tx staking delegate <valoper_address> 10000000ugrain--from=<wallet_name> --chain-id=paloma-testnet-5 --gas=auto
```
##### Withdraw all rewards
```Bash
palomad tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=paloma-testnet-5 --gas=auto
```
##### Withdraw rewards with commision
```Bash
palomad tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=paloma-testnet-5 --gas=auto
```
##### Edit validator
```Bash
palomad tx staking edit-validator \
--from=<wallet_name> \
--website="web" \
--identity="identity" \
--details="details" \
--chain-id=paloma-testnet-5 \
--node cat "$HOME/.paloma/config/config.toml" \
| grep -oPm1 "(?<=^laddr = \")([^%]+)(?=\")"
```
##### Unjail validator
```Bash
palomad tx slashing unjail \
  --broadcast-mode=block \
  --from=<wallet_name> \
  --chain-id=paloma-testnet-5 \
  --gas=auto
```
##### Delete node
```Bash
sudo systemctl stop palomad && \
sudo systemctl disable palomad && \
sudo rm /etc/systemd/system/paloma* -rf && \
sudo rm $(which palomad) -rf && \
sudo rm $HOME/.paloma* -rf && \
sudo rm $HOME/paloma -rf
```
