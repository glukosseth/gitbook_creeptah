```Bash
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.0.0-rc3
make install

Default config file location: ~/.relayer/config/config.yaml

rly config init --memo "My custom memo"

rly chains add --url https://raw.githubusercontent.com/glukosseth/testnet_guide/main/cosmos/usefull_for_cosmos/relayer-2.0.0/gaia.json gaia
rly chains add --url https://raw.githubusercontent.com/glukosseth/testnet_guide/main/cosmos/usefull_for_cosmos/relayer-2.0.0/stride.json stride

rly keys restore gaia testkey "mnemonic words here"
rly keys restore stride testkey "mnemonic words here"

rly q balance gaia
rly q balance stride
```

```Bash
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
