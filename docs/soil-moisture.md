# Expérimentation : mesure d'humidité du terreau

[Retour au dépôt](../README.md) · [Configuration ESPHome](../soil-moisture.yaml)

Ce prototype sur ESP32 NodeMCU-32S vise une mesure intermittente du terreau
avec une consommation réduite et publication MQTT, sans API native
Home Assistant.

## Mesure

GPIO23 alimente le circuit de mesure uniquement pendant l'acquisition.
Après 300 ms de stabilisation, l'ADC GPIO32 est lu dix fois ; une moyenne
glissante publie le résultat, puis l'alimentation du circuit est coupée.
Les valeurs extrêmes 0 et 3,9 sont filtrées.

La conversion linéaire en pourcentage d'humidité est **commentée** :
la valeur publiée est une mesure analogique non calibrée, malgré le nom
« Moisture » et l'icône de pourcentage. Un étalonnage du capteur et du terreau
serait nécessaire pour obtenir un pourcentage interprétable.

## Veille et maintenance

Une acquisition est demandée au démarrage. L'appareil est prévu pour rester
éveillé 120 secondes, puis dormir 15 minutes. Un intervalle de mesure de
3 minutes est aussi déclaré, utile si la veille est empêchée.

Une commande MQTT `ON` sur
`esphome_iot_gardening_1/switch/prevent_deep-sleep_for_ota_action/command`
empêche la veille pour une mise à jour OTA ; sa découverte Home Assistant est
publiée par le module. La commande `OFF` n'est pas traitée : le commentaire
du YAML prévoit un redémarrage après remise à OFF pour reprendre le cycle.

Les messages MQTT de disponibilité sont désactivés afin que les connexions
éphémères n'écartent pas les mesures. Le broker, la calibration et les
identifiants de découverte sont à adapter. Ce fichier reste une expérimentation
avec une ancienne syntaxe ESPHome/OTA.
