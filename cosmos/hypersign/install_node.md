Hypersign Protocol mainnet validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/hypersign-protocol/networks/tree/master/testnet/jagrat#after-final-genesis-release)
##### Explorer:
> http://explorer.creeptah.xyz/hypersign

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git ncdu gcc unzip chrony liblz4-tool -y
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
git clone https://github.com/hypersign-protocol/hid-node.git 
cd hid-node
git checkout v0.1.0
make install
```
### Configuration of Shell Variables
```Bash
echo "export CHAIN_ID=jagrat" >> ~/.bash_profile
echo "export SETUP_NODE_CONFIG_ENV=TRUE" >> ~/.bash_profile
echo "export SETUP_NODE_ENV=TRUE" >> ~/.bash_profile
echo "export SETUP_NODE_MASTER=TRUE" >> ~/.bash_profile
echo "export DAEMON_NAME=hid-noded" >> ~/.bash_profile
echo "export MONIKER=<your-moniker>" >> ~/.bash_profile
echo "export WALLET=<wallet-mane>" >> ~/.bash_profile
echo "export DAEMON_HOME=$HOME/.hid-noded >> ~/.bash_profile
echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=true" >> ~/.bash_profile
echo "export DAEMON_LOG_BUFFER_SIZE=512" >> ~/.bash_profile
echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.bash_profile
echo "export UNSAFE_SKIP_BACKUP=true" >> ~/.bash_profile
source ~/.bash_profile
```
### Init app
```Bash
hid-noded init "$MONIKER" --chain-id $CHAIN_ID
```
### Download genesis
```Bash
wget https://raw.githubusercontent.com/rebuschain/rebus.mainnet/master/jagrat/genesis.zip
unzip -o genesis.zip -d $HOME/.hid-noded/config/
rm genesis.zip
```
### Config app
Setup chain to config
```Bash
hid-noded config chain-id $CHAIN_ID
```
Setup seeds and peers
```Bash
seeds=""
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.hid-noded/config/config.toml
peers=""
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.hid-noded/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.hid-noded/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.hid-noded/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.hid-noded/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.hid-noded/config/app.toml
```
Set min gas price
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.02uhid\"/" $HOME/.hid-noded/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.hid-noded/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.hid-noded/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.hid-noded/config/config.toml
```
### Reset chain data
```Bash
hid-noded tendermint unsafe-reset-all --home $HOME/.hid-noded
```
### Install cosmovisor (recommended approach):
```Bash
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.2.0/cosmovisor-v1.2.0-linux-amd64.tar.gz && \
tar -C /usr/local/bin/ -xzf cosmovisor-v1.2.0-linux-amd64.tar.gz
```
### Set up folder structure
```Bash
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
mkdir $DAEMON_HOME/cosmovisor/upgrades
```
### Set up genesis binary
```Bash
cp ~/go/bin/$DAEMON_NAME $DAEMON_HOME/cosmovisor/genesis/bin
```
### Create service
```Bash
sudo tee /lib/systemd/system/hided.service > /dev/null <<EOF
[Unit]
Description=Hypersign
After=network-online.target
[Service]
Environment="DAEMON_NAME=hid-noded"
Environment="DAEMON_HOME=$HOME/.hid-noded"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=always
RestartSec=3
LimitNOFILE=infinity
LimitNPROC=infinity
[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```Bash
sudo systemctl daemon-reload && \
sudo systemctl restart systemd-journald && \
sudo systemctl enable hided && \
sudo systemctl restart hided && sudo journalctl -u hided -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
hid-noded status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
hid-noded keys add $WALLET
```
To recover your wallet using seed phrase (optional)
```Bash
hid-noded keys add $WALLET --recover
```
### Check balance
```Bash
hid-noded query bank balances <wallet_address>
```
### Create validator
```Bash
hid-noded tx staking create-validator \
--from $WALLET \
--amount 1000000000000uhid \
--pubkey "$(hid-noded tendermint show-validator)" \
--chain-id $CHAIN_ID \
--moniker="$MONIKER" \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.2 \
--commission-rate=0.07 \
--min-self-delegation="500000000000" \
--details="XXXXXXXX" \
--security-contact="XXXXXXXX" \
--website="XXXXXXXX"
```
### Show address validator
```Bash
hid-noded  keys show $WALLET --bech val -a
```
### Check validator status
```Bash
hid-noded  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
hid-noded status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
hid-noded status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
hid-noded tendermint show-node-id
```
##### Get wallet balance
```Bash
hid-noded query bank balances <wallet_address>
```
##### Transfer funds
```Bash
hid-noded tx bank send <wallet_address> <to_wallet_address> 10000000uhid
```
##### Voting
```Bash
hid-noded tx gov vote 1 yes --from $WALLET --chain-id=$CHAIN_ID
```
##### Delegate stake
```Bash
hid-noded tx staking delegate <valoper_address> 10000000uhid --from=$WALLET --chain-id=$CHAIN_ID
```
##### Withdraw all rewards
```Bash
hid-noded tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID
```
##### Withdraw rewards with commision
```Bash
hid-noded tx distribution withdraw-rewards <valoper_address> --from=$WALLET --commission --chain-id=$CHAIN_ID
```
##### Edit validator
```Bash
hid-noded tx staking edit-validator \
--moniker=$MONIKER \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=$CHAIN_ID \
--from=$WALLET
```
##### Unjail validator
```Bash
hid-noded tx slashing unjail \
  --from=$WALLET \
  --chain-id=$CHAIN_ID
```
##### Delete node
```Bash
sudo systemctl stop hid-noded
sudo systemctl disable hid-noded
sudo rm /etc/systemd/system/hid-noded* -rf
sudo rm $(which hid-noded) -rf
sudo rm $HOME/.hid-node* -rf
sudo rm $HOME/hid-node -rf
```
