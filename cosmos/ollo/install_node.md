Sei validator node
=
##### Official documentation:
> [Validator setup instructions](https://docs.ollo.zone/validators/running_a_node)
##### Explorer:
> https://explorer.nodestake.top/hypersign-testnet

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils ncdu git jq liblz4-tool -y
```
### Install go
```Bash
ver="1.18.3"
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
git clone https://github.com/OLLO-Station/ollo
cd ollo
make install
```
### Init app
```Bash
ollod init <moniker> --chain-id "ollo-testnet-0"
```
### Download genesis
```Bash
curl https://raw.githubusercontent.com/OllO-Station/ollo/master/networks/ollo-testnet-0/genesis.json > ~/.ollo/config/genesis.json
```
### Config app
Setup seeds and peers
```Bash
seeds=""
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.ollo/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.ollo/config/config.toml
```
Set min gas price
```Bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utollo\"/;" $HOME/.ollo/config/app.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.ollo/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.ollo/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.ollo/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.ollo/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.ollo/config/config.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.ollo/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.ollo/config/config.toml
```
Change port (optional)
```Bash
OLLO_PORT=
echo "export OLLO_PORT=${OLLO_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OLLO_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${OLLO_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OLLO_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OLLO_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OLLO_PORT}660\"%" $HOME/.ollo/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OLLO_PORT}317\"%; s%^address = \":8080\"%address = \":${OLLO_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OLLO_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OLLO_PORT}091\"%" $HOME/.ollo/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${OLLO_PORT}657\"%" $HOME/.ollo/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${OLLO_PORT}656\"/" $HOME/.ollo/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.ollo/config/config.toml
```
### Reset chain data
```Bash
ollod tendermint unsafe-reset-all --home $HOME/.ollo
```
### Create service
```Bash
sudo tee /etc/systemd/system/ollod.service > /dev/null <<EOF
[Unit]
Description=sei
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ollod) start --home $HOME/.ollo
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
sudo systemctl enable ollod && \
sudo systemctl restart ollod && sudo journalctl -u ollod -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
ollod status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
ollod keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
ollod keys add <wallet_name> --recover
```
### Check balance
```Bash
ollod query bank balances <wallet_address>
```
### Create validator
```Bash
ollod tx staking create-validator \
--chain-id "ollo-testnet-0" \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation 1 \
--amount 1000000utollo \
--pubkey $(ollod tendermint show-validator) \
--moniker "<moniker>" \
--from <wallet_name>
```
### Show address validator
```Bash
ollod  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
ollod  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
ollod status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
ollod status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
ollod tendermint show-node-id
```
##### Get wallet balance
```Bash
ollod query bank balances <wallet_name>
```
##### Transfer funds
```Bash
ollod tx bank send <wallet_address> <to_wallet_address> 10000000utollo --fees 5555utollo -y
```
##### Voting
```Bash
ollod tx gov vote <number_proposal> <yes/no> --from <wallet_name> --chain-id=ollo-testnet-0
```
##### Delegate stake
```Bash
ollod tx staking delegate <valoper_address> 10000000utollo --from=<wallet_name> --chain-id=ollo-testnet-0 --gas=auto
```
##### Withdraw all rewards
```Bash
ollod tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=ollo-testnet-0 --gas=auto
```
##### Withdraw rewards with commision
```Bash
ollod tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=ollo-testnet-0 --gas=auto
```
##### Edit validator
```Bash
ollod tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=ollo-testnet-0 \
--from=<wallet_name>
```
##### Unjail validator
```Bash
ollod tx slashing unjail \
  --from=<wallet_name> \
  --chain-id=ollo-testnet-0
```
##### Delete node
```Bash
sudo systemctl stop ollod && \
sudo systemctl disable ollod && \
sudo rm /etc/systemd/system/ollo* -rf && \
sudo rm $(which ollod) -rf && \
sudo rm $HOME/.ollo* -rf && \
sudo rm $HOME/ollo -rf
```
