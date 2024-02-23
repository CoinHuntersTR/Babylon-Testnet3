# Babylon Kurulum Rehberi

![babylon](https://pbs.twimg.com/profile_banners/1558731723243810816/1691512780/1500x500)



## Sistem gereksinimleri:

**Ubuntu 22.04+**

NODE TİPİ | CPU     | RAM      | SSD     |
| ------------- | ------------- | ------------- | -------- |
| Babylon | 4          | 8         | 160  |
  
  

**Gerekli Güncellemeler ve Kurulum**

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```
**Go Kurulum**

```
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.21.6.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```

**Babylon indiriyoruz**
```
cd $HOME
rm -rf babylon
git clone https://github.com/babylonchain/babylon.git
cd babylon
git checkout v0.8.3
make install
babylond version
```


**MONİKER adımızı giriyoruz.**
> MonikerName yerine istediğiniz ismi arasında boşluk kalmadan giriyoruz.
```
babylond init MonikerName --chain-id=bbn-test-3
```

**Genesis Dosyalarını indiriyoruz.**
```
curl -Ls https://ss-t.babylon.nodestake.org/genesis.json > $HOME/.babylond/config/genesis.json
```

**Addrbook indiriyoruz.**
```
curl -Ls https://ss-t.babylon.nodestake.org/addrbook.json > $HOME/.babylond/config/addrbook.json
```

**Servis Dosyasını Oluşturuyoruz.**
```
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=babylond Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable babylond
```

**Snapshot (Opsiyonel)**
```
SNAP_NAME=$(curl -s https://ss-t.babylon.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss-t.babylon.nodestake.org/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.babylond
```
**Node Tekrar Başlatıyoruz**
```
sudo systemctl restart babylond
```
```
sudo journalctl -u babylond -f --no-hostname -o cat
```


**Node Tekrar Başlatıyoruz**
> Senkronize olup olmadığımızı kontrol etmek için aşağıdaki kodu kullanıyoruz. `false` çıktısı aldığımızda işlem tamamdır. 
```
babylond status 2>&1 | jq .SyncInfo
```



**Babylon Explorer**
> Tüm adımları tamamladıktan sonra [BURADAN](https://explorer.nodestake.org/babylon-testnet/staking) kendi validatorünüzü kontrol edebilirsiniz.
