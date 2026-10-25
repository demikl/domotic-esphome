# domotic-esphome

Configurations [ESPHome](https://esphome.io/) de mes modules domotiques maison,
principalement sur ESP32, intégrés à Home Assistant. Ce dépôt rassemble les
firmwares déclaratifs, les composants partagés et les détails de fonctionnement
de montages destinés à l'éclairage, la détection de présence, la télécommande RF
et au suivi des consommations.

L'intérêt est de relier des équipements parfois dépourvus d'interface domotique
(prises RF, compteur d'eau, pompe à chaleur) et de créer des usages sur mesure :
éclairage de lit avec commandes locales, signal de réunion sur une porte vitrée,
horloge sensible aux mouvements ou guirlandes animées.

## Modules réellement utilisés

Chaque fiche décrit l'usage, le matériel, le fonctionnement et les dépendances.

| Module | Usage | Documentation |
| --- | --- | --- |
| [parents_bedroom](parents_bedroom.yaml) | Lumière de tour de lit, boutons avec préréglages et télécommande infrarouge. | [Chambre des parents](docs/parents_bedroom.md) |
| [m5-atom-lite-ld2410](m5-atom-lite-ld2410.yaml) | Radar de présence millimétrique LD2410 et afficheur horaire dans le salon. | [Présence et horloge du salon](docs/m5-atom-lite-ld2410.md) |
| [esp32-rf](esp32-rf.yaml) | Émission/réception RF 433 MHz pour commander des prises secteur télécommandées. | [Passerelle RF](docs/esp32-rf.md) |
| [ledstrip-m5-atom-matrix](ledstrip-m5-atom-matrix.yaml) | Deux bandeaux RGB sur la porte du bureau pour signaler l'activité du microphone et de la webcam. | [Signal lumineux du bureau et ambiances](docs/ledstrip-m5-atom-matrix.md) |
| [esp32-compteur-eau](esp32-compteur-eau.yaml) | Télérelève du compteur d'eau par réception de ses trames Wireless M-Bus avec un CC1101. | [Compteur d'eau](docs/esp32-compteur-eau.md) |
| [esp32-mitsubishi-heatpump](esp32-mitsubishi-heatpump.yaml) | Communication avec la pompe à chaleur Mitsubishi Ecodan : mesures, états et commandes. | [Pompe à chaleur](docs/esp32-mitsubishi-heatpump.md) |
| [guirlande-a-bouboules](guirlande-a-bouboules.yaml) | Deux guirlandes de Noël bicolores par inversion du courant avec un pont en H. | [Guirlandes à boules](docs/guirlande-a-bouboules.md) |

## Expérimentations

Les autres configurations sont des essais ou des prototypes, pas des modules
actuellement utilisés. Certaines conservent une ancienne syntaxe ESPHome ou des
données de démonstration ; leur présence ne garantit pas une compilation avec
une version récente.

| Configuration | Objet | Documentation |
| --- | --- | --- |
| [eink_dashboard](eink_dashboard.yaml) | Maquette de tableau de bord sur écran e-paper Waveshare 4,2 pouces. | [Dashboard e-paper](docs/eink_dashboard.md) |
| [eink_dashboard_pro](eink_dashboard_pro.yaml) | Variante du dashboard avec une mise en page plus riche et des imports Home Assistant. | [Dashboard e-paper « pro »](docs/eink_dashboard_pro.md) |
| [ledstrip](ledstrip.yaml) | Essai de commande de deux canaux RGB P9813 sur ESP8266. | [Bandeaux sur ESP8266](docs/ledstrip.md) |
| [salon](salon.yaml) | Essais de réception et d'émission RF sur ESP8266. | [Prototype RF du salon](docs/salon.md) |
| [m5-atom-lite-raphael](m5-atom-lite-raphael.yaml) | LED RGB avec effets et bouton local sur M5 Atom Lite. | [M5 Atom Lite de Raphaël](docs/m5-atom-lite-raphael.md) |
| [m5-atom-matrix-raphael](m5-atom-matrix-raphael.yaml) | Texte MQTT défilant sur une matrice LED 5 × 5. | [Matrice défilante](docs/m5-atom-matrix-raphael.md) |
| [soil-moisture](soil-moisture.yaml) | Mesure analogique d'humidité du terreau avec MQTT et mise en veille profonde. | [Capteur de terreau](docs/soil-moisture.md) |

## Organisation et adaptation

- Les fichiers YAML à la racine sont les configurations des appareils.
- [common/](common/) mutualise le Wi-Fi, les diagnostics, les bases ESP32/ESP8266,
  le radar LD2410, la RF, le BME280, le buzzer et des effets lumineux.
- [includes/](includes/) contient le composant C++ du texte défilant ;
  [resources/](resources/) contient sa police locale.
- [wled/sdb/](wled/sdb/README.md) contient aussi des presets et playlists WLED
  pour des spots de salle de bain : c'est un complément, pas un module ESPHome.

Ces configurations documentent une installation personnelle, pas des montages
universels prêts à flasher. Avant de les réutiliser, adapter les cartes, brochages,
adresses IP, identifiants d'appareils, entités Home Assistant et paramètres radio.
Les références `!secret` sont à renseigner dans un fichier local `secrets.yaml` ;
vérifier également les paramètres propres à l'installation présents directement
dans les YAML, sans publier de clés ni d'identifiants sensibles.

Les automatisations Home Assistant ne sont pas incluses ici. Les composants
externes, lorsqu'il y en a, sont indiqués dans les fiches concernées. Ce travail
de documentation ne constitue pas une validation électrique ni une compilation
des firmwares.