# Expérimentation : prototype RF du salon

[Retour au dépôt](../README.md) · [Configuration ESPHome](../salon.yaml)

Ce prototype sur ESP8266 NodeMCU v2 explore la réception et l'émission de
commandes de prises radio :

- récepteur sur D2, avec décodage `rc_switch` dans les logs ;
- émetteur sur D4, sans porteuse modulée ;
- deux commutateurs, `RF-P2` et `RF-P5`, avec répétition des commandes 24 fois.

Les essais mélangent des impulsions brutes, le protocole 1 et une définition
de durée d'impulsion personnalisée. Les commutateurs n'ont pas de mécanisme
de confirmation de l'état réel des prises.

Cette configuration ancienne est une expérimentation, distincte de la
[passerelle ESP32 RF utilisée](esp32-rf.md). Elle conserve notamment l'ancienne
déclaration `platform`/`board` dans `esphome` et une ancienne syntaxe OTA :
prévoir une adaptation avant utilisation avec un ESPHome récent.
