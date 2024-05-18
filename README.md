![Car Crash Tv (20)](https://github.com/okannako/initia/assets/73176377/2d7c386b-9d20-4095-8552-6e6d2b6fae24)

## Sistem Gereksinimleri
- CPU: 4 Cores
- RAM: 16GB
- SSD: 1 TB SSD
- Network Bandwidth: 100 Mbps
- Operating System: Ubuntu 20.04LTS/22.04

## Güncellemeler
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y && cd $HOME
```

## Go Yükleme
```
ver="1.22.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Inıtıa Dosyaları İndirme ve Yükleme
```
cd $HOME
rm -rf initia
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.14
make build
make install
```

## Node Yükleme
```
MONIKER="MONIKERISMI"
initiad init $MONIKER --chain-id initiation-1
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"20\"/" $HOME/.initia/config/app.toml
wget https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json -O $HOME/.initia/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/initia-testnet/addrbook.json > $HOME/.initia/config/addrbook.json
```

## Seeds ve Peer Eklemek
```
SEEDS="cd69bcb00a6ecc1ba2b4a3465de4d4dd3e0a3db1@initia-testnet-seed.itrocket.net:51656"
PEERS="3f472746f46493309650e5a033076689996c8881@initia-testnet.rpc.kjnodes.com:17959, aee7083ab11910ba3f1b8126d1b3728f13f54943@initia-testnet-peer.itrocket.net:11656,dae24101e66118156701ca2ad80b45bf008939a2@158.220.96.33:26656,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,633775ca828f8fc7f5c689a8c950664e7f198223@184.174.32.188:26656,9f0ae0790fae9a2d327d8d6fe767b73eb8aa5c48@176.126.87.65:22656,7317b8c930c52a8183590166a7b5c3599f40d4db@185.187.170.186:26656,6a64518146b8c902ef5930dfba00fe61a15ec176@43.133.44.152:26656,a45314423c15f024ff850fad7bd031168d937931@162.62.219.188:26656,35e4b461b38107751450af25e03f5a61e7aa0189@43.133.229.136:26656,3762209e1122580ce885d4e0fcc091d8602708b1@31.220.89.237:26656,d5b2a720e062d5b6b3ced8dfd3d57bf0a05080ce@217.76.57.121:51656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.initia/config/config.toml
```

## Snapshot
```
initiad tendermint unsafe-reset-all --home $HOME/.initia
if curl -s --head curl https://testnet-files.itrocket.net/initia/snap_initia.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/initia/snap_initia.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.initia
    else
  echo no have snap
fi
```

## Servis Ekleme
```
sudo tee /etc/systemd/system/initiad.service > /dev/null <<EOF
[Unit]
Description=Initia node
After=network-online.target

[Service]
User=root
WorkingDirectory=$HOME/.initia
ExecStart=$(which initiad) start --home $HOME/.initia
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Node Başlatmak
```
sudo systemctl daemon-reload && sudo systemctl enable initiad
sudo systemctl restart initiad && sudo journalctl -u initiad -f
```

- Node nuzun bloğu ile ağın bloğu (https://scan.testnet.initia.xyz/initiation-1/blocks) eşitlendiğinde aşağıdaki işlemlere devam edebilirsiniz.

## Cüzdan Oluşturma
- Aşağıdaki kodda cüzdanismi yazan yeri silip cüzdanımıza hangi ismi vermek istiyorsak onu yazıyoruz ve kodu giriypruz. Gelen ekranda şifre belirleyip enterlayıp 2. bir defa şifreyi girdikten sonra gelen kelimeler ve cüzdan adresini MUTLAKA kaydediyoruz.
```
initiad keys add cüzdanismi
```

## Test Tokenı Almak
- Daha önceden https://discord.gg/initia Discord kanalına girip faucet-verification kanalında onaylama yapmanız gerekiyor.
- Test tokenı almak için https://faucet.testnet.initia.xyz/ siteye gidip cüzdan adresimizi yazıp submit yaptığımız zaman bizim discord ile cüzdan adresini bağlıyor ve fauceti yolluyor. Gelmesi 2-3 dakika sürebilir
- Test tokenlarınızı https://scan.testnet.initia.xyz/ adresinde cüzdan adresinizi aratarak görebilirsiniz.

## Validator Oluşturmak
```
initiad tx mstaking create-validator \
--amount 1000000uinit \
--pubkey $(initiad tendermint show-validator) \
--moniker "Monikerismi" \
--from wallet \
--chain-id initiation-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--identity "Keybase ID" \
--details "Ayrıntı" \
--website "Website" \
--fees 6000uinit
```

#Validator oluşturduktan sonra https://forms.gle/LtxqGcJPNYXwwkxP9 adresine gidip formu mutlaka 19 Mayıs 17:59'a kadar gönderin.
