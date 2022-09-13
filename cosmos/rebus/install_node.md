Rebus validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/rebuschain/rebus.mainnet/tree/master/reb_1111-1)
##### Explorer:
> https://explorer.nodestake.top/rebus

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git ncdu gcc jq chrony liblz4-tool -y
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
git clone https://github.com/rebuschain/rebus.core.git 
cd rebus.core && git checkout master
make install
```
### Init app
```Bash
rebusd init <moniker> --chain-id reb_1111-1
```
### Download genesis
```Bash
wget https://raw.githubusercontent.com/rebuschain/rebus.mainnet/master/reb_1111-1/genesis.zip
unzip -o genesis.zip -d $HOME/.rebusd/config/
rm genesis.zip
```
### Config app
Setup chain and keyring-backend to config
```Bash
rebusd config chain-id reb_1111-1
```
Setup seeds and peers
```Bash
seeds="e056318da91e77585f496333040e00e12f6941d1@51.83.97.166:26656"
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.rebusd/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.rebusd/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.rebusd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.rebusd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.rebusd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.rebusd/config/app.toml
```
Set min gas price
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0arebus\"/" $HOME/.rebusd/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.rebusd/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.rebusd/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.rebusd/config/config.toml
```
Set commit timeout
```Bash
timeout_commit="2s" && \
sed -i.bak -e "s/^timeout_commit *=.*/timeout_commit = \"$timeout_commit\"/" $HOME/.rebusd/config/config.toml
```
### Reset chain data
```Bash
rebusd tendermint unsafe-reset-all --home $HOME/.rebusd
```
### Create service
```Bash
sudo tee /etc/systemd/system/rebusd.service > /dev/null <<EOF
[Unit]
Description=rebus
After=network-online.target

[Service]
User=$USER
ExecStart=$(which rebusd) start --home $HOME/.rebusd
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
sudo systemctl enable rebusd && \
sudo systemctl restart rebusd && sudo journalctl -u rebusd -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
rebusd status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
rebusd keys add <wallet_name> --coin-type 118 --algo secp256k1
```
To recover your wallet using seed phrase (optional)
```Bash
rebusd keys add <wallet_name> --coin-type 118 --algo secp256k1 --recover
```
### Check balance
```Bash
rebusd query bank balances <wallet_address>
```
### Create validator
```Bash
rebusd tx staking create-validator \
--moniker="<moniker>" \
--amount=1000000000000000000arebus \
--pubkey=$(rebusd tendermint show-validator) \
--chain-id=reb_1111-1 \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.10 \
--min-self-delegation=1 \
--from=<wallet_name>
```
### Show address validator
```Bash
rebusd  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
rebusd  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
rebusd status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
rebusd status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
rebusd tendermint show-node-id
```
##### Get wallet balance
```Bash
rebusd query bank balances <wallet_name>
```
##### Transfer funds
```Bash
rebusd tx bank send <wallet_address> <to_wallet_address> 10000000arebus
```
##### Voting
```Bash
rebusd tx gov vote 1 yes --from <wallet_name> --chain-id=reb_1111-1
```
##### Delegate stake
```Bash
rebusd tx staking delegate <valoper_address> 10000000arebus --from=<wallet_name> --chain-id=reb_1111-1
```
##### Withdraw all rewards
```Bash
rebusd tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=reb_1111-1
```
##### Withdraw rewards with commision
```Bash
rebusd tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=reb_1111-1
```
##### Edit validator
```Bash
rebusd tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=reb_1111-1 \
--from=<wallet_name>
```
##### Unjail validator
```Bash
rebusd tx slashing unjail \
  --from=<wallet_name> \
  --chain-id=reb_1111-1
```
##### Delete node
```Bash
sudo systemctl stop rebusd
sudo systemctl disable rebusd
sudo rm /etc/systemd/system/rebusd* -rf
sudo rm $(which rebusd) -rf
sudo rm $HOME/.rebusd* -rf
sudo rm $HOME/rebus.core -rf
```
