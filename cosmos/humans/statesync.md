Statesync instructions
=
### Get state sync information
```bash
RPC=https://humans-rpc.creeptah.xyz:443 && \
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height) && \
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 1000)) && \
SYNC_BLOCK_HASH=$(curl -s "$RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
### Configure the state sync
```Bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.humansd/config/config.toml
```
### Set peers
```Bash
peers="78957f3ca436c1086d08b713b9eb81d55998b1f5@65.109.159.94:15656" && \
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.humansd/config/config.toml
```
### Stop the service and reset the data
```bash
sudo systemctl stop humansd
cp $HOME/.humansd/data/priv_validator_state.json $HOME/.humansd/priv_validator_state.json.backup
humansd tendermint unsafe-reset-all --keep-addr-book --home $HOME/.humansd
mv $HOME/.humansd/priv_validator_state.json.backup $HOME/.humansd/data/priv_validator_state.json
```
### Restart the service and check the log
```bash
sudo systemctl restart humansd && sudo journalctl -u humansd -f --no-hostname -o cat
```
