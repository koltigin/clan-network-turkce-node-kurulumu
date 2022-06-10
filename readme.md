# Clan Network Türkçe Node Kurulumu

Clan Network için test ağları deposu

# Minimum sistem gereksinimleri :
```
4GB RAM
50GB+ disk
2 vcpu
```

# Node kontrol ve stake işlemi için linkler:
```
https://secretnodes.com/clan/chains/playstation-2
https://stake-testnet.clan.network/
```

# Gerekli paketleri güncelleyip varsa mevcut yeni sürümlerine güncelleriz:

```
sudo apt-get update && sudo apt upgrade -y
```

# Toolchain indiririz:

```
sudo apt install build-essential jq -y
```

# Go kurulumu:

```
wget https://go.dev/dl/go1.18.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz

cat <<EOF >> ~/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF


source ~/.profile
```

Bu komutu girdiğiniz zaman aşağıda ki çıktıyı alırsınız, komut:

```
go version
```

Alacağınız çıktı: 

```
go version go1.18 linux/amd64
```

# Clan network dosyalarını indirme:

```
wget https://github.com/ClanNetwork/clan-network/releases/download/v1.0.4-alpha/clan-network_v1.0.4-alpha_linux_amd64.tar.gz
sudo tar -C /usr/local/bin -zxvf clan-network_v1.0.4-alpha_linux_amd64.tar.gz


git clone https://github.com/ClanNetwork/clan-network
cd clan-network
git fetch origin --tags
git checkout v1.0.4-alpha

make install
```

Gerekli düzenlemeleri yapıyoruz: chainid - peer - monikername

```
CHAIN_ID=playstation-2
```

# TEST yazan kısmı kendinize göre düzenleyin, örnek MONIKER_NAME="KolTigin" :

```
MONIKER_NAME="TEST"
```
```
CHAIN_REPO="https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN_ID" && \
export PEERS="$(curl -s "$CHAIN_REPO/persistent-peers.txt")"
```

# Bu komuttan sonra aşşağıdaki gibi bir çıktı almalısınız, komut:
```
echo $PEERS
```

Örnek çıktı: be8f9c8ff85674de396075434862d31230adefa4@35.231.178.87:26656,0cb936b2e3256c8d9d90362f2695688b9d3a1b9e@34.73.151.40:26656,e85dc5ec5b77e86265b5b731d4c555ef2430472a@23.88.43.130:26656,9d7ec4cb534717bfa51cdb1136875d17d10f93c3@116.203.60.243:26656,3049356ee6e6d7b2fa5eef03555a620f6ff7591b@65.108.98.218:56656,61db9dede0dff74af9309695b190b556a4266ebf@34.76.96.82:26656,d97c9ac4a8bb0744c7e7c1a17ac77e9c33dc6c34@34.116.229.135:26656

# Bu komutu girdikten sonra çıkacak uzun yazıyı not edip saklayın: 

```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uclan\"/" ~/.clan/config/app.toml
cland init $MONIKER_NAME --chain-id=$CHAIN_ID
```

# Genesis dosyasını indiriyoruz ve peer listesini configurasyon dosyasına ekliyoruz:

```
curl https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN_ID/genesis.json > ~/.clan/config/genesis.json
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.clan/config/config.toml
```

# Key name kısmını kendi ismimize göre düzenlemek lazım, örnek: cland keys add RUES , oluşan key bilgilerini kaydedin.

```
cland keys add <key-name>
```

# Clan Networkü başlatıyoruz:

```
apt-get install screen
screen -S Clan
cland start
```

# Screenden ctrl a d ile çıkıp işlemlerin arkada aktif olarak çalışması için service dosyası oluşturuyoruz:

```
sudo tee /etc/systemd/system/cland.service > /dev/null <<'EOF'
[Unit]
Description=Clan daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/cland start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

# Şimdi start emrimizi verelim:
```
sudo -S systemctl daemon-reload
sudo -S systemctl enable cland
sudo -S systemctl start cland
```

# Node durumuna bakmak için
```
systemctl status cland
```

# Node'un sync olup olmadığını anlamak için (false çıktısı almayı unutmayın) : 
```
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

# Validator kurmak için faucetten token alalım:
```
https://faucet-testnet.clan.network/
```

# Validator kurma:
```
cland tx staking create-validator \
  --amount 1000000000uclan \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(cland tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0uclan \
--from <key-name>
```



# Hesaplar:

[Twitter](https://twitter.com/mehmetkoltigin)

[Forum](https://forum.rues.info/index.php)

[Telegram Announcement](https://t.me/RuesAnnouncement)

[Telegram Chat](https://t.me/RuesChat)

[Telegram Node](https://t.me/RuesNode)

[Telegram Node Chat](https://t.me/RuesNodeChat)
