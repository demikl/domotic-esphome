# Outils pour les schémas de circuits et de câblage

[Retour au dépôt](../README.md)

Comparatif établi le 25 octobre 2026. Aucun outil n'est retenu pour le moment.
Les intégrations MCP et leurs compatibilités seront à vérifier lors du choix.

## Besoin

Documenter les montages du dépôt avec des schémas très visuels :

- cartes et modules reconnaissables ;
- connecteurs et numéros de broches explicites ;
- fils colorés, notamment pour repérer les conducteurs des câbles ;
- noms des signaux et tensions ;
- sources versionnées et modifiables avec l'aide de GitHub Copilot ;
- images consultables directement dans la documentation sur GitHub.

La collaboration avec Copilot peut passer par un format textuel ou par un
serveur MCP permettant de piloter un éditeur graphique.

### Exemples de rendu

- [Câblage CC1101 vers ESP32](https://github.com/zibous/ha-watermeter/blob/master/README.md#wiring-cc1101-to-esp32) :
  représentation ASCII et tableau de correspondance des broches.
- [Circuit Mitsubishi CN105](https://github.com/echavet/MitsubishiCN105ESPHome#circuit-diagram) :
  illustrations de cartes, fils colorés et tableau de raccordement.

Le second exemple est plus proche du rendu recherché.

## Deux types de schémas à distinguer

- **Câblage physique** : quelle broche relier, avec quel fil, sur quelle carte.
- **Schéma électrique** : fonctionnement du circuit, résistances, transistors,
  adaptation des niveaux et alimentations.

Le besoin actuel porte surtout sur le câblage physique. Un dessin de montage
ne remplace pas une validation électrique du circuit.

## Comparatif

| Outil | Rendu et usage | Collaboration avec Copilot | Principale limite |
|---|---|---|---|
| **draw.io / diagrams.net** | Très libre : images de cartes, connecteurs, fils colorés, annotations | Bonne : XML éditable et serveurs MCP disponibles | Ne vérifie pas électriquement le circuit |
| **WireViz** | Câbles, faisceaux, brochages et couleurs des conducteurs | Excellente : description YAML et génération SVG/PNG | Moins adapté à une vue réaliste des cartes |
| **Fritzing** | Montage avec cartes reconnaissables et fils colorés | Possible via le XML du projet, mais moins pratique | Bibliothèque de composants et format complexe |
| **Wokwi** | Montage visuel avec simulation de microcontrôleurs | Très bonne : composants et connexions décrits en JSON | Catalogue de composants et objectif simulation plus contraignants |

## 1. draw.io : premier choix pour les illustrations

[draw.io / diagrams.net](https://www.drawio.com/) offre un bon compromis entre
liberté visuelle, édition manuelle et assistance par Copilot.

Pour un montage ESP32, convertisseur de niveaux et connecteur, il permet de :

- placer des images ou illustrations des cartes ;
- représenter les broches et leurs points de connexion ;
- tracer des fils colorés avec un parcours lisible ;
- annoter les signaux, numéros de broches et tensions ;
- organiser le montage en sous-ensembles et en calques.

Il n'est pas nécessaire que chaque module existe dans une bibliothèque :
une illustration accompagnée de points de connexion correctement placés
peut suffire. Il faut utiliser des illustrations dont la provenance et la
licence permettent la réutilisation.

### Collaboration sans MCP

Copilot peut modifier directement le XML du diagramme. Privilégier du
**XML non compressé**, plus facile à inspecter et à versionner.

Une extension draw.io pour VS Code permet d'ouvrir le document dans un
éditeur graphique. Conserver le source et un export SVG pour l'affichage
dans le Markdown sur GitHub.

### Collaboration avec MCP

Deux projets à examiner :

- [Serveur MCP officiel de draw.io](https://github.com/jgraph/drawio-mcp) :
  génération et ouverture de diagrammes, avec plusieurs modes d'intégration.
- [Serveur communautaire de lgazo](https://github.com/lgazo/drawio-mcp-server) :
  inspection et modification des formes, connexions, labels, pages et calques
  d'un document ouvert.

Le second mérite particulièrement un essai pour modifier un dessin existant
de façon interactive. Son éditeur intégré fonctionne dans le navigateur :
ne pas supposer que le pilotage MCP se connecte automatiquement à l'extension
draw.io de VS Code.

Le serveur officiel propose des modes locaux et un service hébergé. Le choix
du mode détermine notamment si le contenu du diagramme est envoyé au service
draw.io ; examiner ce point avant la configuration.

### Limite

draw.io relie des points graphiques, mais ne garantit ni le bon brochage,
ni la compatibilité des tensions, ni la validité électrique du montage.

## 2. WireViz : premier choix pour les câbles et leurs couleurs

[WireViz](https://github.com/wireviz/WireViz) est spécialisé dans les câbles,
faisceaux et brochages.

Sa description YAML permet de définir :

- les connecteurs et leurs broches ;
- les câbles et leurs conducteurs ;
- les couleurs, y compris des combinaisons de couleurs ;
- les connexions ;
- les longueurs, sections et blindages si nécessaire.

Il produit notamment du SVG, du PNG et une nomenclature. Consulter la
[galerie d'exemples](https://github.com/wireviz/WireViz/blob/master/examples/readme.md)
pour évaluer le rendu.

### Collaboration avec Copilot

Le workflow est simple : modifier le YAML, lancer le générateur et examiner
le résultat. Aucun MCP n'est nécessaire. La génération utilise Python et
Graphviz.

Dans ce dépôt, WireViz serait particulièrement intéressant pour un câble
RJ45, les couleurs de ses paires et les correspondances entre les broches
de chaque extrémité.

### Limite

Le rendu est celui d'une documentation de faisceau plutôt que d'un montage
réaliste avec images de cartes. Le projet vise les câbles et faisceaux
individuels, pas la représentation complète d'un système.

## 3. Fritzing : rendu proche d'un tutoriel électronique

[Fritzing](https://fritzing.org/) propose notamment une vue de montage
« breadboard » avec :

- cartes et composants reconnaissables ;
- fils colorés ;
- placement et cheminement visuels ;
- vues schématique et PCB complémentaires.

C'est une bonne option si le rendu visuel du second exemple est le critère
principal.

### Collaboration avec Copilot

Le fichier de projet `.fzz` est une archive ZIP contenant notamment un
fichier `.fz` en XML et, éventuellement, des composants personnalisés.
Voir la [description du format](https://github.com/fritzing/fritzing-app/wiki/2.2-Sketch-file-format).

La modification par Copilot est donc techniquement possible, mais moins
naturelle que du YAML ou du JSON. Il s'agit plutôt d'un workflow
principalement graphique avec une assistance ponctuelle.

### Limites

- La carte ou le module exact doit exister dans les bibliothèques, ou être
  créé comme composant personnalisé.
- La création de composants peut représenter une part importante du travail.
- Le format du projet est plus complexe à manipuler et à comparer dans Git.

## 4. Wokwi : intéressant pour ajouter de la simulation

[Wokwi](https://wokwi.com/) combine un montage visuel et un simulateur de
microcontrôleurs.

Son [format de diagramme JSON](https://github.com/wokwi/wokwi-docs/blob/main/docs/diagram-format.md)
décrit les composants, leurs positions et rotations, les broches reliées,
ainsi que les couleurs et parcours des fils.

### Collaboration avec Copilot

Copilot peut modifier directement le JSON, sans MCP, tout en conservant
la possibilité d'utiliser l'éditeur graphique.

### Limites

Le catalogue des composants et l'objectif de simulation sont plus
contraignants qu'un éditeur de dessin libre. Vérifier :

- la disponibilité des cartes et modules nécessaires ;
- l'effort de représentation des connecteurs ou modules spécifiques ;
- le workflow d'export d'images pour la documentation.

À privilégier si la simulation d'un ESP32 et de composants pris en charge
fait également partie du besoin.

## Recommandation pour le dépôt

### Commencer avec draw.io

Ne pas ajouter de MCP dans un premier temps :

1. Versionner un source draw.io en XML non compressé.
2. Réutiliser des illustrations de cartes avec une provenance et une licence
   compatibles.
3. Exporter en SVG et intégrer l'image au Markdown.
4. Faire modifier le source par Copilot, puis contrôler le résultat dans
   l'éditeur graphique.

Ajouter un MCP ensuite si le besoin de collaboration interactive le justifie.

### Ajouter WireViz si les câbles deviennent un sujet à part entière

Les deux vues sont complémentaires :

- **draw.io** montre où brancher les éléments ;
- **WireViz** décrit exactement le câble à fabriquer.

Fritzing reste une alternative si le rendu réaliste devient prioritaire.
Wokwi est surtout à retenir si la simulation est également recherchée.

## Règles de documentation à conserver

- Indiquer **couleur + nom du signal + numéro de broche**, jamais la couleur
  seule.
- Préciser si un connecteur est représenté **côté broches ou côté fils** pour
  éviter les inversions en miroir.
- Indiquer les tensions et les éventuelles adaptations de niveaux.
- Conserver un tableau de correspondance des broches en complément du dessin.
- Vérifier le brochage et le circuit réels avant de publier un guide de
  raccordement.

Pour le [module Mitsubishi Ecodan](./esp32-mitsubishi-heatpump.md), le brochage
complet côté PAC et l'adaptation électrique restent à confirmer avant de
produire un schéma de raccordement faisant référence.
