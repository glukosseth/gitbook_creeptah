StarkNet node
=
## Install guide
### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install Pyton
Install `Pip` to manage software packages for Python.
```Bash
sudo apt install -y python3-pip
```
Install a few more packages and development tools
```Bash
sudo apt install -y curl git screen build-essential libssl-dev libffi-dev python3-dev
sudo apt-get install libgmp-dev
pip3 install fastecdsa
sudo apt-get install -y pkg-config
```
### Install Rust
```Bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
### Install Cargo
```Bash
sudo apt install cargo -y
source $HOME/.cargo/env
```
### Update Rust to the latest version
```Bash
rustup update stable
```
### Clone Pathfinder github repository
```Bash
ver=""
git clone --branch $ver https://github.com/eqlabs/pathfinder.git
```
### Create a virtual environment for a node
```Bash
sudo apt install python3.8-venv

# Open separate terminal to run node
screen -S stark

# Move to py folder
cd pathfinder/py

# Create the virtual environment called venv
python3 -m venv .venv

# and activate it
source .venv/bin/activate

# Install some more tools for node
PIP_REQUIRE_VIRTUALENV=true pip install --upgrade pip
PIP_REQUIRE_VIRTUALENV=true pip install -r requirements-dev.txt

# Test if previous steps were successful
pytest
```
### Assemble node
```Bash
cargo build --release --bin pathfinder
```
### Create Infura or Alchemy account

- Go to [Alchemy](https://www.alchemy.com/) (for ex.) and sing up.
- On dashboard click on Create App
- Give new app a name and choose either Mainnet or Goerli Network
- You will now be able to see the endpoint on your dashboard. Copy the https one

### Run node
```Bash
cargo run --release --bin pathfinder -- --ethereum.url https://mainnet.infura.io/v3/xxxxx

# or

cargo run --release --bin pathfinder -- --ethereum.url https://eth-goerli.alchemyapi.io/v2/xxxxx
```
To exit from `screen` tap `CTRL+a` and than `d`.

## Update node

Enter to screen
```Bash
screen -r stark
```
To stop node tap `CTRL+c`, than `CTRL+a` and `d`

Update bin
```Bash
cd pathfinder
git fetch
ver=""
git checkout $ver
git clone --branch $ver https://github.com/eqlabs/pathfinder.git
```
Enter to screen
```Bash
screen -r stark
```
Run node
```Bash
cd $HOME/pathfinder/py
python3 -m venv .venv
source .venv/bin/activate
PIP_REQUIRE_VIRTUALENV=true pip install --upgrade pip
PIP_REQUIRE_VIRTUALENV=true pip install -r requirements-dev.txt
pytest
cargo build --release --bin pathfinder

# run node
cargo run --release --bin pathfinder -- --ethereum.url https://mainnet.infura.io/v3/xxxxx

# or

cargo run --release --bin pathfinder -- --ethereum.url https://eth-goerli.alchemyapi.io/v2/xxxxx
```
To exit from `screen` tap `CTRL+a` and than `d`.
