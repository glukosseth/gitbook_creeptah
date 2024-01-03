# List of usefull commands

## General

#### Check consensus
```Bash
curl -s localhost:26657/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array'
```
#### Check unjail time
```Bash
<app> q slashing signing-info $(<app> tendermint show-validator)
```
#### Export private key
```Bash
<app> keys export <name_wallet> --unarmored-hex --unsafe
```
#### Get EVM (EIP-55), Hex or Bech32 Valoper address from your wallet address
```Bash
<app> debug addr $(<app> keys show <key_name> -a)
```
#### List active validatirs
```Bash
<app> q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' | sort -gr | nl
```
#### List not active validatirs
```Bash
<app> q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED" or .status=="BOND_STATUS_UNBONDING")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```
#### Check that your validator is active
```Bash
<app> query tendermint-validator-set | grep "$(<app> tendermint show-address)"
```
#### Check rewards from your validator
```Bash
<app> query distribution rewards $(<app> keys show <key_name> -a) $(<app> keys show <key_name> --bech val -a)
```

## Peers

#### To know peer id and ip
```Bash
echo "$(<app> tendermint show-node-id)@$(curl ifconfig.me):$(cat $HOME/.<app>/config/config.toml | grep laddr | grep -E '([[:digit:]]{4}6)' -o)"
```
#### Check number of peers
```Bash
curl -s http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr | split(":")[2])"' | wc -l
```
#### Check connected peers
```Bash
curl -s http://localhost:26657/net_info | jq '.result.peers[].node_info.moniker'
```

## Change config

#### Change laddr adress 127.0.0.1 to 0.0.0.0 for RPC
```Bash
NODE_PRPC=$(grep -A 3 "\[rpc\]" ~/.<app>/config/config.toml | egrep -o ":[0-9]+")
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:$(NODE_PRPC)\"%laddr = \"tcp://0.0.0.0:$(NODE_PRPC)\"%" $HOME/.<app>/config/config.toml
```




