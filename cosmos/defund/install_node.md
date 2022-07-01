Defund validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/defund-labs/defund/blob/main/testnet/private/validators.md)
##### Explorer:
> https://defund.explorers.guru/

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
git clone https://github.com/defund-labs/defund
cd defund
make install
```
### Init app
```Bash
defundd init <moniker> --chain-id defund-private-1
```
### Download genesis
```Bash
wget -qO $HOME/.defund/config/genesis.json "https://raw.githubusercontent.com/defund-labs/defund/163e2669b6870aa26b73d843312b22c9948b29c6/testnet/private/genesis.json"
```
### Config app
Setup seeds and peers
```Bash
seeds="1b3e596531dd8f36363b13339beed2364900e4c6@104.131.41.157:26656"
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.defund/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.defund/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.defund/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.defund/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.defund/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.defund/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.defund/config/config.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.defund/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.defund/config/config.toml
```
Change port (optional)
```Bash
DEFUND_PORT=
echo "export DEFUND_PORT=${DEFUND_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${DEFUND_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${DEFUND_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${DEFUND_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${DEFUND_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${DEFUND_PORT}660\"%" $HOME/.defund/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${DEFUND_PORT}317\"%; s%^address = \":8080\"%address = \":${DEFUND_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${DEFUND_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${DEFUND_PORT}091\"%" $HOME/.defund/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${DEFUND_PORT}657\"%" $HOME/.defund/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${DEFUND_PORT}656\"/" $HOME/.defund/config/config.toml
```
Change laddr adress 127.0.0.1 to 0.0.0.0 (optional)
```Bash
DEFUND_PRPC=$(grep -A 3 "\[rpc\]" ~/.defund/config/config.toml | egrep -o ":[0-9]+") && \
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:$(DEFUND_PRPC)\"%laddr = \"tcp://0.0.0.0:$(DEFUND_PRPC)\"%" $HOME/.defund/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.defund/config/config.toml
```
### Reset chain data
```Bash
defundd tendermint unsafe-reset-all --home $HOME/.defund
```
### Create service
```Bash
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=defundd
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```Bash
sudo systemctl restart systemd-journald && \
sudo systemctl daemon-reload && \
sudo systemctl enable defundd && \
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
defundd status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
defundd keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
defundd keys add <wallet_name> --recover
```
### Check balance
```Bash
defundd query bank balances <wallet_address>
```
### Create validator
```Bash
defundd tx staking create-validator \
  --amount 1000000ufetf \
  --from <wallet_name> \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(defundd tendermint show-validator) \
  --moniker <moniker> \
  --chain-id defund-private-1
```
### Show address validator
```Bash
defundd  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
defundd  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
defundd status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
defundd status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
defundd tendermint show-node-id
```
##### Get wallet balance
```Bash
defundd query bank balances <wallet_name>
```
##### Transfer funds
```Bash
defundd tx bank send <wallet_address> <to_wallet_address> 10000000ufetf
```
##### Voting
```Bash
defundd tx gov vote 1 yes --from <wallet_name> --chain-id=defund-private-1
```
##### Delegate stake
```Bash
defundd tx staking delegate <valoper_address> 10000000ufetf --from=<wallet_name> --chain-id=defund-private-1 --gas=auto
```
##### Withdraw all rewards
```Bash
defundd tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=defund-private-1 --gas=auto
```
##### Withdraw rewards with commision
```Bash
defundd tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=defund-private-1 --gas=auto
```
##### Edit validator
```Bash
defundd tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=defund-private-1 \
--from=<wallet_name>
```
##### Unjail validator
```Bash
defundd tx slashing unjail \
  --broadcast-mode=block \
  --from=<wallet_name> \
  --chain-id=defund-private-1 \
  --gas=auto
```
##### Delete node
```Bash
sudo systemctl stop defundd && \
sudo systemctl disable defundd && \
sudo rm /etc/systemd/system/defund* -rf && \
sudo rm $(which defundd) -rf && \
sudo rm $HOME/.defund* -rf && \
sudo rm $HOME/defund -rf
```
