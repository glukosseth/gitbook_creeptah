# Massa node

##### Official documentation:
> [Validator setup instructions](https://docs.massa.net/docs/node/install)
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

# tap Ctrl+A and D
```
### Enable routing
```Bash
# get config ip
ifconfig

# create config file
sudo tee <<EOF >/dev/null $HOME/massa/massa-node/config/config.toml
[network]
routable_ip = "external_ip_address"
EOF

# restart node
```
### Run client
```Bash
# tap Ctrl+A and D
cd massa/massa-client/
cargo run --release -- -p "wallet_password"
```
### Generate wallet
```Bash
wallet_generate_secret_key
```
### Add staking secret keys
```Bash
node_add_staking_secret_keys <secret_key>
```
### Get test tokens
Go to Discord channel `#testnet-faucet` and ask tokens

![image](https://user-images.githubusercontent.com/108256873/183116732-45119e15-f938-4186-a3bc-353263a03a10.png)

### Check balance and by rolls
```Bash
wallet_info
```
![image](https://user-images.githubusercontent.com/108256873/183117052-d36bf679-cb60-47cd-bad7-1c2977d3d343.png)

```Bash
buy_rolls <wallet_address> 1 0
```
![image](https://user-images.githubusercontent.com/108256873/183117161-4d7c2855-bbee-4ebb-8762-e7c409fcbd7e.png)

### Incentivise registration
Tag bot any emoji in Discord channel `#testnet-rewards-registration`. In the DM send to bot ip address and hash from command:
```Bash
node_testnet_rewards_program_ownership_proof <staking_address> <discord_id>
```
