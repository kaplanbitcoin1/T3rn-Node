
![1](https://github.com/user-attachments/assets/26bffc22-fd35-4388-975a-72de3009ec75)


* Let's set up the Node for the [T3rn](https://www.t3rn.io/) project with a few operations. Since the system requirements are low, you can run it on any server.


* NOTE: You need to hold more than 0.1 tokens on these networks. (Arbitrum-sepolia, Base-sepolia, Optimism-sepolia, L1rn)


* [Faucet](https://faucet.brn.t3rn.io/)



*  Update the Server

```console
sudo apt update -q && sudo apt upgrade -qy
```


* Download the necessary binary file

```console
LATEST_VERSION=$(curl -s https://api.github.com/repos/t3rn/executor-release/releases/latest | grep 'tag_name' | cut -d\" -f4)
EXECUTOR_URL="https://github.com/t3rn/executor-release/releases/download/${LATEST_VERSION}/executor-linux-${LATEST_VERSION}.tar.gz"
curl -L -o executor-linux-${LATEST_VERSION}.tar.gz $EXECUTOR_URL
```


* Extract the binary to the appropriate file path

```console
tar -xzvf executor-linux-${LATEST_VERSION}.tar.gz
rm -rf executor-linux-${LATEST_VERSION}.tar.gz
cd executor/executor/bin || exit
```



* Enter a single command and write the private key 🧨


```console
read -p "Metamask Özel Anahtarınızı girin (0x ön eki olmadan): " PRIVATE_KEY_LOCAL
PRIVATE_KEY_LOCAL=${PRIVATE_KEY_LOCAL#0x}
```

* Değişkenleri ayarlayalım

```console
export NODE_ENV=testnet
export LOG_LEVEL=info
export LOG_PRETTY=false
export ENABLED_NETWORKS='arbitrum-sepolia,base-sepolia,optimism-sepolia,l1rn'
```


* Service dosyasını oluşturalım

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


* Son adımlar



```console
sudo systemctl daemon-reload
sudo systemctl enable executor.service
sudo systemctl start executor.service
```

* İşlem tamamdır 🐅


* Log kontrol

```console
sudo journalctl -u executor.service -f
```

* Ufak bir yıldız bırakırsan sevinirim 🐅

