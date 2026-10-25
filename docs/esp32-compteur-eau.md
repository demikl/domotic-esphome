# Télérelève du compteur d'eau

[Retour au dépôt](../README.md) · [Configuration ESPHome](../esp32-compteur-eau.yaml)

Un ESP32 et un récepteur CC1101 captent les trames Wireless M-Bus (wM-Bus)
émises par le compteur d'eau situé en amont de la maison. Le module écoute le
compteur par radio : il n'utilise ni contact à impulsions ni lecture optique.

## Acquisition et données exposées

La configuration utilise le composant externe
[SzczepanLeon/esphome-components](https://github.com/SzczepanLeon/esphome-components),
figé sur la version `4.1.0`, et le décodeur `dme07`.
Un identifiant de compteur et une clé de décodage sélectionnent les trames utiles.
Ces paramètres sont propres au compteur et ne doivent pas être recopiés dans
une autre installation ni diffusés comme exemples.

Les entités publiées sont :

- **eau potable** : index total en m³, avec trois décimales et la classe
  `total_increasing`, exploitable pour le suivi de consommation Home Assistant ;
- **eau potable RF RSSi** : puissance du signal reçu en dBm ;
- **eau potable statut compteur** : statut textuel décodé.

L'index est publié même s'il ne change pas (`force_update`), ce qui permet de
repérer les réceptions successives sans consommation. Après une heure sans
nouvelle mesure, le filtre `timeout` invalide sa valeur pour éviter de présenter
un ancien index comme une mesure actuelle. Le YAML ne calcule pas lui-même
de débit ni de détection de fuite.

## Matériel et dépendances

Carte configurée : `az-delivery-devkit-v4`, framework Arduino.

| Signal CC1101 | GPIO ESP32 |
| --- | --- |
| MOSI | GPIO19 |
| MISO | GPIO5 |
| CLK | GPIO18 |
| CS | GPIO4 |
| GDO0 | GPIO16 |
| GDO2 | GPIO17 |

L'heure provient de SNTP en fuseau `Europe/Paris`. La cadence des mesures dépend
des émissions du compteur, pas d'une interrogation périodique de l'ESP.
Pour réutiliser ce module, vérifier le modèle et le protocole du compteur,
son identifiant et sa clé, ainsi que la compatibilité du composant externe.
