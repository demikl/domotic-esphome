# Chambre des parents : éclairage de tour de lit

[Retour au dépôt](../README.md) · [Configuration ESPHome](../parents_bedroom.yaml)

## Usage et matériel

Ce module commande le bandeau RGB du tour de lit, nommé « Eclairage tête de lit »
dans ESPHome. Un ESP32 NodeMCU-32S pilote les trois canaux par PWM :
rouge sur GPIO19, vert sur GPIO5 et bleu sur GPIO18.
Le bandeau forme une seule lumière RGB, pas une suite de pixels adressables.

Deux boutons, côté Soso (GPIO25) et côté Mick (GPIO26), permettent de l'utiliser
sans Home Assistant. Un récepteur infrarouge sur GPIO27 complète les commandes.
Le module expose aussi un PIR sur GPIO4 et les mesures de température,
d'humidité et de pression d'un BME280 sur I²C (SDA GPIO21, SCL GPIO22).

## Fonctionnement quotidien

- **Clic simple sur un bouton** : éteint la lumière si elle est allumée ;
  sinon, l'allume avec la couleur et la luminosité du préréglage de ce côté.
- **Double clic** : si la lumière est éteinte, l'allume à 7 % avec la couleur
  du préréglage ; sinon, alterne entre une faible luminosité de 7 % et 100 %.
- **Appui maintenu** : après 3 secondes, des flashs blanc/rouge annoncent
  l'enregistrement. En maintenant encore environ 2 secondes, la couleur et la
  luminosité courantes sont mémorisées pour ce côté, puis confirmées par des
  flashs rouge, vert et bleu. Relâcher pendant l'avertissement permet d'annuler.

Les deux préréglages sont des lumières RGB virtuelles, sans sortie physique,
dont les valeurs sont restaurées au démarrage. Ils sont aussi réglables depuis
Home Assistant.

À chaque allumage, une extinction automatique est prévue : attente de 5 minutes,
puis diminution progressive sur 10 minutes. La lumière principale reste éteinte
au redémarrage (`ALWAYS_OFF`).

Le PIR publie une présence après filtrage de 1 seconde ; il **n'allume pas
automatiquement le bandeau** dans cette configuration.

## Télécommande infrarouge 44 touches

38 touches sont actives ; les six DIY restent libres. Les codes LG 32 bits
correspondent au modèle décrit sur
[Wood's Good](https://woodsgood.ca/projects/2015/02/13/rgb-led-strip-controllers-ir-codes/).

| Touches | Action |
| --- | --- |
| Marche/arrêt | Bascule avec transition de 2,5 secondes. |
| Luminosité + / − | Réglage par pas de 5 %, y compris pendant une animation. |
| R/G/B/W et les 16 pastilles | Couleurs fixes et arrêt de l'animation ; nuances approximatives selon le ruban. |
| Flèches R/G/B | Ajustement du canal de 10 %, borné entre 0 et 100 %, et retour à une couleur fixe. |
| JUMP3 / JUMP7 | Succession franche de 3 / 7 couleurs. |
| FADE3 / FADE7 | Fondus entre 3 / 7 couleurs. |
| FLASH | Alternance blanc/noir sans modifier la luminosité choisie. |
| AUTO | Fondus aléatoires parmi sept couleurs ; un second appui arrête l'effet. |
| QUICK / SLOW | Durée d'une étape par pas de 100 ms, entre 100 ms et 5 secondes ; 1 seconde au démarrage. |
| Lecture/pause | Fige/reprend l'animation ou relance la dernière sélectionnée ; JUMP3 au démarrage. |

L'effet `Telecommande` est calculé localement toutes les 50 ms.
Ses réglages d'animation sont volatils. QUICK/SLOW n'affecte pas les effets
Random/Flicker proposés séparément à Home Assistant, et lecture/pause ne bascule
pas Flicker.

Les touches DIY ne sont pas implémentées. Quelques idées possibles restent :
coucher de soleil, veilleuse ambrée, aurore boréale, bougie, respiration lumineuse
ou rappel des préréglages.

## Intégration Home Assistant

L'API est chiffrée avec une clé référencée par `!secret`.
Les commandes physiques et infrarouges ainsi que les animations sont locales.
Les capteurs et lumières sont exposés à Home Assistant.

Un effet « Alarme via mouvement et ouverture » utilise les agrégats
`binary_sensor.any_motion_sensor` et `binary_sensor.any_opening_sensor` :
flashs bleus pour les mouvements, rouges pour les ouvertures, avec des
séquences blanches intermédiaires. C'est un **effet à sélectionner**, pas une
alarme automatiquement armée par ces capteurs.
