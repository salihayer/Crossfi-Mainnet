<h1 align="center"> CrossFi Mainnet </h1>


![image](https://github.com/Core-Node-Team/Testnet-TR/assets/91562185/2d6d845d-3c30-495f-a9bb-74b9e56fada6)



 * [Crossfi Website](https://crossfi.org/)<br>
 * [Blockchain Explorer](https://xfiscan.com/validators)/)<br>
 * [Discord](https://discord.gg/crossfi)<br>
 * [Twitter](https://twitter.com/crossfichain)<br>

## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	8|
| RAM	| 16+ GB |
| Storage	| 400 GB SSD |

### 🚧 Update Yapıyoruz
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
### 🚧 Go Kuruyoruz
```
ver="1.21.6"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
### 🚧 Varyasyonlar Hazırlıyoruz
Not: moniker-yaz kısmına moniker adınızı yazınız..
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="moniker-yaz"" >> $HOME/.bash_profile
echo "export CROSSFI_CHAIN_ID="mineplex-mainnet-1"" >> $HOME/.bash_profile
echo "export CROSSFI_PORT="37"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### 🚧 Gerekli Dosyaları Çekelim
```
cd $HOME
mkdir -p $HOME/.mineplex-chain/cosmovisor/genesis/bin
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.1.1/mineplex-2-node._v0.1.1_linux_amd64.tar.gz
tar -xvf mineplex-2-node._v0.1.1_linux_amd64.tar.gz
mv mineplex-chaind crossfid
mv $HOME/crossfid $HOME/.mineplex-chain/cosmovisor/genesis/bin
```
```
sudo ln -s $HOME/.mineplex-chain/cosmovisor/genesis $HOME/.mineplex-chain/cosmovisor/current -f
sudo ln -s $HOME/.mineplex-chain/cosmovisor/current/bin/crossfid /usr/local/bin/crossfid -f
```

### 🚧 Download and install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### 🚧 Service Oluşturuyoruz
```
sudo tee /etc/systemd/system/crossfid.service > /dev/null << EOF
[Unit]
Description=crossfi node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home $HOME/.mineplex-chain
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.mineplex-chain"
Environment="DAEMON_NAME=crossfid"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.mineplex-chain/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable crossfid.service
```
### 🚧 Moniker İnit
NOT: moniker-adi-yaz buraya node/moniker adımızı yazıyoruz
```
crossfid config node tcp://localhost:${CROSSFI_PORT}657
crossfid config keyring-backend os
crossfid config chain-id mineplex-mainnet-1
```
```
crossfid init moniker-adi-yaz --chain-id mineplex-mainnet-1
```
```
git clone https://github.com/crossfichain/mainnet.git
cp -r $HOME/mainnet/config/* $HOME/.mineplex-chain/config
```
### 🚧 Genesis ve Addrbook
```
wget -O $HOME/.mineplex-chain/config/genesis.json https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/Crossfi-Mainnet/genesis.json
wget -O $HOME/.mineplex-chain/config/addrbook.json https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/Crossfi-Mainnet/addrbook.json
```
### 🚧 Port
```
sed -i.bak -e "s%:1317%:${CROSSFI_PORT}317%g;
s%:8080%:${CROSSFI_PORT}080%g;
s%:9090%:${CROSSFI_PORT}090%g;
s%:9091%:${CROSSFI_PORT}091%g;
s%:8545%:${CROSSFI_PORT}545%g;
s%:8546%:${CROSSFI_PORT}546%g;
s%:6065%:${CROSSFI_PORT}065%g" $HOME/.mineplex-chain/config/app.toml
```
### 🚧 Port
```
sed -i.bak -e "s%:26658%:${CROSSFI_PORT}658%g;
s%:26657%:${CROSSFI_PORT}657%g;
s%:6060%:${CROSSFI_PORT}060%g;
s%:26656%:${CROSSFI_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CROSSFI_PORT}656\"%;
s%:26660%:${CROSSFI_PORT}660%g" $HOME/.mineplex-chain/config/config.toml
```
### 🚧 Pruning yapıyoruz
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.mineplex-chain/config/app.toml
```
### 🚧 Ayarlar
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10000000000000mpx"|g' $HOME/.mineplex-chain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mineplex-chain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.mineplex-chain/config/config.toml
```

### 🚧 Snap
```
sudo apt install liblz4-tool

systemctl stop crossfid

cp $HOME/.mineplex-chain/data/priv_validator_state.json $HOME/.mineplex-chain/priv_validator_state.json.backup

curl -L http://37.120.189.81/crossfi_mainnet/crossfi_snap.tar.lz4 | tar -I lz4 -xf - -C $HOME/.mineplex-chain

mv $HOME/.mineplex-chain/priv_validator_state.json.backup $HOME/.mineplex-chain/data/priv_validator_state.json

```
### Node Başlatıyoruz
```
sudo systemctl daemon-reload
sudo systemctl restart crossfid
```
### Logları Görüyoruz
```
sudo journalctl -u crossfid -f
```
### Cüzdan oluşturma
```
crossfid keys add cüzdan-adi
```
### Var Olan Cüzdanın Ekleme
```
crossfid keys add wallet --recover
```

### Validator oluşturma
```
crossfid tx staking create-validator \
--amount 1000000000000000000mpx \
--from wallet \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(crossfid tendermint show-validator) \
--moniker "Adini-yaz" \
--chain-id mineplex-mainnet-1 \
--node http://localhost:36657 \
--identity="" \
--website="" \
--details="" \
--security-contact="" \
--fees 2100000000000000000mpx \
--gas auto \
--gas-adjustment 1.1 \
-y
```


### Hadi Geçmiş Olsun
