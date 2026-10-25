# Expérimentation : tableau de bord e-paper « pro »

[Retour au dépôt](../README.md) · [Configuration ESPHome](../eink_dashboard_pro.yaml)

Cette variante du [dashboard e-paper](eink_dashboard.md) utilise le même écran
Waveshare 4,2 pouces en niveaux de gris et le même brochage SPI.
Elle explore une mise en page 400 × 300 plus riche, avec heure, météo,
prévisions et symboles d'état de la maison.

## Données et affichage

Des imports Home Assistant sont déclarés pour `weather.forecast_home`
(température, humidité, pression, vent, état et prévisions), ainsi que
`sensor.indoor_temperature` et `sensor.indoor_humidity`.
Cependant, la présence de ces imports ne signifie pas que tous les widgets
sont dynamiques : le dessin contient encore des valeurs et états de
démonstration, notamment l'icône de soleil. Il reste à relier le rendu aux
données réelles avant un usage quotidien.

L'affichage est dessiné directement dans une lambda ESPHome ; malgré un
commentaire mentionnant LVGL, aucune interface LVGL n'est configurée.
Le rafraîchissement est prévu toutes les 10 minutes, 2 secondes après
synchronisation horaire, et via le bouton logiciel « Refresh Dashboard ».

## Maintenance et limites

Le bouton GPIO0 empêche la veille et demande un rafraîchissement.
Un composant `deep_sleep` définit 30 minutes de sommeil, mais **aucune durée
d'éveil ni action d'entrée en veille n'est configurée** : cette seule déclaration
ne met donc pas en place un cycle autonome de veille.

Le pilote externe `github://pr#6209`, les polices distantes et les entités
Home Assistant sont à vérifier avant compilation. Comme la version simple,
cette configuration est une expérimentation et partage avec elle le nom
d'appareil et l'adresse IP.
