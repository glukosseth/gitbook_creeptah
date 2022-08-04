```Bash
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.0.0-rc3
make install

Default config file location: ~/.relayer/config/config.yaml

rly config init --memo "My custom memo"

rly chains add --url https://raw.githubusercontent.com/glukosseth/testnet_guide/main/cosmos/usefull_for_cosmos/relayer-2.0.0/gaia.json
rly chains add --url https://raw.githubusercontent.com/glukosseth/testnet_guide/main/cosmos/usefull_for_cosmos/relayer-2.0.0/stride.json

rly keys restore gaia testkey "mnemonic words here"
rly keys restore stride testkey "mnemonic words here"

rly q balance gaia
rly q balance stride
```
