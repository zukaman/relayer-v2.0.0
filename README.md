# Set up a hermes relayer
In current example we will learn how to set up IBC relayer between two cosmos chains

## Update system
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install unzip -y
```

## Make hermes home dir
```
cd $HOME
mkdir -p $HOME/.hermes/bin
```

## Download Hermes
```
verison="v0.15.0"
wget "https://github.com/informalsystems/ibc-rs/releases/download/$verison/hermes-$verison-x86_64-unknown-linux-gnu.tar.gz"
tar -C $HOME/.hermes/bin/ -vxzf hermes-$verison-x86_64-unknown-linux-gnu.tar.gz
rm hermes-$verison-x86_64-unknown-linux-gnu.tar.gz
echo "export PATH=$PATH:$HOME/.hermes/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
 
```
# Setup hermes config

## Set up variables
All settings below are just example for IBC Relayer between stride `STRIDE-TESTNET-2` and juno `GAIA` testnets. Please fill with your own values.

### Chain-1
```
MEMO='zuka#5870' #add your discord username here
CH1_RPC='127.0.0.1' #add the IP of your first node
CH1_RPC_PORT='26657' #add RPC port of your first node
CH1_GRPC_PORT='9090' #add gRPC port of your first node
CH1_CHAIN_ID='STRIDE-TESTNET-2' #add chainID of youe first node
CH1_ACC_PREFIX='stride' #add account prefix
CH1_DENOM='ustrd' #add denom
CH1_TRUST='10h 29m' #add trusting period. Must be less than unbond time
CH1_REL_WALLET='CH1_REL_WALLET'
```

### Chain-2
You don't need to change anything if you don't have a gaia installed (my node is used here)
```
CH2_RPC='78.107.234.44' #add the IP of your second node
CH2_RPC_PORT='36657' #add RPC port of your second node
CH2_GRPC_PORT='9797' #add gRPC port of your second node
CH2_CHAIN_ID='GAIA' #add ChainID of your second node
CH2_ACC_PREFIX='cosmos' #add account prefix
CH2_DENOM='uatom' #add denom
CH2_TRUST='10h 29m' #add trusting period. Must be less than unbond time
CH2_REL_WALLET='CH2_REL_WALLET'
```
### Save vars (copy paste ALL LINES at once) 
```
echo "
export CH1_CHAIN_ID=${CH1_CHAIN_ID}
export CH1_DENOM=${CH1_DENOM}
export CH1_REL_WALLET=${CH1_REL_WALLET}
export CH2_CHAIN_ID=${CH2_CHAIN_ID}
export CH2_DENOM=${CH2_DENOM}
export CH2_REL_WALLET=${CH2_REL_WALLET}
export MEMO=${MEMO}
" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Create hermes config
Generate hermes config file using variables we have defined above
(copy and paste into the terminal with one command)
```
echo "[global]
log_level = 'info'
[mode]
[mode.clients]
enabled = true
refresh = true
misbehaviour = false
[mode.connections]
enabled = false
[mode.channels]
enabled = false
[mode.packets]
enabled = true
clear_interval = 100
clear_on_start = true
tx_confirmation = true
[rest]
enabled = true
host = '0.0.0.0'
port = 3000
[telemetry]
enabled = true
host = '0.0.0.0'
port = 3001
[[chains]]
id = '$CH1_CHAIN_ID'
rpc_addr = 'http://$CH1_RPC:$CH1_RPC_PORT'
grpc_addr = 'http://$CH1_RPC:$CH1_GRPC_PORT'
websocket_addr = 'ws://$CH1_RPC:$CH1_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$CH1_ACC_PREFIX'
key_name = '$CH1_REL_WALLET'
store_prefix = 'ibc'
max_tx_size = 100000
max_gas = 20000000
gas_price = { price = 0.001, denom = '$CH1_DENOM' }
gas_adjustment = 0.1
max_msg_num = 30
clock_drift = '5s'
trusting_period = '$CH1_TRUST'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = '$MEMO'
[chains.packet_filter]
policy = 'allow'
list = [
 ['transfer', 'channel-0'],# gaia
 ['icacontroller-GAIA.DELEGATION', 'channel-1'],
 ['icacontroller-GAIA.FEE', 'channel-2'],
 ['icacontroller-GAIA.WITHDRAWAL', 'channel-3'],
 ['icacontroller-GAIA.REDEMPTION', 'channel-4'],
]
[[chains]]
id = '$CH2_CHAIN_ID'
rpc_addr = 'http://$CH2_RPC:$CH2_RPC_PORT'
grpc_addr = 'http://$CH2_RPC:$CH2_GRPC_PORT'
websocket_addr = 'ws://$CH2_RPC:$CH2_RPC_PORT/websocket'
rpc_timeout = '10s'
account_prefix = '$CH2_ACC_PREFIX'
key_name = '$CH2_REL_WALLET'
store_prefix = 'ibc'
default_gas = 100000
max_gas = 2000000
gas_price = { price = 0.01, denom = '$CH2_DENOM' }
gas_adjustment = 0.1
max_msg_num = 30
max_tx_size = 2097152
clock_drift = '5s'
max_block_time = '30s'
trusting_period = '$CH2_TRUST'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = '$MEMO'
address_type = { derivation = 'cosmos' }
[chains.packet_filter]
policy = 'allow'
list = [
 ['transfer', 'channel-0'],# gaia
 ['icahost', 'channel-4'],
 ['icahost', 'channel-1'],
 ['icahost', 'channel-2'],
 ['icahost', 'channel-3'],
]" > $HOME/.hermes/config.toml
```

