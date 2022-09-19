# List of usefull command

### Check number of peers
```Bash
curl -s http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr | split(":")[2])"' | wc -l
```
### Change laddr adress 127.0.0.1 to 0.0.0.0 for RPC
```Bash
NODE_PRPC=$(grep -A 3 "\[rpc\]" ~/.<*>d/config/config.toml | egrep -o ":[0-9]+")
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:$(NODE_PRPC)\"%laddr = \"tcp://0.0.0.0:$(NODE_PRPC)\"%" $HOME/.<*>d/config/config.toml
```
