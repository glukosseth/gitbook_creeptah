# Charon Distributed Validator Node

##### Official documentation:
> [Validator setup instructions](https://github.com/ObolNetwork/charon-distributed-validator-node#step-2-leader-creates-the-dkg-configuration-file-and-distributes-it-to-everyone-else)

## Creating and backing up a private key for charon

### Update packages
```Bash
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```Bash
sudo apt install curl build-essential git wget jq tmux chrony -y
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
### Creating an enr private key
```Bash
# clone repo
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git

# change directory
cd charon-distributed-validator-node

# create charon ENR private key, this will create a charon-enr-private-key file in the .charon directory
docker run --rm -v "$(pwd):/opt/charon" ghcr.io/obolnetwork/charon:v0.8.1 create enr
```
You should expect to see a console output like
```Bash
Created ENR private key: .charon/charon-enr-private-key
enr:-JG4QGQpV4qYe32QFUAbY1UyGNtNcrVMip83cvJRhw1brMslPeyELIz3q6dsZ7GblVaCjL_8FKQhF6Syg-O_kIWztimGAYHY5EvPgmlk
gnY0gmlwhH8AAAGJc2VjcDI1NmsxoQKzMe_GFPpSqtnYl-mJr8uZAUtmkqccsAx7ojGmFy-FY4N0Y3CCDhqDdWRwgg4u
```
!! Backup this private key

### Fill form

Submit the created ENR public address (the console output starting with enr:-... not the contents of the private key file) to the appropriate [typeform](https://obol.typeform.com/AthenaTestnet).
