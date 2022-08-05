# Massa node

##### Official documentation:
> [Validator setup instructions](https://docs.massa.net/en/latest/testnet/install.html)
##### Explorer:
> https://massa.net/testnet/

Install guide
=
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install pkg-config curl git build-essential libssl-dev libclang-dev screen -y
```
### Install rustup
```Bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
### Configure path
```Bash
source $HOME/.cargo/env
```
### Check rust version
```Bash
rustc --version
```
### Install nigthly and set it as default
```Bash
rustup toolchain install nightly
rustup default nightly
rustc --version
```
### Download and run node
```Bash
screen -S massa
git clone --branch testnet https://github.com/massalabs/massa.git
cd massa/massa-node/
RUST_BACKTRACE=full cargo run --release -- -p "wallet_password" |& tee logs.txt
```
### Run client
```Bash
# tap Ctrl+A and D
cd massa/massa-client/
cargo run --release -- -p "wallet_password"
```
