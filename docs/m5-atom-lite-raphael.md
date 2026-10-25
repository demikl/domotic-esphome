# Expérimentation : M5 Atom Lite de Raphaël

[Retour au dépôt](../README.md) · [Configuration ESPHome](../m5-atom-lite-raphael.yaml)

Ce module teste une petite lumière RGB commandable depuis Home Assistant
sur un M5 Atom Lite :

- une LED WS2812B intégrée sur GPIO27 ;
- bouton physique sur GPIO39, qui bascule cette LED ;
- effets aléatoires, scintillement, arc-en-ciel, stroboscope et respiration
  lumineuse avec montées et descentes de luminosité ;
- diagnostics Wi-Fi, durée de fonctionnement et commande de redémarrage.

Un émetteur infrarouge est déclaré sur GPIO12, mais aucune commande IR
spécifique n'est définie dans ce fichier.

Il s'agit d'une expérimentation, pas d'un module actuellement utilisé.
Le fichier utilise des anciennes syntaxes ESPHome (carte dans `esphome`,
FastLED et OTA), à vérifier ou migrer avant une nouvelle compilation.
