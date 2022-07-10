# Gentx for Sei Incentivized testnet (chain-id atlantic-1)

## Method 1. If you install new node

### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils ncdu gcc git jq liblz4-tool -y
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
git clone https://github.com/sei-protocol/sei-chain.git && cd sei-chain
git checkout 1.0.6beta
make install
```
### Init app
```Bash
seid init <moniker> --chain-id atlantic-1
```
### Create wallet (!Safe your mnemonic)
```Bash
seid keys add <wallet_name>
```
To recover your wallet using seed phrase (optional)
```Bash
seid keys add <wallet_name> --recover
```
### Add genesis account
```Bash
seid add-genesis-account <wallet_name> 100000000usei
```
### Generate gentx
```Bash
seid gentx <wallet_name> 100000000usei \
--moniker="<moniker>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.05 \
--chain-id atlantic-1 \
```
### Submit PR with Gentx
```Bash
cat ~/.sei/config/gentx/gentx-*
```
Copy the content and create a PR

## Method 2. If you have installed node

```Bash
cd $HOME

# create new dirrecrtory
mkdir atlantic-1

# init node
seid init <moniker> --chain-id atlantic-1 --home $HOME/atlantic-1

# create wallet from mnemonic
seid keys add <wallet_name> --recover --home $HOME/atlantic-1 

# add genesis account
seid add-genesis-account <wallet_name> 100000000usei --home $HOME/atlantic-1

# generate gentx
seid gentx <wallet_name> 100000000usei \
--moniker="<moniker>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.05 \
--chain-id atlantic-1 \

# copy the content
cat ~/atlantic-1/config/gentx/gentx-*
```
