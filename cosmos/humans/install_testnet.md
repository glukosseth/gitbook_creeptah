Humans mainnet validator node
=
|Chain id|Latest app ver.|Cosmos SDK ver.|go ver.|Wasm|
|:------:|:-------------:|:-------------:|:-----:|:--:|
|humans_3000-31|v1.0.0|v0.46|v1.20.1+|off|

##### Official documentation:
> [Validator setup instructions](https://github.com/humansdotai/testnets/blob/master/Install.md)
##### Explorer:
> https://explorer.creeptah.xyz/testnet-humans

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl git jq lz4 build-essential libssl-dev -y
```
### Install variables
```Bash
MONIKER="your_moniker"
CHAIN_ID="humans_3000-31"
```
### Install go
```Bash
cd $HOME && \
wget "https://golang.org/dl/go1.20.1.linux-amd64.tar.gz" && \
sudo tar -C /usr/local -xzf "go1.20.1.linux-amd64.tar.gz" && \
rm "go1.20.1.linux-amd64.tar.gz" && \
echo "export GOROOT=/usr/local/go" >> ~/.bash_profile && \
echo "export GOPATH=$HOME/go" >> ~/.bash_profile && \
echo "export GO111MODULE=on" >> ~/.bash_profile && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```
### Download and build binaries
```Bash
cd $HOME && \
git clone https://github.com/humansdotai/humans.git && \
cd humans && \
git checkout v1.0.0 && \
make build
```
### Install Cosmovisor
```Bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### Setup Cosmovisor
```Bash
mkdir -p $HOME/.humansd/cosmovisor/genesis/bin && \
mv build/humansd $HOME/.humansd/cosmovisor/genesis/bin/ && \
sudo ln -s $HOME/.humansd/cosmovisor/genesis $HOME/.humansd/cosmovisor/current -f && \
sudo ln -s $HOME/.humansd/cosmovisor/current/bin/humansd /usr/local/bin/humansd -f
```
### Init app
```Bash
humansd init "$MONIKER" --chain-id $CHAIN_ID
```
### Download genesis
```Bash
wget -O $HOME/.humansd/config/genesis.json "https://raw.githubusercontent.com/humansdotai/testnets/master/friction/mission-3/genesis.json"
```
### Config app

Setup chain to config
```Bash
humansd config chain-id $CHAIN_ID
```
Config pruning
```Bash
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.humansd/config/app.toml
```
Set min gas price
```Bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1800000000aheart\"/" $HOME/.humansd/config/app.toml
```
Enable/Disable snapshot (optional)
```Bash
snapshot_interval="1000" && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.humansd/config/app.toml
```
Enable prometheus (optional)
```Bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.humansd/config/config.toml
```
### Create service
```Bash
sudo tee /etc/systemd/system/humansd.service > /dev/null << EOF
[Unit]
Description=humans node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --metrics --pruning=nothing --evm.tracer=json --minimum-gas-prices=1800000000aheart json-rpc.api eth,net,web3,miner --api.enable
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.humansd"
Environment="DAEMON_NAME=humansd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.humansd/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```Bash
sudo systemctl daemon-reload && \
sudo systemctl enable humansd && \
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```
### Check synchronization status ("catching_up": false is synced)
```Bash
humansd status 2>&1 | jq .SyncInfo
```
### Create wallet (!Safe your mnemonic)
```Bash
humansd keys add wallet
```
To recover your wallet using seed phrase (optional)
```Bash
humansd keys add wallet --recover
```
### Check balance
```Bash
humansd query bank balances <wallet_address>
```
### Create validator
```Bash
humansd tx staking create-validator \
--amount 1000000aheart \
--pubkey $(humansd tendermint show-validator) \
--moniker "$MONIKER" \
--chain-id $CHAIN_ID \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--identity "KEYBASE_ID" \
--details "DETAILS" \
--website "WEBSITE_URL" \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 1800000000aheart \
-y
```
### Optimisation config.toml
```Bash
sed -i \
  -e 's|^create_empty_blocks *=.*|create_empty_blocks = false|' \
  -e 's|^prometheus *=.*|prometheus = true|' \
  -e 's|^create_empty_blocks_interval *=.*|create_empty_blocks_interval = "30s"|' \
  -e 's|^timeout_propose *=.*|timeout_propose = "30s"|' \
  -e 's|^timeout_propose_delta *=.*|timeout_propose_delta = "5s"|' \
  -e 's|^timeout_prevote *=.*|timeout_prevote = "10s"|' \
  -e 's|^timeout_prevote_delta *=.*|timeout_prevote_delta = "5s"|' \
  -e 's|^cors_allowed_origins *=.*|cors_allowed_origins = ["*.humans.ai","*.humans.zone"]|' \
  -e 's|^timeout_prevote_delta *=.*|timeout_prevote_delta = "5s"|' \
  $HOME/.humansd/config/config.toml
```
### Optimisation app.toml
```Bash
sed -i \
  -e 's|^prometheus-retention-time *=.*|prometheus-retention-time = 1000000000000|' \
  -e 's|^enabled *=.*|enabled = true|' \
  -e '/^\[api\]$/,/^\[/ s/enable = false/enable = true/' \
  -e 's|^swagger *=.*|swagger = true|' \
  -e 's|^max-open-connections *=.*|max-open-connections = 100|' \
  -e 's|^rpc-read-timeout *=.*|rpc-read-timeout = 5|' \
  -e 's|^rpc-write-timeout *=.*|rpc-write-timeout = 3|' \
  -e 's|^rpc-max-body-bytes *=.*|rpc-max-body-bytes = 1000000|' \
  -e 's|^enabled-unsafe-cors *=.*|enabled-unsafe-cors = false|' \
  $HOME/.humansd/config/app.toml
  ```

Upgrade guide
=
