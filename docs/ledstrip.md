# Expérimentation : bandeaux RGB sur ESP8266

[Retour au dépôt](../README.md) · [Configuration ESPHome](../ledstrip.yaml)

Ce prototype de commande d'éclairage de bureau utilise un ESP8266 NodeMCU v2
et deux canaux P9813, représentés par deux pixels logiques dans NeoPixelBus.
Les données passent par D7 et l'horloge par D5.

Contrairement au [module de bureau utilisé](../ledstrip-m5-atom-matrix.md),
la configuration active expose une seule lumière regroupant les deux canaux.
Les partitions « porte bas » et « porte haut », ainsi que les exemples d'effets,
sont commentés et ne sont donc pas disponibles.

Les packages ajoutent un BME280 sur D2/D1, un buzzer RTTTL sur D6 et les
agrégats de mouvement/ouverture Home Assistant. L'effet d'alarme est toutefois
commenté dans la lumière : importer les capteurs ne suffit pas à l'activer.

Il s'agit d'une expérimentation, non d'un module actuellement utilisé.
Vérifier la compatibilité de NeoPixelBus, de l'OTA et des packages partagés
avec la version ESPHome choisie avant de compiler.
