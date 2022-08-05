# Faucet

A faucet that uses [cosmos-sdk](https://github.com/cosmos/cosmos-sdk) executable binaries only.

The main purpose of this faucet is to avoid using RPC or API endpoints, and use the CLI binary instead, more specifically, the commands:

```Bash
{app}d tx bank send
```
and:
```Bash
{app}d query txs
```
Since the faucet only uses the CLI binary, it is compatible with practically any blockchain built with cosmos-sdk even if different types of keys are used.

## Instalation

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
git clone https://github.com/glukosseth/faucet.git
cd faucet
make install
```
## Usage
### Configuration
You can configure the faucet either using command line flags or environment variables. Use `faucet --help` for more commands

```Bash

faucet --cli-name seid --denoms usei  --keyring-backend test --keyring-password <wallet_password> --mnemonic "<mnemonic>" --credit-amount 100000 --max-credit 200000 --node tcp://localhost:26657
```
### Request tokens
You can request tokens by sending a `POST` request to the faucet, with a key address in a `JSON`:
```Bash
curl -X POST -d '{"address": "<your_sei_address>"}' http://<faucet_ip>:8000
```
