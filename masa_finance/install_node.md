Masa Finance node
=
##### Official documentation:
> [Setup instructions](https://github.com/masa-finance/masa-node-v1.0)

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git ncdu git jq ncdu htop liblz4-tool -y
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
rm -rf masa-node-v1.0
git clone https://github.com/masa-finance/masa-node-v1.0
cd $HOME/masa-node-v1.0/src
git checkout v1.03
make all
```
### Copy binaries
```Bash
cd $HOME/masa-node-v1.0/src/build/bin
sudo cp * /usr/local/bin
```
### Init app
```Bash
cd $HOME/masa-node-v1.0
geth --datadir data init ./network/testnet/genesis.jso
```
### Set node name
```Bash
MASA_NODENAME="NODE_NAME"
```
### Create service
```Bash
sudo tee /etc/systemd/system/masad.service > /dev/null <<EOF
[Unit]
Description=MASA103
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which geth) \
  --identity ${MASA_NODENAME} \
  --datadir $HOME/masa-node-v1.0/data \
  --port 30300 \
  --syncmode full \
  --verbosity 5 \
  --emitcheckpoints \
  --istanbul.blockperiod 10 \
  --mine \
  --miner.threads 1 \
  --networkid 190260 \
  --http --http.corsdomain "*" --http.vhosts "*" --http.addr 127.0.0.1 --http.port 8545 \
  --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul \
  --maxpeers 50 \
  --bootnodes enode://91a3c3d5e76b0acf05d9abddee959f1bcbc7c91537d2629288a9edd7a3df90acaa46ffba0e0e5d49a20598e0960ac458d76eb8fa92a1d64938c0a3a3d60f8be4@54.158.188.182:21000
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
Environment="PRIVATE_CONFIG=ignore"
[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```Bash
sudo systemctl daemon-reload
sudo systemctl enable masad
sudo systemctl start masad && sudo journalctl -u masad -f -o cat
```
Usefull command
=
### Status ETH node
```Bash
geth attach ipc:$HOME/masa-node-v1.0/data/geth.ipc
```
##### Console commands
```Bash
# Node info
admin.nodeInfo

# Synchronization info (false = synced)
eth.syncing

# Number of peers
net.peerCount

# Connecting info (true = connected)
net.listening

# Node catalog
admin.datadir

# List connection (short list)

admin.peers.forEach(function(value){console.log(value.network.remoteAddress+"\t"+value.name)})

# List connection (long list)
admin.peers
```
### Backup nodekey
```Bash
cp data/geth/nodekey $HOME
```
### Restore nodekey
```Bash
cp $HOME/nodekey data/geth/
```
### Clean database
```Bash
cd $HOME/masa-node-v1.0 && geth removedb --datadir data
```
