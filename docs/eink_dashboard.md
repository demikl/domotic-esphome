# Expérimentation : tableau de bord e-paper

[Retour au dépôt](../README.md) · [Configuration ESPHome](../eink_dashboard.yaml)

Cette maquette graphique utilise un ESP32 et un écran Waveshare e-paper
4,2 pouces, modèle `4.20in-v2`, en quatre niveaux de gris.
Elle dessine une interface avec météo, heure/date et éléments de prévision.

L'heure vient de Home Assistant en fuseau `Europe/Paris`, mais les informations
météorologiques du rendu sont des **valeurs de démonstration**.
Les exemples d'import de capteurs météo Home Assistant sont commentés :
ce n'est pas un dashboard météo opérationnel.

L'écran est actualisé toutes les 5 minutes, 10 secondes après synchronisation
horaire, ou par le bouton logiciel « Refresh E-Ink Display ».

## Connexions et dépendances

- SPI : CLK GPIO18, MOSI GPIO23.
- Écran : CS GPIO5, DC GPIO17, BUSY GPIO4, RESET GPIO16.
- LED de statut sur GPIO2.
- Polices Roboto via Google Fonts et icônes Material Design téléchargées.
- Pilote `waveshare_epaper` externe référencé par `github://pr#6209`.

Cette expérimentation doit être vérifiée avec la version ESPHome et la révision
de l'écran utilisées. La [variante « pro »](eink_dashboard_pro.md) explore
une autre présentation ; les deux YAML ont le même nom d'appareil et la même
adresse IP, ils ne sont pas prêts à être déployés comme deux appareils distincts.
