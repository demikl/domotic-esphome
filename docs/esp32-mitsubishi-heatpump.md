# Interface avec la pompe à chaleur Mitsubishi Ecodan

[Retour au dépôt](../README.md) · [Configuration ESPHome](../esp32-mitsubishi-heatpump.yaml)

Ce module relie la pompe à chaleur Mitsubishi Ecodan à Home Assistant pour
centraliser ses mesures, suivre son fonctionnement et agir sur ses réglages.
La carte configurée est une Lolin S2 Mini (ESP32-S2).

## Communication

Le composant externe
[tobias-93/esphome-ecodan-heatpump](https://github.com/tobias-93/esphome-ecodan-heatpump)
est chargé depuis sa branche `main`.
La liaison UART est configurée à 2 400 bauds avec parité paire :
TX sur GPIO18 en drain ouvert, RX sur GPIO16 avec pull-up et tampon de
1 024 octets. Une LED verte de connecteur RJ45, sur GPIO15, bascule lors
des séquences de debug UART pour signaler l'activité.

Le YAML décrit les broches de l'ESP, mais pas le brochage complet du connecteur
de la PAC ni un circuit d'adaptation électrique. Ce n'est donc pas un guide
de raccordement direct à la machine.

## Informations remontées

- États : mode de fonctionnement, chauffage/refroidissement, phase de chauffe,
  dégivrage, mode vacances et programmation d'eau chaude sanitaire (ECS).
- Températures : pièce de la zone 1, extérieur, départ et retour d'eau,
  retour de gaz et ballon d'ECS.
- Fonctionnement : code erreur, fréquence, puissance de sortie, durée de marche
  et débit d'eau.
- Énergie de la veille : consommation et production séparées pour le chauffage,
  le refroidissement et l'ECS, ainsi que les dates des relevés énergétiques.

Ces valeurs sont celles fournies par le composant Ecodan ; le YAML ne recalcule
pas un COP ni un bilan énergétique.

## Commandes exposées

- Marche/arrêt et forçage de la production d'ECS.
- Choix du mode général et du mode d'eau chaude.
- Consigne du ballon d'ECS.
- Consigne d'ambiance de la zone 1 et consigne de température de départ.

L'heure Home Assistant est déclarée dans la configuration. Les scénarios de
pilotage externes ne sont pas inclus. La dépendance externe suivant `main`,
sa compatibilité est à vérifier avant une recompilation ou une adaptation à
un autre contrôleur Ecodan.
