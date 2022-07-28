Stride validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/Stride-Labs/testnet)
##### Explorer:
> https://stride.explorers.guru/

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
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout c53f6c562d9d3e098aab5c27303f41ee055572cb
make build
chmod +x ./build/strided
sudo mv ./build/strided /usr/local/bin/strided
```
### Init app
```Bash
strided init <moniker> --chain-id STRIDE-1
```
### Download genesis
```Bash
wget -qO $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"
```
### Config app
Setup chain and keyring-backend to config
```Bash
strided config chain-id STRIDE-1
strided config keyring-backend test
```
Setup seeds and peers
```Bash
seeds="baee9ccc2496c2e3bebd54d369c3b788f9473be9@seedv1.poolparty.stridenet.co:26656"
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.stride/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.stride/config/config.toml
```
Set min gas price
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ustrd\"/" $HOME/.stride/config/app.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stride/config/config.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.stride/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stride/config/config.toml
```
Change port (optional)
```Bash
STRIDE_PORT=
echo "export STRIDE_PORT=${STRIDE_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${STRIDE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${STRIDE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${STRIDE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${STRIDE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${STRIDE_PORT}660\"%" $HOME/.stride/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${STRIDE_PORT}317\"%; s%^address = \":8080\"%address = \":${STRIDE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${STRIDE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${STRIDE_PORT}091\"%" $HOME/.stride/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${STRIDE_PORT}657\"%" $HOME/.stride/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${STRIDE_PORT}656\"/" $HOME/.stride/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.stride/config/config.toml
```
### Reset chain data
```Bash
strided tendermint unsafe-reset-all --home $HOME/.stride
```
### Create service
```Bash
sudo tee /etc/systemd/system/strided.service > /dev/null <<EOF
[Unit]
Description=strideNode
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start
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
sudo systemctl enable strided && \
sudo systemctl restart strided && sudo journalctl -u strided -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
strided status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
strided keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
strided keys add <wallet_name> --recover
```
### Check balance
```Bash
strided query bank balances <wallet_address>
```
### Create validator
```Bash
strided tx staking create-validator \
  --amount 10000000ustrd \
  --from <wallet_name> \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.09" \
  --min-self-delegation "1" \
  --pubkey $(strided tendermint show-validator) \
  --moniker <moniker> \
  --chain-id STRIDE-1
```
### Show address validator
```Bash
strided keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
strided query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
strided status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
strided status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
strided tendermint show-node-id
```
##### Get wallet balance
```Bash
strided query bank balances <wallet_name>
```
##### Transfer funds
```Bash
strided tx bank send <wallet_address> <to_wallet_address> 10000000ustride --chain-id STRIDE-1 -y
```
##### Voting
```Bash
strided tx gov vote <number_proposal> <yes/no> --from <wallet_name> --chain-id=STRIDE-1
```
##### Delegate stake
```Bash
strided tx staking delegate <valoper_address> 10000000ustride --from=<wallet_name> --chain-id=STRIDE-1 --gas=auto
```
##### Withdraw all rewards
```Bash
strided tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=STRIDE-1 --gas=auto
```
##### Withdraw rewards with commision
```Bash
strided tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=STRIDE-1 --gas=auto
```
##### Edit validator
```Bash
strided tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=STRIDE-1 \
--from=<wallet_name>
```
##### Unjail validator
```Bash
strided tx slashing unjail \
  --from=<wallet_name> \
  --chain-id=STRIDE-1
```
##### Delete node
```Bash
sudo systemctl stop strided && \
sudo systemctl disable strided && \
sudo rm /etc/systemd/system/stride* -rf && \
sudo rm $(which strided) -rf && \
sudo rm $HOME/.stride* -rf && \
sudo rm $HOME/stride -rf
```
