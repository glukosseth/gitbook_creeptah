# Relayer v.2.0.0

In IBC, blockchains do not directly pass messages to each other over the network. This is where relayer comes in. A relayer process monitors for updates on opens paths between sets of IBC enabled chains. The relayer submits these updates in the form of specific message types to the counterparty chain. Clients are then used to track and verify the consensus state.

In addition to relaying packets, this relayer can open paths across chains, thus creating clients, connections and channels.

# Install

### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```

### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils ncdu git jq liblz4-tool -y
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
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.0.0-rc3
make install
```
Default config file location: ~/.relayer/config/config.yaml

### Init app
```Bash
rly config init --memo "My custom memo"
```

### Config app
Set chains
```Bash
rly chains add --url https://raw.githubusercontent.com/glukosseth/testnet_guide/main/cosmos/usefull_for_cosmos/relayer-2.0.0/gaia.json gaia
rly chains add --url https://raw.githubusercontent.com/glukosseth/testnet_guide/main/cosmos/usefull_for_cosmos/relayer-2.0.0/stride.json stride
```
Restore key
```Bash
rly keys restore gaia testkey "mnemonic words here"
rly keys restore stride testkey "mnemonic words here"
```
Check balance
```Bash
rly q balance gaia
rly q balance stride
```
Change `paths`
```Bash
# open config.yaml
nano ~/.relayer/config/config.yaml

# change paths in config
paths:
    stride-gaia:
        src:
            chain-id: STRIDE-TESTNET-2
            client-id: 07-tendermint-0
            connection-id: connection-0
        dst:
            chain-id: GAIA
            client-id: 07-tendermint-0
            connection-id: connection-0
        src-channel-filter:
            rule: allowlist
            channel-list:
                - channel-0
                - channel-1
                - channel-2
                - channel-3
                - channel-4
```
Verify valid chain, client, and connection
```Bash
rly paths list
```
Verify that you have a healthy RPC address. \
If
```Bash
-> chns(✔) clnts(✔) conn(✔)
```

### Create service
```Bash
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=relayer_go
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start stride-gaia --log-format logfmt --processor events
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```

### Register and start service
```Bash
sudo systemctl daemon-reload
sudo systemctl enable rlyd
sudo systemctl restart rlyd && sudo journalctl -u rlyd -f
```
