# Passerelle RF 433 MHz

[Retour au dépôt](../README.md) · [Configuration ESPHome](../esp32-rf.yaml)

## Usage principal

Cet ESP32 NodeMCU-32S fait le lien entre Home Assistant et des prises secteur
220 V télécommandées par radio. Il émet les codes de commande et écoute les
télécommandes physiques, sans intervenir directement sur le secteur.

Le [package RF partagé](../common/rf433.yaml) définit cinq prises logiques
`SW1` à `SW5`, avec des codes ON/OFF distincts en protocole `rc_switch` 7.
Chaque commande est répétée dix fois. À la réception d'un code connu, l'état
du commutateur correspondant est mis à jour dans ESPHome.

**Cet état reste supposé** : les prises ne renvoient pas d'accusé de réception.
Entendre une télécommande ou émettre un ordre ne prouve pas que la prise a
effectivement commuté.

## Matériel et autres fonctions

| Fonction | Connexion |
| --- | --- |
| Émetteur RF | GPIO23 |
| Récepteur RF, mentionné RXB6 dans les notes de montage | GPIO32 |
| BME280 : température, humidité, pression | SDA GPIO18, SCL GPIO19 |
| Sonde de température DS18B20 | Bus 1-Wire GPIO13 |
| Mesure analogique de luminosité | GPIO34 |
| Buzzer RTTTL | GPIO16 |
| LED monochrome et effets lumineux | GPIO0 |

La luminosité est lue chaque minute et convertie en pourcentage par une
régression linéaire réglée avec deux points de calibration min/max. Des
commandes permettent de forcer la mesure et d'enregistrer le maximum courant.
Ce pourcentage est un indicateur relatif, pas une mesure en lux.
Il est notamment réutilisé par l'[horloge du salon](m5-atom-lite-ld2410.md).

Le package RF contient également les commandes d'un projecteur d'étoiles :
bascule marche/arrêt, vitesse du moteur et minuterie 2/4/6 h ou remise à zéro,
en protocole 1. Sa commande marche/arrêt étant une bascule unique, son état peut
se désynchroniser du projecteur réel.

Le buzzer expose un service `play_rtttl` et un bouton de test Mario 1-UP.

## Adaptation

Les codes radio sont propres aux équipements de cette installation : les
observer et les remplacer pour d'autres prises. Les commentaires en tête du
YAML conservent des TODO et corrections de câblage ; ils ne constituent pas
un schéma de montage validé. Vérifier les niveaux électriques du récepteur
avant toute connexion à l'ESP32.