## Verify hermes configuration is correct
Before proceeding further please check if your configuration is correct
```
hermes health-check
```

Healthy output should look like:
```
2022-07-21T19:38:15.571398Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
2022-07-21T19:38:15.573884Z  INFO ThreadId(01) [STRIDE-TESTNET-2] performing health check...
2022-07-21T19:38:15.614273Z  INFO ThreadId(01) chain is healthy chain=STRIDE-TESTNET-2
2022-07-21T19:38:15.614313Z  INFO ThreadId(01) [GAIA] performing health check...
2022-07-21T19:38:15.627747Z  INFO ThreadId(01) chain is healthy chain=GAIA
Success: performed health check for all chains in the config
```

## Recover wallets using mnemonic phrase
Before you proceed with this step, please make sure you have created and funded with tokens seperate wallets on each chain!
Replace `<insert mnemonic here>` with the mnemonic from your wallet
```
hermes keys restore $CH1_CHAIN_ID -n $CH1_REL_WALLET -m "<insert mnemonics here>"
hermes keys restore $CH2_CHAIN_ID -n $CH2_REL_WALLET -m  "<insert mnemonics here>"
```
Successful output should look like:
```
2022-07-21T19:54:13.778550Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
Success: Restored key 'wallet' (stride19u539q85fun7nax92w853gcqtl74kr0evd65cl) on chain STRIDE-TESTNET-2
2022-07-21T19:54:14.956171Z  INFO ThreadId(01) using default configuration from '/root/.hermes/config.toml'
Success: Restored key 'wallet' (juno19u539q85fun7nax92w853gcqtl74kr0ee5ent0) on chain GAIA
```

## Create hermes service file
 (copy and paste into the terminal with one command)
```
sudo tee /etc/systemd/system/hermesd.service > /dev/null <<EOF
[Unit]
Description=HERMES
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which hermes) start
Restart=on-failure
RestartSec=10
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
EOF
```

