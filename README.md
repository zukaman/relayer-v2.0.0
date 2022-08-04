# Set up a v2.0.0-rc4 relayer
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

## Download go-v2 relayer
```
     git clone https://github.com/cosmos/relayer.git
     cd relayer && git checkout v2.0.0-rc3
     make install
 
```
## Dowload go-v2 config
```
wget -O $HOME/.relayer/config/config.yaml https://raw.githubusercontent.com/zukaman/relayer-v2.0.0-rc4/main/configs/config.yaml
```

## Import  keys for the relayer to use when signing and relaying transactions
   ```
    rly keys restore stride [key-name] "mnemonic words here"
    rly keys restore GAIA [key-name] "mnemonic words here"
   ```
## Make changes to config.yaml
```
nano $HOME/.relayer/config/config.yaml
```
Set a memo to identify your relay. Replace `<your discord name>` on your values
You need to change the IP addresses of your nodes and RPC ports
Replace `<your key name>`  with the name of the keys specified in the previous paragraph

## Create go-v2 relayer service file
 (copy and paste into the terminal with one command)
```
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=HERMES
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start task --log-format logfmt --processor events

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
sudo systemctl enable rlyd
sudo systemctl restart rlyd && journalctl -u rlyd -f
```
