## Install chain-node Kyve ```chain-id kyve-1```

```bash
cd $HOME
sudo apt update && sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop -y < "/dev/null"
```
#### install go
```bash
cd $HOME
VERSION=1.20.1
wget -O go.tar.gz https://go.dev/dl/go$VERSION.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go.tar.gz && rm go.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```
#### build binaries
```bash
cd $HOME
wget https://github.com/KYVENetwork/chain/releases/download/v1.0.0/kyved_linux_amd64.tar.gz
tar -xvzf kyved_linux_amd64.tar.gz
chmod +x kyved
sudo mv kyved /usr/local/bin/kyved
rm -rf kyved_linux_amd64.tar.gz
```
#### init node and download genesis
```bash
cd $HOME
kyved init node --chain-id kyve-1
wget https://raw.githubusercontent.com/KYVENetwork/networks/main/kyve-1/genesis.json
sudo mv genesis.json ~/.kyve/config/genesis.json
```
#### add peers
```bash
cd $HOME
peers="b950b6b08f7a6d5c3e068fcd263802b336ffe047@18.198.182.214:26656,25da6253fc8740893277630461eb34c2e4daf545@3.76.244.30:26656,146d27829fd240e0e4672700514e9835cb6fdd98@34.212.201.1:26656,fae8cd5f04406e64484a7a8b6719eacbb861c094@44.241.103.199:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kyve/config/config.toml
```
#### create service and start node
```bash
echo "[Unit]
Description=Kyve
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/kyved start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyved.service
sudo mv $HOME/kyved.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```

```bash
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable kyved 

sudo systemctl restart kyved
```