## Start service
```
sudo systemctl daemon-reload
sudo systemctl enable hermesd
sudo systemctl restart hermesd && journalctl -u hermesd -f
```
## Successfull logs should look like this:
```
 2022-07-30T07:05:15.322025Z  INFO ThreadId(01) Scanned chains:
 2022-07-30T07:05:15.322031Z  INFO ThreadId(01) # Chain: STRIDE-TESTNET-2
   - Client: 07-tendermint-0
     * Connection: connection-0
       | State: OPEN
       | Counterparty state: OPEN
       + Channel: channel-0
         | Port: transfer
         | State: OPEN
         | Counterparty: channel-0
       + Channel: channel-1
         | Port: icacontroller-GAIA.DELEGATION
         | State: CLOSED
         | Counterparty: channel-4
       + Channel: channel-2
         | Port: icacontroller-GAIA.FEE
         | State: OPEN
         | Counterparty: channel-1
       + Channel: channel-3
         | Port: icacontroller-GAIA.WITHDRAWAL
         | State: OPEN
         | Counterparty: channel-2
       + Channel: channel-4
         | Port: icacontroller-GAIA.REDEMPTION
         | State: OPEN
         | Counterparty: channel-3
 # Chain: GAIA
   - Client: 07-tendermint-0
     * Connection: connection-0
       | State: OPEN
       | Counterparty state: OPEN
       + Channel: channel-0
         | Port: transfer
         | State: OPEN
         | Counterparty: channel-0
       + Channel: channel-1
         | Port: icahost
         | State: OPEN
         | Counterparty: channel-2
       + Channel: channel-2
         | Port: icahost
         | State: OPEN
         | Counterparty: channel-3
       + Channel: channel-3
         | Port: icahost
         | State: OPEN
         | Counterparty: channel-4
       + Channel: channel-4
         | Port: icahost
         | State: CLOSED
         | Counterparty: channel-1
 2022-07-30T07:05:15.322071Z  INFO ThreadId(01) connection is Open, state on destination chain is Open chain=STRIDE-TESTNET-2 connection=connection-0 counterparty_chain=GAIA
 2022-07-30T07:05:15.322079Z  INFO ThreadId(01) connection is already open, not spawning Connection worker chain=STRIDE-TESTNET-2 connection=connection-0
 2022-07-30T07:05:15.322088Z  INFO ThreadId(01) no connection workers were spawn chain=STRIDE-TESTNET-2 connection=connection-0
 2022-07-30T07:05:15.322098Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=STRIDE-TESTNET-2 counterparty_chain=GAIA channel=channel-0
 2022-07-30T07:05:15.329270Z  INFO ThreadId(01) spawned client worker: client::GAIA->STRIDE-TESTNET-2:07-tendermint-0
 2022-07-30T07:05:15.370754Z  INFO ThreadId(01) done spawning channel workers chain=STRIDE-TESTNET-2 channel=channel-0
 2022-07-30T07:05:15.370788Z  INFO ThreadId(01) channel is CLOSED, state on destination chain is CLOSED chain=STRIDE-TESTNET-2 counterparty_chain=GAIA channel=channel-1
 2022-07-30T07:05:15.370807Z  INFO ThreadId(01) no channel workers were spawned chain=STRIDE-TESTNET-2 channel=channel-1
 2022-07-30T07:05:15.370818Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=STRIDE-TESTNET-2 counterparty_chain=GAIA channel=channel-2
 2022-07-30T07:05:15.393068Z  INFO ThreadId(01) done spawning channel workers chain=STRIDE-TESTNET-2 channel=channel-2
 2022-07-30T07:05:15.393092Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=STRIDE-TESTNET-2 counterparty_chain=GAIA channel=channel-3
 2022-07-30T07:05:15.416351Z  INFO ThreadId(01) done spawning channel workers chain=STRIDE-TESTNET-2 channel=channel-3
 2022-07-30T07:05:15.416385Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=STRIDE-TESTNET-2 counterparty_chain=GAIA channel=channel-4
 2022-07-30T07:05:15.442518Z  INFO ThreadId(01) done spawning channel workers chain=STRIDE-TESTNET-2 channel=channel-4
 2022-07-30T07:05:15.442712Z  INFO ThreadId(01) spawning Wallet worker: wallet::STRIDE-TESTNET-2
 2022-07-30T07:05:15.442745Z  INFO ThreadId(01) connection is Open, state on destination chain is Open chain=GAIA connection=connection-0 counterparty_chain=STRIDE-TESTNET-2
 2022-07-30T07:05:15.442755Z  INFO ThreadId(01) connection is already open, not spawning Connection worker chain=GAIA connection=connection-0
 2022-07-30T07:05:15.442763Z  INFO ThreadId(01) no connection workers were spawn chain=GAIA connection=connection-0
 2022-07-30T07:05:15.442773Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=GAIA counterparty_chain=STRIDE-TESTNET-2 channel=channel-0
 2022-07-30T07:05:15.465813Z  INFO ThreadId(01) spawned client worker: client::STRIDE-TESTNET-2->GAIA:07-tendermint-0
 2022-07-30T07:05:15.493443Z  INFO ThreadId(01) done spawning channel workers chain=GAIA channel=channel-0
 2022-07-30T07:05:15.493482Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=GAIA counterparty_chain=STRIDE-TESTNET-2 channel=channel-1
 2022-07-30T07:05:15.506726Z  INFO ThreadId(01) done spawning channel workers chain=GAIA channel=channel-1
 2022-07-30T07:05:15.506751Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=GAIA counterparty_chain=STRIDE-TESTNET-2 channel=channel-2
 2022-07-30T07:05:15.520773Z  INFO ThreadId(01) done spawning channel workers chain=GAIA channel=channel-2
 2022-07-30T07:05:15.520800Z  INFO ThreadId(01) channel is OPEN, state on destination chain is OPEN chain=GAIA counterparty_chain=STRIDE-TESTNET-2 channel=channel-3
 2022-07-30T07:05:15.533915Z  INFO ThreadId(01) done spawning channel workers chain=GAIA channel=channel-3
 2022-07-30T07:05:15.533939Z  INFO ThreadId(01) channel is CLOSED, state on destination chain is CLOSED chain=GAIA counterparty_chain=STRIDE-TESTNET-2 channel=channel-4
 2022-07-30T07:05:15.533949Z  INFO ThreadId(01) no channel workers were spawned chain=GAIA channel=channel-4
 2022-07-30T07:05:15.534097Z  INFO ThreadId(01) spawning Wallet worker: wallet::GAIA
 2022-07-30T07:05:15.569809Z  INFO ThreadId(01) Hermes has started
```
