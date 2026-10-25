# Expérimentation : matrice LED à texte défilant

[Retour au dépôt](../README.md) · [Configuration ESPHome](../m5-atom-matrix-raphael.yaml)

Ce prototype transforme un M5 Atom Matrix en afficheur de messages MQTT sur
ses 25 LED WS2812B (matrice 5 × 5 sur GPIO27).

## Fonctionnement

Au démarrage, la matrice lance un arc-en-ciel à 80 % de luminosité pendant
5 secondes, puis passe à l'effet **Scrolling Info**.
Le texte provient du topic `m5atom_marquee/text` ; en l'absence de message,
elle fait défiler `WAITING FOR DATA`.

Le rendu est recalculé toutes les 120 ms par le composant C++
[MatrixLedScrollingDisplay](../includes/my_matrixled_scrolling_display.h),
avec la police [DejaVuSans](../resources/DejaVuSans.ttf).
Un texte commençant par `+` s'affiche en vert, par `-` en rouge ; sinon,
il utilise la couleur courante de la lumière.

Le bouton GPIO39 est exposé, sans action locale définie. Des effets lumineux,
les diagnostics Wi-Fi et le redémarrage sont également disponibles.

## Limites du prototype

Le broker MQTT est configuré par son adresse locale dans le YAML.
Cette expérimentation conserve l'ancien moteur FastLED, des références à ses
objets internes et l'ancienne syntaxe ESPHome/OTA : ne pas la considérer comme
une configuration prête à compiler sur une version actuelle. La taille des
messages doit aussi être maîtrisée avant réutilisation : le code de rendu
emploie un tampon fixe de 64 caractères sans copie bornée.
