## Key management
#### Add new key
```bash
humansd keys add wallet
```
#### Recover existing key
```bash
humansd keys add wallet --recover
```
#### List all keys
```bash
humansd keys list
```
#### Delete key
```bash
humansd keys delete wallet
```
#### Export key to the file
```bash
humansd keys export wallet
```
#### Import key from the file
```bash
humansd keys import wallet wallet.backup
```
#### Query wallet balance
```bash
humansd q bank balances $(humansd keys show wallet -a)
```

## Validator management
#### Create new validator
```bash
humansd tx staking create-validator \
--amount 1000000aheart \
--pubkey $(humansd tendermint show-validator) \
--moniker "MONIKER_NAME" \
--identity "KEYBASE_ID" \
--details "DETAILS" \
--website "WEBSITE_URL" \
--chain-id humans_1089-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 1800000000aheart \
-y
```
#### Edit existing validator
```bash
humansd tx staking edit-validator \
--new-moniker "MONIKER_NAME" \
--identity "KEYBASE_ID" \
--details "DETAILS" \
--website "WEBSITE_URL" \
--chain-id humans_1089-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 1800000000aheart \
-y
```
#### Remove node
```bash
cd $HOME && \
sudo systemctl stop humansd && \
sudo systemctl disable humansd && \
sudo rm /etc/systemd/system/humansd.service && \
sudo systemctl daemon-reload && \
rm -f $(which humansd) && \
rm -rf $HOME/.humansd && \
rm -rf $HOME/humans
```
#### Unjail validator
```bash
humansd tx slashing unjail --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Jail reason
```bash
humansd query slashing signing-info $(humansd tendermint show-validator)
```
#### List all active validators
```bash
humansd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### List all inactive validators
```bash
humansd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### View validator details
```bash
humansd q staking validator $(humansd keys show wallet --bech val -a)
```

## Token management
#### Withdraw rewards from all validators
```bash
humansd tx distribution withdraw-all-rewards --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Withdraw commission and rewards from your validator
```bash
humansd tx distribution withdraw-rewards $(humansd keys show wallet --bech val -a) --commission --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Delegate tokens to yourself
```bash
humansd tx staking delegate $(humansd keys show wallet --bech val -a) 1000000aheart --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Delegate tokens to validator
```bash
humansd tx staking delegate <TO_VALOPER_ADDRESS> 1000000aheart --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Redelegate tokens to another validator
```bash
humansd tx staking redelegate $(humansd keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000aheart --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Unbond tokens from your validator
```bash
humansd tx staking unbond $(humansd keys show wallet --bech val -a) 1000000aheart --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Send tokens to the wallet
```bash
humansd tx bank send wallet <TO_WALLET_ADDRESS> 1000000aheart --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```

## Governance
#### List all proposals
```bash
humansd query gov proposals
```
#### View proposal by id
```bash
humansd query gov proposal 1
```
#### Vote 'Yes'
```bash
humansd tx gov vote 1 yes --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Vote 'No'
```bash
humansd tx gov vote 1 no --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Vote 'Abstain'
```bash
humansd tx gov vote 1 abstain --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```
#### Vote 'NoWithVeto'
```bash
humansd tx gov vote 1 NoWithVeto --from wallet --chain-id humans_1089-1 --gas-adjustment 1.4 --gas auto --gas-prices 1800000000aheart -y
```

## Maintenance
#### Get validator info
```bash
humansd status 2>&1 | jq .ValidatorInfo
```
#### Get sync info
```bash
humansd status 2>&1 | jq .SyncInfo
```
#### Get node peer
```bash
echo $(humansd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.humansd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```
#### Check if validator key is correct
```bash
[[ $(humansd q staking validator $(humansd keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(humansd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```
#### Get live peers
```bash
curl -sS http://localhost:12257/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```
#### Reset chain data
```bash
humansd tendermint unsafe-reset-all --keep-addr-book --home $HOME/.humansd --keep-addr-book
```
