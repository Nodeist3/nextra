
# Another-1 Kurulum Rehberi
## Donanım Gereksinimleri
Herhangi bir Cosmos-SDK zinciri gibi, donanım gereksinimleri de oldukça mütevazı.

### Minimum Donanım Gereksinimleri
 - 3x CPU; saat hızı ne kadar yüksek olursa o kadar iyi
 - 4GB RAM
 - 80GB Disk
 - Kalıcı İnternet bağlantısı (testnet sırasında trafik minimum 10Mbps olacak - üretim için en az 100Mbps bekleniyor)

### Önerilen Donanım Gereksinimleri
 - 4x CPU; saat hızı ne kadar yüksek olursa o kadar iyi
 - 8GB RAM
 - 200 GB depolama (SSD veya NVME)
 - Kalıcı İnternet bağlantısı (testnet sırasında trafik minimum 10Mbps olacak - üretim için en az 100Mbps bekleniyor)

## Anone Full Node Kurulum Adımları
### Tek Script İle Otomatik Kurulum
Aşağıdaki otomatik komut dosyasını kullanarak Anone fullnode'unuzu birkaç dakika içinde kurabilirsiniz. 
Script sırasında size node isminiz (NODENAME) sorulacak!


```bash
wget -O ANONE.sh https://raw.githubusercontent.com/Nodeist/Kurulumlar/main/Anone/ANONE && chmod +x ANONE.sh && ./ANONE.sh
```

### Kurulum Sonrası Adımlar

Doğrulayıcınızın blokları senkronize ettiğinden emin olmalısınız. 
Senkronizasyon durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```bash
anoned status 2>&1 | jq .SyncInfo
```

### Cüzdan Oluşturma
Yeni cüzdan oluşturmak için aşağıdaki komutu kullanabilirsiniz. Hatırlatıcıyı (mnemonic) kaydetmeyi unutmayın.
```bash
anoned keys add $ANONE_WALLET
```

(OPSIYONEL) Cüzdanınızı hatırlatıcı (mnemonic) kullanarak kurtarmak için:
```bash
anoned keys add $ANONE_WALLET --recover
```

Mevcut cüzdan listesini almak için:
```bash
anoned keys list
```

### Cüzdan Bilgilerini Kaydet
Cüzdan Adresi Ekleyin:
```bash
ANONE_WALLET_ADDRESS=$(anoned keys show $ANONE_WALLET -a)
ANONE_VALOPER_ADDRESS=$(anoned keys show $ANONE_WALLET --bech val -a)
echo 'export ANONE_WALLET_ADDRESS='${ANONE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export ANONE_VALOPER_ADDRESS='${ANONE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Doğrulayıcı oluştur
Doğrulayıcı oluşturmadan önce lütfen en az 1 an1'ye sahip olduğunuzdan (1 an1 1000000 uan1'e eşittir) ve düğümünüzün senkronize olduğundan emin olun.

Cüzdan bakiyenizi kontrol etmek için:
```bash
anoned query bank balances $ANONE_WALLET_ADDRESS
```
> Cüzdanınızda bakiyenizi göremiyorsanız, muhtemelen düğümünüz hala eşitleniyordur. Lütfen senkronizasyonun bitmesini bekleyin ve ardından devam edin. 

Doğrulayıcı Oluşturma:
```bash
anoned tx staking create-validator \
  --amount 1000000uan1 \
  --from $ANONE_WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(anoned tendermint show-validator) \
  --moniker $ANONE_NODENAME \
  --chain-id $ANONE_ID \
  --fees 250uan1
```



## Kullanışlı Komutlar
### Servis Yönetimi
Logları Kontrol Et:
```bash
journalctl -fu anoned -o cat
```

Servisi Başlat:
```bash
systemctl start anoned
```

Servisi Durdur:
```bash
systemctl stop anoned
```

Servisi Yeniden Başlat:
```bash
systemctl restart anoned
```

### Node Bilgileri
Senkronizasyon Bilgisi:
```bash
anoned status 2>&1 | jq .SyncInfo
```

Validator Bilgisi:
```bash
anoned status 2>&1 | jq .ValidatorInfo
```

Node Bilgisi:
```bash
anoned status 2>&1 | jq .NodeInfo
```

Node ID Göser:
```bash
anoned tendermint show-node-id
```

### Cüzdan İşlemleri
Cüzdanları Listele:
```bash
anoned keys list
```

Mnemonic kullanarak cüzdanı kurtar:
```bash
anoned keys add $ANONE_WALLET --recover
```

Cüzdan Silme:
```bash
anoned keys delete $ANONE_WALLET
```

Cüzdan Bakiyesi Sorgulama:
```bash
anoned query bank balances $ANONE_WALLET_ADDRESS
```

Cüzdandan Cüzdana Bakiye Transferi:
```bash
anoned tx bank send $ANONE_WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000uan1
```

### Oylama
```bash
anoned tx gov vote 1 yes --from $ANONE_WALLET --chain-id=$ANONE_ID
```

### Stake, Delegasyon ve Ödüller
Delegate İşlemi:
```bash
anoned tx staking delegate $ANONE_VALOPER_ADDRESS 10000000uan1 --from=$ANONE_WALLET --chain-id=$ANONE_ID --gas=auto --fees 250uan1
```

Payını doğrulayıcıdan başka bir doğrulayıcıya yeniden devretme:
```bash
anoned tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uan1 --from=$ANONE_WALLET --chain-id=$ANONE_ID --gas=auto --fees 250uan1
```

Tüm ödülleri çek:
```bash
anoned tx distribution withdraw-all-rewards --from=$ANONE_WALLET --chain-id=$ANONE_ID --gas=auto --fees 250uan1
```

Komisyon ile ödülleri geri çekin:
```bash
anoned tx distribution withdraw-rewards $ANONE_VALOPER_ADDRESS --from=$ANONE_WALLET --commission --chain-id=$ANONE_ID
```

### Doğrulayıcı Yönetimi
Validatör İsmini Değiştir:
```bash
anoned tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$ANONE_ID \
--from=$ANONE_WALLET
```

Hapisten Kurtul(Unjail): 
```bash
anoned tx slashing unjail \
  --broadcast-mode=block \
  --from=$ANONE_WALLET \
  --chain-id=$ANONE_ID \
  --gas=auto --fees 250uan1
```


Node Tamamen Silmek:
```bash
sudo systemctl stop anoned
sudo systemctl disable anoned
sudo rm /etc/systemd/system/anone* -rf
sudo rm $(which anoned) -rf
sudo rm $HOME/.anone* -rf
sudo rm $HOME/anone -rf
sed -i '/ANONE_/d' ~/.bash_profile
```
