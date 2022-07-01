StaFi validator node
=
##### Official documentation:
> [Validator setup instructions](https://docs.stafihub.io/welcome-to-stafihub/developer/getting-started/join-the-public-testnet)
##### Explorer:
> https://testnet-explorer.stafihub.io/stafi-hub-testnet

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git ncdu git jq liblz4-tool -y
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
git clone --depth 1 --branch public-testnet-v2 https://github.com/stafihub/stafihub
cd stafihub
make install
```
### Init app
```Bash
stafihubd init <moniker> --chain-id stafihub-public-testnet-2
```
### Download genesis
```Bash
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-2/genesis.json"
```
### Config app
Set up seeds and peers
```Bash
seeds=""
sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.stafihub/config/config.toml
peers="4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```
Set up gRPC and API
```Bash
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stafihub/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stafihub/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stafihub/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stafihub/config/app.toml
```
Disable indexing
```Bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stafihub/config/config.toml
```
Set min gas price
```Bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.stafihub/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stafihub/config/config.toml
```
Change port (optional)
```Bash
STAFI_PORT=
stafihubd config node tcp://localhost:${STAFI_PORT}657
echo "export STAFI_PORT=${STAFI_PORT}" >> ~/.bash_profile && \
source ~/.bash_profile

sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${STAFI_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${STAFI_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${STAFI_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${STAFI_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${STAFI_PORT}660\"%" $HOME/.stafihub/config/config.toml && \
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${STAFI_PORT}317\"%; s%^address = \":8080\"%address = \":${STAFI_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${STAFI_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${STAFI_PORT}091\"%" $HOME/.stafihub/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${STAFI_PORT}657\"%" $HOME/.stafihub/config/client.toml && \
external_address=$(wget -qO- eth0.me) && \
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${STAFI_PORT}656\"/" $HOME/.stafihub/config/config.toml
```
Change laddr adress 127.0.0.1 to 0.0.0.0 (optional)
```Bash
STAFI_PRPC=$(grep -A 3 "\[rpc\]" ~/.stafihubd/config/config.toml | egrep -o ":[0-9]+") && \
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:$(STAFI_PRPC)\"%laddr = \"tcp://0.0.0.0:$(STAFI_PRPC)\"%" $HOME/.stafihubd/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.stafihub/config/config.toml
```
### Reset chain data
```Bash
stafihubd tendermint unsafe-reset-all --home ~/.stafihub
```
### Create service
```Bash
sudo tee /etc/systemd/system/stafihubd.service > /dev/null <<EOF
[Unit]
Description=StaFiHub Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which stafihubd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```
### Register and start service
```Bash
sudo systemctl restart systemd-journald && \
sudo systemctl daemon-reload && \
sudo systemctl enable stafihubd && \
sudo systemctl restart stafihubd && sudo journalctl -u stafihubd -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
stafihubd status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
stafihubd keys add <wallet_name> --keyring-backend file
```
To recover your wallet using seed phrase (optional)
```Bash
stafihubd keys add <wallet_name> --keyring-backend file --recover
```
### Check balance
```Bash
stafihubd query bank balances <wallet_address>
```
### Create validator
```Bash
stafihubd tx staking create-validator \
  --amount=1000000ufis \
  --from <wallet_name> \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey=$(stafihubd tendermint show-validator) \
  --moniker <moniker> \
  --chain-id=stafihub-public-testnet-2 \
  --gas-prices=0.025ufis \
  --keyring-backend file
```
### Show address validator
```Bash
stafihubd  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
stafihubd  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
stafihubd status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
stafihubd status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
stafihubd tendermint show-node-id
```
##### Get wallet balance
```Bash
stafihubd query bank balances <wallet_name>
```
##### Transfer funds
```Bash
stafihubd tx bank send <wallet_address> <to_wallet_address> 10000000uqck --keyring-backend file
```
##### Voting
```Bash
stafihubd tx gov vote 1 yes --from <wallet_name> --chain-id=stafihub-public-testnet-2 --keyring-backend file
```
##### Delegate stake
```Bash
stafihubd tx staking delegate <valoper_address> 10000000ufis --from=<wallet_name> --chain-id=stafihub-public-testnet-2 --gas=auto --keyring-backend file
```
##### Withdraw all rewards
```Bash
stafihubd tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=stafihub-public-testnet-2 --gas=auto --keyring-backend file
```
##### Withdraw rewards with commision
```Bash
stafihubd tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=stafihub-public-testnet-2 --gas=auto --keyring-backend file
```
##### Edit validator
```Bash
stafihubd tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=stafihub-public-testnet-2 \
--from=<wallet_name> \
--gas-prices=0.025ufis \
--keyring-backend file

```
##### Unjail validator
```Bash
stafihubd tx slashing unjail \
  --broadcast-mode=block \
  --from=<wallet_name> \
  --chain-id=stafihub-public-testnet-2 \
  --gas-prices=0.025ufis \
  --keyring-backend file

```
##### Delete node
```Bash
sudo systemctl stop stafihubd
sudo systemctl disable stafihubd
sudo rm /etc/systemd/system/stafihubd* -rf
sudo rm $(which stafihubd) -rf
sudo rm $HOME/.stafihubd* -rf
sudo rm $HOME/stafihub -rf
```
