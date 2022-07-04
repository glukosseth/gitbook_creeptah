Quicksilver validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/ingenuity-build/testnets)
##### Explorer:
> https://quicksilver.explorers.guru/

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
rm quicksilver -rf
git clone https://github.com/ingenuity-build/quicksilver.git --branch v0.4.1
cd quicksilver
make build
sudo chmod +x ./build/quicksilverd && sudo mv ./build/quicksilverd /usr/local/bin/quicksilverd
```
### Init app
```Bash
quicksilverd init <moniker> --chain-id killerqueen-1
```
### Download genesis
```Bash
wget -qO $HOME/.quicksilverd/config/genesis.json "https://raw.githubusercontent.com/ingenuity-build/testnets/main/killerqueen/genesis.json"
```
### Config app
Setup chain and keyring-backend to config
```Bash
quicksilverd config chain-id killerqueen-1
quicksilverd config keyring-backend test
```
Setup seeds and peers
```Bash
seeds="dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.killerqueen-1.quicksilver.zone:26656"
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.quicksilverd/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.quicksilverd/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.quicksilverd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.quicksilverd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.quicksilverd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.quicksilverd/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.quicksilverd/config/config.toml
```
Set min gas price
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uqck\"/" $HOME/.quicksilverd/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.quicksilverd/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.quicksilverd/config/config.toml
```
Change port (optional)
```Bash
QUICKSILVER_PORT=
quicksilverd config node tcp://localhost:${QUICKSILVER_PORT}657
echo "export QUICKSILVER_PORT=${QUICKSILVER_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${QUICKSILVER_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${QUICKSILVER_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${QUICKSILVER_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${QUICKSILVER_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${QUICKSILVER_PORT}660\"%" $HOME/.quicksilverd/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${QUICKSILVER_PORT}317\"%; s%^address = \":8080\"%address = \":${QUICKSILVER_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${QUICKSILVER_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${QUICKSILVER_PORT}091\"%" $HOME/.quicksilverd/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${QUICKSILVER_PORT}657\"%" $HOME/.quicksilverd/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${QUICKSILVER_PORT}656\"/" $HOME/.quicksilverd/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.quicksilverd/config/config.toml
```
### Reset chain data
```Bash
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd
```
### Create service
```Bash
sudo tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
[Unit]
Description=quicksilver
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quicksilverd) start
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
sudo systemctl enable quicksilverd && \
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
quicksilverd status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
quicksilverd keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
quicksilverd keys add <wallet_name> --recover
```
### Check balance
```Bash
quicksilverd query bank balances <wallet_address>
```
### Create validator
```Bash
quicksilverd tx staking create-validator \
  --amount 1000000uqck \
  --from <wallet_name> \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(quicksilverd tendermint show-validator) \
  --moniker <moniker> \
  --chain-id killerqueen-1
```
### Show address validator
```Bash
quicksilverd  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
quicksilverd  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
quicksilverd status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
quicksilverd status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
quicksilverd tendermint show-node-id
```
##### Get wallet balance
```Bash
quicksilverd query bank balances <wallet_name>
```
##### Transfer funds
```Bash
quicksilverd tx bank send <wallet_address> <to_wallet_address> 10000000uqck
```
##### Voting
```Bash
quicksilverd tx gov vote 1 yes --from <wallet_name> --chain-id=killerqueen-1
```
##### Delegate stake
```Bash
quicksilverd tx staking delegate <valoper_address> 10000000uqck --from=<wallet_name> --chain-id=killerqueen-1 --gas=auto
```
##### Withdraw all rewards
```Bash
quicksilverd tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=killerqueen-1 --gas=auto
```
##### Withdraw rewards with commision
```Bash
quicksilverd tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=killerqueen-1 --gas=auto
```
##### Edit validator
```Bash
quicksilverd tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=killerqueen-1 \
--from=<wallet_name>
```
##### Unjail validator
```Bash
quicksilverd tx slashing unjail \
  --broadcast-mode=block \
  --from=<wallet_name> \
  --chain-id=killerqueen-1 \
  --gas=auto
```
##### Delete node
```Bash
sudo systemctl stop quicksilverd
sudo systemctl disable quicksilverd
sudo rm /etc/systemd/system/quicksilverd* -rf
sudo rm $(which quicksilverd) -rf
sudo rm $HOME/.quicksilverd* -rf
sudo rm $HOME/quicksilver -rf
```
