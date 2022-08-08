# Relay interchain-queries using the Relayer IBC GO v.2.0.0
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
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```
### Create config.yaml
```Bash
cd $HOME

# create home dir
mkdir $HOME/.icq

# create config.yaml
tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride
chains:
  gaia:
    key: default
    chain-id: GAIA
    rpc-addr: http://78.107.234.44:36657
    grpc-addr: http://78.107.234.44:9797
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: $HOME/.icq/keys
    debug: true
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride:
    key: default
    chain-id: STRIDE-TESTNET-2
    rpc-addr: http://127.0.0.1:26677
    grpc-addr: http://127.0.0.1:9094
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: $HOME/.icq/keys
    debug: true
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```
### Restore key
```Bash
icq keys restore --chain stride default
icq keys restore --chain gaia default
```
### Create service
```Bash
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=icq
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```Bash
sudo systemctl daemon-reload && \
sudo systemctl enable icqd && \
sudo systemctl restart icqd && sudo journalctl -u icqd -f -o cat
```
