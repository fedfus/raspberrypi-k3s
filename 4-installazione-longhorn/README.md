# Installazione e configurazione [Longhorn](https://longhorn.io/)

## Prerequisiti 

### installazione [open-iscsi](https://github.com/open-iscsi/open-iscsi)
Per il corretto funzionamento di longhorn è necessario installare open-iscsi

```
sudo apt-get install open-iscsi -y
```
---
### mount del disco o chiavetta usb
Su ogni nodo del cluster individuare la pen drive o l'hdd usb con il comando 

``` 
lsblk -f

```
l'output sarà simile al seguente, ovvero il disco è su sda1

```
NAME        FSTYPE LABEL  UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda
└─sda1      vfat          32AA-7491
mmcblk0
├─mmcblk0p1 vfat   boot   4BBD-D3E7                             203.4M    19% /boot
└─mmcblk0p2 ext4   rootfs 45e99191-771b-4e12-a526-0779148892cb   22.2G    19% /
```
Nel caso in cui ci siano più dischi collegati, si otterà un output come il seguente 

```
NAME        FSTYPE LABEL                    UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda
├─sda1      ntfs   Riservato per il sistema 8AECD174ECD15AD1
├─sda2      ntfs                            6AAAD59BAAD563DB
├─sda3      ntfs                            5EA00C2FA00C1067
└─sda4      ntfs   Dati                     E860A66D60A641E4
sdb
└─sdb1      vfat                            A6E7-94DC
mmcblk0
├─mmcblk0p1 vfat   boot                     5DE4-665C                             203,4M    19% /boot
└─mmcblk0p2 ext4   rootfs                   7295bbc3-bbc2-4267-9fa0-099e10ef5bf0   15,2G    42% /
```
in questo caso il disco è su sdb1 .

Una volta individuato il disco, eseguire un wipe

```
sudo wipefs -a /dev/sda1
```
formattare la usb in ext4 con il comando 

```
sudo mkfs.ext4 /dev/sda1
```
Longhorn utilizza di default `/var/lib/longhorn` come default data path. Per utilizzare la configurazione di default sarà necessario creare la directory con il comando

```
sudo mkdir /var/lib/longhorn
```
e quindi montare il device con il comando 

```
sudo mount /dev/sda1 /var/lib/longhorn
```

L'ultimo step è quello di automatizzare il mount del disco usb all'avvio: per far questo, anziché utilizzare il mount point /dev/sd** utilizzeremo l'UUID del dispositivo. Questo permette di poter montare il disco sempre nella stessa posizione anche se dovesse cambiare la porta usb al quale è collegato.
Recuperiamo quindi l'UUID con il comando

```
sudo blkid -s UUID -o value /dev/sda1
```
il quale ci darà in risposta l'UUID associato (es.: `f8054aec-29a6-4b14-9db0-dc19ea8ba425` ) .
A questo punto andiamo a modificare il file `/etc/fstab` aggiungendo in coda il seguente comando:

```
echo 'UUID=f8054aec-29a6-4b14-9db0-dc19ea8ba425 /var/lib/longhorn ext4 defaults 0 2' >> /etc/fstab
```

oppure, è possibile accorpare le due istruzioni in un unico comando da lanciare come super user, e facendo attenzione al path /dev/sd**

```
echo 'UUID='$(blkid -s UUID -o value /dev/sda1) '/var/lib/longhorn ext4 defaults 0 2' >> /etc/fstab
```
---

## Installazione Longhorn

Di seguito procederemo con l'installazione della versione 1.2.2 utilizzando `kubectl` ma è possibile anche installarlo tramite `helm` o direttamente dal Rancher Catalog App. Per questi ultimi due [rimando alla documentazione ufficiale](https://longhorn.io/docs/1.2.2/deploy/install/)

Per procedere, basterà eseguire il comando 

```
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.2.2/deploy/longhorn.yaml
```
verrà creato in automatico un nuovo namespace `longhorn-system`
