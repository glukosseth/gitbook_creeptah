haqq validator node
=
##### Official documentation:
> [Validator setup instructions](https://github.com/haqq-network/validators-contest)
##### Explorer:
> https://haqq.explorers.guru/

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
cd $HOME && git clone https://github.com/haqq-network/haqq && \
cd haqq && \
git checkout v1.0.3 && \
make install && \
haqqd version --long | head
```
### Init app
```Bash
haqqd init <moniker> --chain-id haqq_54211-2
```
### Download genesis
```Bash
cd $HOME/.haqqd/config/ && wget https://raw.githubusercontent.com/haqq-network/validators-contest/master/genesis.json
```
### Config app
Setup chain to config
```Bash
haqqd config chain-id haqq_54211-2
```
Setup seeds and peers
```Bash
seeds="62bf004201a90ce00df6f69390378c3d90f6dd7e@seed2.testedge2.haqq.network:26656,23a1176c9911eac442d6d1bf15f92eeabb3981d5@seed1.testedge2.haqq.network:26656"
peers="b3ce1618585a9012c42e9a78bf4a5c1b4bad1123@65.21.170.3:33656,952b9d918037bc8f6d52756c111d0a30a456b3fe@213.239.217.52:29656,85301989752fe0ca934854aecc6379c1ccddf937@65.109.49.111:26556,d648d598c34e0e58ec759aa399fe4534021e8401@109.205.180.81:29956,f2c77f2169b753f93078de2b6b86bfa1ec4a6282@141.95.124.150:20116,eaa6d38517bbc32bdc487e894b6be9477fb9298f@78.107.234.44:45656,37513faac5f48bd043a1be122096c1ea1c973854@65.108.52.192:36656,d2764c55607aa9e8d4cee6e763d3d14e73b83168@66.94.119.47:26656,fc4311f0109d5aed5fcb8656fb6eab29c15d1cf6@65.109.53.53:26656,297bf784ea674e05d36af48e3a951de966f9aa40@65.109.34.133:36656,bc8c24e9d231faf55d4c6c8992a8b187cdd5c214@65.109.17.86:32656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.haqqd/config/config.toml
```
Config pruning
```Bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.haqqd/config/app.toml
```
Set min gas price
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0aISLM\"/" $HOME/.haqqd/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.haqqd/config/app.toml
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
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.haqqd/config/config.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.haqqd/config/config.toml
```
### Reset chain data
```Bash
haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
```
### Create service
```Bash
sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqq node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which haqqd) start --home $HOME/.haqqd
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
sudo systemctl enable haqqd && \
sudo systemctl restart haqqd && sudo journalctl -u haqqd -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
haqqd status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
haqqd keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
haqqd keys add <wallet_name> --recover
```
### Check balance
```Bash
haqqd query bank balances <wallet_address>
```
### Create validator
```Bash
haqqd tx staking create-validator \
--moniker="<moniker>" \
--amount=1000000000000000000aISLM \
--pubkey=$(haqqd tendermint show-validator) \
--chain-id=haqq_54211-2\
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.10 \
--min-self-delegation=1000000 \
--from=<wallet_name>
```
### Show address validator
```Bash
haqqd  keys show <wallet_name> --bech val -a
```
### Check validator status
```Bash
haqqd  query staking validator <valoper_address>
```

Usefull command
=
##### Synchronization info
```Bash
haqqd status 2>&1 | jq .SyncInfo
```
##### Node info
```Bash
haqqd status 2>&1 | jq .NodeInfo
```
##### Show node id
```Bash
haqqd tendermint show-node-id
```
##### Get wallet balance
```Bash
haqqd query bank balances <wallet_name>
```
##### Transfer funds
```Bash
haqqd tx bank send <wallet_address> <to_wallet_address> 10000000aISLM
```
##### Voting
```Bash
haqqd tx gov vote 1 yes --from <wallet_name> --chain-id=haqq_54211-2
```
##### Delegate stake
```Bash
haqqd tx staking delegate <valoper_address> 10000000aISLM --from=<wallet_name> --chain-id=haqq_54211-2
```
##### Withdraw all rewards
```Bash
haqqd tx distribution withdraw-all-rewards --from=<wallet_name> --chain-id=haqq_54211-2
```
##### Withdraw rewards with commision
```Bash
haqqd tx distribution withdraw-rewards <valoper_address> --from=<wallet_name> --commission --chain-id=haqq_54211-2
```
##### Edit validator
```Bash
haqqd tx staking edit-validator \
--moniker=<moniker> \
--identity=<identity> \
--website=<web> \
--details=<any details> \
--chain-id=haqq_54211-2\
--from=<wallet_name>
```
##### Unjail validator
```Bash
haqqd tx slashing unjail \
  --from=<wallet_name> \
  --chain-id=haqq_54211-2
```
##### Delete node
```Bash
sudo systemctl stop haqqd
sudo systemctl disable haqqd
sudo rm /etc/systemd/system/haqqd* -rf
sudo rm $(which haqqd) -rf
sudo rm $HOME/.haqqd* -rf
sudo rm $HOME/haqq -rf
```
