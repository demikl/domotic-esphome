# Salon : radar de présence et horloge

[Retour au dépôt](../README.md) · [Configuration ESPHome](../m5-atom-lite-ld2410.yaml)

Le M5 Atom Lite installé dans le salon associe un radar millimétrique LD2410,
un afficheur quatre chiffres TM1637 et l'émetteur infrarouge du M5.

## Présence et réglages du radar

Le LD2410 communique en UART à 256 000 bauds : TX de l'ESP sur GPIO26 et RX
sur GPIO32. Le [package partagé](../common/ld2410.yaml) expose :

- présence globale, cible mobile et cible immobile ;
- distances et niveaux d'énergie des cibles ;
- énergie et seuils des neuf zones de distance `g0` à `g8` ;
- délai de disparition de présence, portées maximales et résolution ;
- mode ingénierie, Bluetooth, version du firmware et commandes de diagnostic.

La détection des cibles immobiles complète celle des mouvements.
Pour l'horloge, le déclencheur local est toutefois **l'apparition d'une cible
mobile**, et non la présence globale.

## Affichage horaire et luminosité

Le TM1637 utilise GPIO21 pour CLK et GPIO25 pour DIO. Il affiche l'heure
Home Assistant en fuseau `Europe/Paris`, avec le séparateur qui clignote.
L'écran est recalculé toutes les 500 ms.

- Un mouvement déclenche un affichage temporaire de 10 secondes ; une nouvelle
  détection peut prolonger cette fenêtre.
- Le capteur Home Assistant `sensor.esp32_rf_861390_luminosity`, fourni par la
  passerelle [ESP32 RF](esp32-rf.md), sert à gérer l'affichage ambiant :
  au-dessus de 5 %, une commande d'allumage permanent est envoyée ; en dessous
  de 5 %, une commande d'extinction est envoyée, sauf pendant un flash temporaire.
- L'intensité est réglée de 0 à 7 : minimum sous 20 %, maximum au-dessus de 60 %,
  interpolation entre les deux. Un réglage manuel est également exposé.
- Un interrupteur Home Assistant permet de commander l'état de l'afficheur.
  Les réactions à la luminosité continuent d'exister ; ce n'est pas un mode
  manuel exclusif.

L'action API `display_mode` affiche `On`, `Off` ou `AUtO` pendant 5 secondes.
Il s'agit d'un message de confirmation fourni par une automatisation externe :
il ne change pas le mode réel de l'horloge. Un nouveau message remplace le
précédent ; les autres valeurs sont refusées avec un avertissement dans les logs.

## Autres fonctions et dépendances

L'émetteur IR sur GPIO12 propose l'action `remote_ir_rc5` et un bouton
« IR standby » (adresse `0x05`, commande `0x0B`). La LED RGB intégrée sur GPIO27
dispose d'effets lumineux ; le bouton GPIO39 la bascule.

L'API est chiffrée via `!secret`. L'heure et la mesure de luminosité viennent de
Home Assistant ; les automatisations qui envoient les messages de mode ne sont
pas dans ce dépôt. Les mesures du radar et la réaction locale aux mouvements
ne nécessitent pas une automatisation Home Assistant.
