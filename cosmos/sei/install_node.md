Sei validator node
=
##### Official documentation:
> [Validator setup instructions](https://docs.seinetwork.io/nodes-and-validators/seinami-incentivized-testnet/joining-incentivized-testnet)
##### Explorer:
> https://sei.explorers.guru/

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
git clone https://github.com/sei-protocol/sei-chain.git && cd sei-chain
git checkout 1.0.6beta
make install
```
### Init app
```Bash
seid init <moniker> --chain-id atlantic-1
```
### Download genesis
```Bash
curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json > ~/.sei/config/genesis.json
```
### Config app
Setup seeds and peers
```Bash
seeds="df1f6617ff5acdc85d9daa890300a57a9d956e5e@sei-atlantic-1.seed.rhinostake.com:16660"
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.sei/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.sei/config/config.toml
```
Set min gas price
```Bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usei\"/;" $HOME/.sei/config/app.toml
```
Increase numbers of peers for connect, ex. persistens peers in `config.toml`
```Bash
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.sei/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.sei/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sei/config/config.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.sei/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.sei/config/config.toml
```
Change port (optional)
```Bash
SEI_PORT=
echo "export sei_PORT=${sei_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SEI_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${SEI_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SEI_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SEI_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SEI_PORT}660\"%" $HOME/.sei/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${SEI_PORT}317\"%; s%^address = \":8080\"%address = \":${SEI_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${SEI_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${SEI_PORT}091\"%" $HOME/.sei/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${SEI_PORT}657\"%" $HOME/.sei/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${SEI_PORT}656\"/" $HOME/.sei/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sei/config/config.toml
```
### Reset chain data
```Bash
seid tendermint unsafe-reset-all --home $HOME/.sei
```
### Create service
```Bash
sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF
[Unit]
Description=sei
After=network-online.target

[Service]
User=$USER
ExecStart=$(which seid) start --home $HOME/.sei
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
sudo systemctl enable seid && \
sudo systemctl restart seid && sudo journalctl -u seid -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
seid status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
seid keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
seid keys add <wallet_name> --recover
```
### Check balance
```Bash
seid query bank balances <wallet_address>
```
### Create validator
```Bash
seid tx staking create-validator \
--chain-id atlantic-1 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation 1 \
--amount 1000000usei \
--pubkey $(seid tendermint show-validator) \
--moniker "<moniker>" \
--from <wallet_name>
```
### Show address validator
```Bash
seid  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
seid  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
seid status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
seid status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
seid tendermint show-node-id
```
##### Get wallet balance
```Bash
seid query bank balances <wallet_name>
```
##### Transfer funds
```Bash
seid tx bank send <wallet_address> <to_wallet_address> 10000000usei --fees 5555usei -y
```
##### Voting
```Bash
seid tx gov vote <number_proposal> <yes/no> --from <wallet_name> --chain-id=atlantic-1
```
##### Delegate stake
```Bash
seid tx staking delegate <valoper_address> 10000000usei --from=<wallet_name> --chain-id=atlantic-1 --gas=auto
```
##### Withdraw all rewards
```Bash
seid tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=atlantic-1 --gas=auto
```
##### Withdraw rewards with commision
```Bash
seid tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=atlantic-1 --gas=auto
```
##### Edit validator
```Bash
seid tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=atlantic-1 \
--from=<wallet_name>
```
##### Unjail validator
```Bash
seid tx slashing unjail \
  --from=<wallet_name> \
  --chain-id=atlantic-1 \
  --gas=auto
```
##### Delete node
```Bash
sudo systemctl stop seid && \
sudo systemctl disable seid && \
sudo rm /etc/systemd/system/sei* -rf && \
sudo rm $(which seid) -rf && \
sudo rm $HOME/.sei* -rf && \
sudo rm $HOME/sei -rf
```
