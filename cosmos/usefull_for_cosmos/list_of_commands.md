# List of usefull commands

## General

#### Check consensus
```Bash
curl -s localhost:26657/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array'
```
#### Check unjail time
```Bash
<app>d q slashing signing-info $(<app>d tendermint show-validator)
```
#### Export private key
```Bash
<app>d keys export <name_wallet> --unarmored-hex --unsafe
```

## Peers

#### To know peer id and ip
```Bash
echo "$(<app>d tendermint show-node-id)@$(curl ifconfig.me):$(cat $HOME/.<app>dd/config/config.toml | grep laddr | grep -E '([[:digit:]]{4}6)' -o)"
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
NODE_PRPC=$(grep -A 3 "\[rpc\]" ~/.<app>d/config/config.toml | egrep -o ":[0-9]+")
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:$(NODE_PRPC)\"%laddr = \"tcp://0.0.0.0:$(NODE_PRPC)\"%" $HOME/.<app>d/config/config.toml
```




