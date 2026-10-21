# TICMeter — firmware maison

Fork personnel du firmware [GammaTroniques/TICMeter](https://github.com/GammaTroniques/TICMeter)
(V3.2.13), le projet d'origine n'étant plus maintenu. Utilisé en mode MQTT / Home Assistant,
alimenté uniquement par le Linky (TIC historique, monophasé).

Licence d'origine conservée : CC BY-NC 4.0 (voir `LICENSE`), auteur original Dorian Benech / GammaTroniques.

## Modifications (branche `maison`)

| Problème | Correction |
|---|---|
| Chaque envoi (~35 s) refaisait un DHCP complet + vérification ARP : ~7 lignes de log par cycle sur le routeur | Après un premier DHCP, IP / passerelle / DNS / canal / BSSID sont gardés en RAM et réutilisés. Cache invalidé à tout échec de connexion ou d'envoi, DHCP complet forcé toutes les 6 h. `CONFIG_LWIP_DHCP_RESTORE_LAST_IP=y`, `CONFIG_LWIP_DHCP_DOES_ARP_CHECK` désactivé. |
| Découverte Home Assistant publiée une seule fois par démarrage : un message perdu n'était jamais renvoyé (capteurs absents ou périmés) | Toute la découverte est republiée toutes les 12 h. |
| `pref` (kVA) annoncé avec la classe `power`, refusée par HA | Classe `apparent_power`. |
| Vérification de mise à jour toutes les 4 h vers update.gammatroniques.fr / GitHub | Désactivée (`CONFIG_TICMETER_OTA_CHECK`). |

Le Wi-Fi reste coupé entre deux envois (contrainte d'alimentation par le Linky) : il reste donc
3 lignes Auth / Assoc / Disassoc par cycle côté routeur.

## Compilation

```sh
docker run --rm -v $PWD:/project -w /project/firmware -e HOME=/tmp espressif/idf:v5.2.1 \
  bash -c 'git config --global --add safe.directory "*" && idf.py build'
```

## Flash (NVS préservée : Wi-Fi, MQTT et réglages conservés)

Toujours sauvegarder d'abord la flash complète :

```sh
esptool.py --chip esp32c6 -b 460800 read_flash 0 0x400000 sauvegarde.bin
esptool.py --chip esp32c6 -b 460800 write_flash --flash_mode dio --flash_size 4MB --flash_freq 80m \
  0x0 bootloader.bin 0x8000 partition-table.bin 0x10000 ota_data_initial.bin \
  0x17000 storage.bin 0x30000 TICMeter.bin
```

Retour arrière : `esptool.py --chip esp32c6 -b 460800 write_flash 0x0 sauvegarde.bin`.

Les binaires compilés sont attachés aux [releases](../../releases) du fork.
