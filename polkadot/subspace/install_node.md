Subspace node
=
##### Official documentation:
> [Setup instructions](https://github.com/subspace/subspace/blob/main/docs/farming.md)

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
### Install Docker
```Bash
sudo -i
cd $HOME
apt purge docker docker-engine docker.io containerd docker-compose -y
rm /usr/bin/docker-compose /usr/local/bin/docker-compose
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
systemctl restart docker
curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
### Set variable parameters
```Bash
# set parameters
SUBSPACE_WALLET=<your_wallet>
SUBSPACE_NODE="<node_name>"
SUBSPACE_PLOT=<plot_size>

# export to .bash_profile
echo 'export SUBSPACE_WALLET='$SUBSPACE_WALLET >> $HOME/.bash_profile
echo 'export SUBSPACE_NODE="'${SUBSPACE_NODE}'"' >> $HOME/.bash_profile
echo 'export SUBSPACE_PLOT='$SUBSPACE_PLOT >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Create home dir
```Bash
mkdir subspace
```
### Create docker-compose.yml
```
cd subspace
sudo tee $HOME/subspace/docker-compose.yml > /dev/null <<EOF
version: "3.7"
services:
  node:
    # Replace `snapshot-DATE` with latest release (like `snapshot-2022-apr-29`)
    # For running on Aarch64 add `-aarch64` after `DATE`
    image: ghcr.io/subspace/node:gemini-2a-2022-sep-10
    volumes:
# Instead of specifying volume (which will store data in `/var/lib/docker`), you can
# alternatively specify path to the directory where files will be stored, just make
# sure everyone is allowed to write there
      - node-data:/var/subspace:rw
#      - /path/to/subspace-node:/var/subspace:rw
    ports:
# If port 30333 is already occupied by another Substrate-based node, replace all
# occurrences of `30333` in this file with another value
      - "0.0.0.0:30333:30333"
    restart: unless-stopped
    command: [
      "--chain", "gemini-2a",
      "--base-path", "/var/subspace",
      "--execution", "wasm",
      "--state-pruning", "archive",
      "--port", "30333",
      "--rpc-cors", "all",
      "--rpc-methods", "safe",
      "--unsafe-ws-external",
      "--validator",
# Replace `INSERT_YOUR_ID` with your node ID (will be shown in telemetry)
      "--name", "$SUBSPACE_NODE"
    ]
    healthcheck:
      timeout: 5s
# If node setup takes longer then expected, you want to increase `interval` and `retries` number.
      interval: 30s
      retries: 5

  farmer:
    depends_on:
      node:
        condition: service_healthy
    # Replace `snapshot-DATE` with latest release (like `snapshot-2022-apr-29`)
    # For running on Aarch64 add `-aarch64` after `DATE`
    image: ghcr.io/subspace/farmer:gemini-2a-2022-sep-10
    volumes:
# Instead of specifying volume (which will store data in `/var/lib/docker`), you can
# alternatively specify path to the directory where files will be stored, just make
# sure everyone is allowed to write there
      - farmer-data:/var/subspace:rw
#      - /path/to/subspace-farmer:/var/subspace:rw
    ports:
# Un-comment following line to unlock farmer's RPC
#      - "127.0.0.1:9955:9955"
# If port 40333 is already occupied by something else, replace all
# occurrences of `40333` in this file with another value
      - "0.0.0.0:40333:40333"
    restart: unless-stopped
    command: [
      "--base-path", "/var/subspace",
      "farm",
      "--node-rpc-url", "ws://node:9944",
      "--ws-server-listen-addr", "0.0.0.0:9955",
      "--listen-on", "/ip4/0.0.0.0/tcp/40333",
# Replace `WALLET_ADDRESS` with your Polkadot.js wallet address
      "--reward-address", "$SUBSPACE_WALLET",
# Replace `PLOT_SIZE` with plot size in gigabytes or terabytes, for instance 100G or 2T (but leave at least 60G of disk space for node and some for OS)
      "--plot-size", "$SUBSPACE_PLOT"
    ]
volumes:
  node-data:
  farmer-data:
EOF
```
### Run node
```Bash
docker-compose up -d && docker-compose logs --tail=1000 -f
```
### Delete node
```Bash
cd $HOME/subspace
docker-compose down -v
cd $HOME && rm -rf $HOME/subspace/
```
