UnUniFi validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/UnUniFi/chain)
##### Explorer:
> https://ununifi.io/explorer/
##### Requirements
Validator Node Server
- OS: Ubuntu 20.04
- Memory: 8 GB or more
- Storage: SSD 160 GB or more
- The following ports: `26656` must be open for peer to peer communication between nodes.

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
ver="1.18.2" && \
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
git clone https://github.com/UnUniFi/chain chain_repo  
cd chain_repo
git checkout main
git pull
make install
ununifid version
```
### Configuration of Shell Variables
```Bash
echo "export CHAIN_REPO=https://github.com/UnUniFi/chain" >> ~/.bash_profile
echo "export CHAIN_REPO_BRANCHE=main" >> ~/.bash_profile
echo "export TARGET=ununifid" >> ~/.bash_profile
echo "export TARGET_HOME=.ununifi" >> ~/.bash_profile
# This value is example of mainnet.
echo "export CHAIN_ID=ununifi-beta-v1" >> ~/.bash_profile
echo "export GENESIS_FILE_URL=https://raw.githubusercontent.com/UnUniFi/network/main/launch/ununifi-beta-v1/genesis.json" >> ~/.bash_profile
echo "export SETUP_NODE_CONFIG_ENV=TRUE" >> ~/.bash_profile
echo "export SETUP_NODE_ENV=TRUE" >> ~/.bash_profile
echo "export SETUP_NODE_MASTER=TRUE" >> ~/.bash_profile
echo "export DAEMON_NAME=$TARGET" >> ~/.bash_profile
# This value will be different for each node.
echo "export MONIKER=<your-moniker>" >> ~/.bash_profile
echo "export DAEMON_HOME=$HOME/.ununifi" >> ~/.bash_profile
echo "export DAEMON_ALLOW_DOWNLOAD_BINARIES=true" >> ~/.bash_profile
echo "export DAEMON_LOG_BUFFER_SIZE=512" >> ~/.bash_profile
echo "export DAEMON_RESTART_AFTER_UPGRADE=true" >> ~/.bash_profile
echo "export UNSAFE_SKIP_BACKUP=true" >> ~/.bash_profile
source ~/.bash_profile
```
### Init app
```Bash
ununifid init "$MONIKER" --chain-id $CHAIN_ID
```
### Download genesis
```Bash
rm ~/.ununifi/config/genesis.json && \
curl -L https://raw.githubusercontent.com/UnUniFi/network/main/launch/ununifi-beta-v1/genesis.json -o ~/.ununifi/config/genesis.json
```
### Config app
Setup seeds and peers
```Bash
seeds=""
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.quicksilverd/config/config.toml
peers="fa38d2a851de43d34d9602956cd907eb3942ae89@a.ununifi.cauchye.net:26656,404ea79bd31b1734caacced7a057d78ae5b60348@b.ununifi.cauchye.net:26656,1357ac5cd92b215b05253b25d78cf485dd899d55@[2600:1f1c:534:8f02:7bf:6b31:3702:2265]:26656,25006d6b85daeac2234bcb94dafaa73861b43ee3@[2600:1f1c:534:8f02:a407:b1c6:e8f5:94b]:26656,caf792ed396dd7e737574a030ae8eabe19ecdf5c@[2600:1f1c:534:8f02:b0a4:dbf6:e50b:d64e]:26656,796c62bb2af411c140cf24ddc409dff76d9d61cf@[2600:1f1c:534:8f02:ca0e:14e9:8e60:989e]:26656,cea8d05b6e01188cf6481c55b7d1bc2f31de0eed@[2600:1f1c:534:8f02:ba43:1f69:e23a:df6b]:26656"
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.ununifi/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.ununifi/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.ununifi/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.ununifi/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.ununifi/config/app.toml
```
Set min gas price
```Bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uguu\"/" $HOME/.ununifi/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.ununifi/config/app.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.ununifi/config/config.toml
```
### Reset chain data
```Bash
ununifid unsafe-reset-all --home $HOME/.ununifi
```
### Install cosmovisor (recommended approach):
```Bash
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```
### Set up folder structure
```Bash
mkdir -p $DAEMON_HOME/cosmovisor
mkdir -p $DAEMON_HOME/cosmovisor/genesis
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
mkdir -p $DAEMON_HOME/cosmovisor/upgrades
```
### Set up genesis binary
```Bash
cp ~/go/bin/$DAEMON_NAME $DAEMON_HOME/cosmovisor/genesis/bin
```
### Create service
```Bash
sudo tee /lib/systemd/system/cosmovisor.service > /dev/null <<EOF
[Unit]
Description=Cosmovisor daemon
After=network-online.target
[Service]
Environment="DAEMON_NAME=ununifid"
Environment="DAEMON_HOME=$HOME/.ununifi"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_LOG_BUFFER_SIZE=512"
Environment="UNSAFE_SKIP_BACKUP=true"
User=$USER
ExecStart=$HOME/go/bin/cosmovisor start
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
sudo systemctl enable cosmovisor && \
sudo systemctl start cosmovisor && sudo journalctl -u cosmovisor -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```
### Create wallet (!Safe your mnemonic)
```Bash
ununifid keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
ununifid keys add <wallet_name> --recover
```
### Check balance
```Bash
ununifid query bank balances <wallet_address>
```
### Create validator
```Bash
ununifid tx staking create-validator \
  --amount 1000000uguu \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(ununifid tendermint show-validator) \
  --moniker "$MONIKER" \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uguu \
  --from <wallet_name>
```
### Show address validator
```Bash
ununifid  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
ununifid  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
ununifid status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
ununifid status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
ununifid tendermint show-node-id
```
##### Get wallet balance
```Bash
ununifid query bank balances <wallet_name>
```
##### Transfer funds
```Bash
ununifid tx bank send <wallet_address> <to_wallet_address> 10000000uguu
```
##### Voting
```Bash
ununifid tx gov vote 1 yes --from <wallet_name> --chain-id=$CHAIN_ID
```
##### Delegate stake
```Bash
ununifid tx staking delegate <valoper_address> 10000000uguu --from=<wallet_name> --chain-id=$CHAIN_ID --gas-prices 0.025uguu
```
##### Withdraw all rewards
```Bash
ununifid tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=$CHAIN_ID --gas-prices 0.025uguu
```
##### Withdraw rewards with commision
```Bash
ununifid tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id $CHAIN_ID --gas-prices 0.025uguu
```
##### Edit validator
```Bash
ununifid tx staking edit-validator \
--moniker "$MONIKER" \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id $CHAIN_ID \
--from=<wallet_name>
```
##### Unjail validator
```Bash
ununifid tx slashing unjail \
  --broadcast-mode=block \
  --from=<wallet_name> \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025uguu
```
##### Delete node
```Bash
sudo systemctl stop ununifid
sudo systemctl disable ununifid
sudo rm /etc/systemd/system/ununifid* -rf
sudo rm $(which ununifid) -rf
sudo rm $HOME/.ununifid* -rf
sudo rm $HOME/ununifi -rf
```
