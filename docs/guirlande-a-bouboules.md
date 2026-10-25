# Guirlandes de Noël à boules

[Retour au dépôt](../README.md) · [Configuration ESPHome](../guirlande-a-bouboules.yaml)

## Principe physique

Le module anime deux guirlandes dont les boules ne sont pas adressables
individuellement. Dans un sens du courant, les boules **rouges et oranges**
s'allument ; dans l'autre, ce sont les boules **vertes et bleues**.
Un pont en H permet d'inverser la polarité et de doser les deux groupes.

ESPHome représente chaque guirlande comme une lumière `cwww` à deux sorties.
Les noms `cold_white` et `warm_white` sont ici une abstraction logicielle :
il ne s'agit pas de LED blanches à température de couleur réglable.
On commande deux groupes de couleurs par guirlande, pas chacune des quatre
couleurs indépendamment.

## Matériel et sorties

Carte : ESP32 `az-delivery-devkit-v4`, framework ESP-IDF.

| Connexion | GPIO |
| --- | --- |
| Guirlande 1, deux commandes du pont | GPIO27 et GPIO26 |
| Guirlande 2, deux commandes du pont | GPIO25 et GPIO33 |
| LED de l'interrupteur | GPIO32 |
| Bouton/interrupteur physique | GPIO14 |

Les quatre sorties LEDC vers les ponts sont limitées à 50 % de puissance,
avec un déphasage de 180° sur une sortie de chaque paire.
La configuration seule ne décrit pas le modèle du pont, l'alimentation ou
les protections électriques : les vérifier sur le montage réel.

## Commandes et animations

Les deux guirlandes démarrent allumées avec l'effet **Bip bip 2**.
Elles restent commandables séparément depuis Home Assistant, avec la même
liste d'effets : groupes de couleurs fixes, séquences Bip bip, effets aléatoires
et alternances lentes ou rapides.

Le bouton physique éteint les deux guirlandes ou les rallume avec leur effet
par défaut. Son voyant est aussi recalé chaque seconde pour indiquer si au
moins une guirlande est allumée ; sa sortie est également utilisée pour
la signalisation d'état ESPHome.

## Désynchronisation

Un interrupteur logiciel active la désynchronisation de **Bip bip 2**.
Toutes les 23 secondes pour la première guirlande et 31 secondes pour la seconde,
un tirage de 20 % peut déclencher une pause de 100 à 400 ms, puis relancer
l'effet. Cela évite un défilement trop systématiquement identique.

L'effet `_Desync_pause` sert à ce mécanisme ; il n'est pas destiné à être choisi
manuellement. Le script ne relance pas Bip bip 2 si l'utilisateur a changé
d'effet ou éteint la guirlande entre-temps.
