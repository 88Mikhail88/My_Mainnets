## Install node Kyve ```chain-id kyve-1```

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
peers="7b3e38a97fdbebe73fd977d7da420019925a80cd@192.168.29.248:26656,100e5f097ebb73ecf0df415e419f3319cfc8458c@167.99.141.216:26656,f645af14c553021f838115caacd6f530a54233fa@192.168.1.177:26656,0fe8e7419225639ec2775e52952dfe74534275c5@135.181.215.62:26656,31afcf1856538f28afcc11e9ce78d32e981ab4ff@10.92.47.216:26656,787701b475be5896b65afa69a6417a38ef2156f1@172.19.0.2:26656,358bbd2745d4547fd34a233dcfed207763cd8b9e@172.26.10.37:26656,e23311fce57e1fafa6672275fa1b390fe3860808@162.19.138.20:26656,85885f970418412191d921fd493b68581e33ccd7@192.168.12.53:26656,38ef1bd70583c8831c3c0e07fad7e1bab56515cc@68.183.143.17:26656,6b1d074c77f8671032245eb801d9c88ea145eb94@172.17.0.2:26656,3d1747891f4e6c41f1a6c8acf04330cd4daa4763@135.181.223.115:26656,59fcea08fe65920f39055b31b4bab5d5d0d9c126@211.219.19.72:26656"
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
