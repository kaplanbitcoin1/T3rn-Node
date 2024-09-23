* T3rn projesinin Node'unu birkaç işlemle kuralım.

* Bu ağlarda 0.1'den fazla token bulundurmak gerekiyor. (arbitrum-sepolia,base-sepolia,optimism-sepolia,l1rn) 

[Faucet](https://holesky.eigenlayer.xyz/operator/0x2B6B967A90985190822EdbbBB1A371Ad28F48bc2)



* Sistemi güncelleyelim

```console
sudo apt update -q && sudo apt upgrade -qy
```


* Gerekli Binary dosyasını indirelim

```console
LATEST_VERSION=$(curl -s https://api.github.com/repos/t3rn/executor-release/releases/latest | grep 'tag_name' | cut -d\" -f4)
EXECUTOR_URL="https://github.com/t3rn/executor-release/releases/download/${LATEST_VERSION}/executor-linux-${LATEST_VERSION}.tar.gz"
curl -L -o executor-linux-${LATEST_VERSION}.tar.gz $EXECUTOR_URL
```


# Binary'i uygun dosya yoluna çıkartalım

```console
tar -xzvf executor-linux-${LATEST_VERSION}.tar.gz
rm -rf executor-linux-${LATEST_VERSION}.tar.gz
cd executor/executor/bin || exit
```



# Tek komut girip Private Key yazalım


```console
read -p "Metamask Özel Anahtarınızı girin (0x ön eki olmadan): " PRIVATE_KEY_LOCAL
PRIVATE_KEY_LOCAL=${PRIVATE_KEY_LOCAL#0x}
```

# Değişkenleri ayarlayalım

```console
export NODE_ENV=testnet
export LOG_LEVEL=info
export LOG_PRETTY=false
export ENABLED_NETWORKS='arbitrum-sepolia,base-sepolia,optimism-sepolia,l1rn'
```


# Service dosyasını oluşturalım

```console
SERVICE_FILE="/etc/systemd/system/executor.service"
sudo bash -c "cat > $SERVICE_FILE" <<EOL
[Unit]
Description=Executor Servisi
After=network.target

[Service]
User=root
WorkingDirectory=/root/executor/executor
Environment="NODE_ENV=testnet"
Environment="LOG_LEVEL=info"
Environment="LOG_PRETTY=false"
Environment="PRIVATE_KEY_LOCAL=0x$PRIVATE_KEY_LOCAL"
Environment="ENABLED_NETWORKS=$ENABLED_NETWORKS"
ExecStart=/root/executor/executor/bin/executor
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
EOL
```


# Servisi başlat



```console
sudo systemctl daemon-reload
sudo systemctl enable executor.service
sudo systemctl start executor.service
```

```console
# Logları göster
sudo journalctl -u executor.service -f
```


