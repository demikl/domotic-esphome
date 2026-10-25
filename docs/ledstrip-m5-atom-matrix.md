# Bureau : signal de réunion et éclairage d'ambiance

[Retour au dépôt](README.md) · [Configuration ESPHome](ledstrip-m5-atom-matrix.yaml)

## Usage principal : statut de réunion

Installé dans le bureau, ce M5 Atom Matrix commande deux bandeaux RGB qui
encadrent chacun un vitrage de la porte vitrée. Leur combinaison de couleurs
indique le statut de réunion : microphone en écoute et/ou webcam en captation.

**Il ne s'agit pas de deux LED physiques ni de rubans à pixels adressables.**
Chaque bandeau contient plusieurs dizaines de LED RGB commandées ensemble :
toutes prennent la même couleur. Les deux contrôleurs P9813 sont adressables
séparément et apparaissent donc comme deux pixels logiques dans ESPHome.
Les partitions `porte_bas` (index 0) et `porte_haut` (index 1) permettent de
commander chaque bandeau indépendamment.

Le YAML expose les lumières, mais ne détecte pas lui-même l'activité du
microphone ou de la webcam et ne définit pas la correspondance des couleurs.
Cette logique relève du pilotage externe, non inclus dans ce dépôt.
La matrice intégrée de 25 LED reste, elle, réellement adressable pixel par pixel.

## Commandes et ambiances disponibles

En complément du signal de réunion, cinq boutons logiciels appliquent un effet
à la matrice et aux deux bandeaux : **Ambiance Forêt**, **Ambiance Aquatique**,
**Ambiance Cheminée**, **Forêt Lumineuse** et **Forêt Scintillante**.
Un autre bouton éteint les trois lumières.

Les graines aléatoires sont initialisées au démarrage ; les calculs d'effets
locaux utilisent des décalages et des périodes différentes pour faire évoluer
les lumières sans une synchronisation systématique. Le bouton physique GPIO39
est exposé comme capteur, sans action locale définie dans ce YAML.

Les sections ci-dessous conservent les notes d'implémentation des ambiances.
Les extraits illustratifs ne remplacent pas la configuration YAML, qui fait foi.

## 📋 Description du Projet

Ce projet ESPHome contrôle un ensemble de lumières RGB connectées à un ESP32 M5 Atom Matrix, avec des effets d'ambiance sophistiqués qui créent des atmosphères immersives tout en évitant la synchronisation monotone.

## 🔧 Configuration Matérielle

- **Microcontrôleur** : ESP32 M5 Atom Matrix (pico32)
- **Matrice LED** : 25 LEDs WS2812 (5x5) sur GPIO27
- **Bandeaux RGB non adressables** : 2 contrôleurs P9813 via SPI (MOSI: GPIO25, CLK: GPIO21)
  - Pixel logique 0 : bandeau entier "Porte bas"
  - Pixel logique 1 : bandeau entier "Porte haut"
- **Bouton** : GPIO39 (inversé)
- **Réseau** : IP fixe 192.168.1.206

## 🎨 Système d'Ambiances Asynchrones

### Principe de Fonctionnement

Le système utilise une approche innovante pour créer des ambiances cohérentes mais non-monotones :

1. **Seeds aléatoires uniques** : Chaque LED/lumière possède sa propre graine pseudo-aléatoire
2. **Algorithmes temporels décalés** : Périodes légèrement différentes pour éviter les coïncidences
3. **Activation synchronisée** : Tous les effets démarrent ensemble via les boutons
4. **Trajectoire indépendante** : Chaque lumière suit sa propre trajectoire colorimétrique
5. **Randomisation temporelle** : Système de hash pseudo-aléatoire pour des événements imprévisibles

### Architecture Technique

```yaml
globals:
  - id: ambiance_seeds        # Tableau de 32 graines aléatoires
    type: uint32_t[32]
  - id: ambiance_inited       # Flag d'initialisation
    type: bool
```

Les seeds sont générées au démarrage avec `esp_random()` et attribuées selon :

- **Matrice LED** : indices dérivés du pixel selon l'effet
- **Porte bas** : indice 30, avec un XOR pour certains effets
- **Porte haut** : indice 31, avec un XOR pour certains effets

### Pourquoi Appliquer un XOR sur les Seeds ?

Certains effets appliquent un XOR (`^`) pour obtenir une variante de la seed
de chaque partition. Par exemple, pour l'ambiance aquatique :

```cpp
uint32_t seed = id(ambiance_seeds)[30] ^ 0xA55A5A5A;  // Porte bas
uint32_t seed = id(ambiance_seeds)[31] ^ 0xA55A5A5A;  // Porte haut
```

Le XOR transforme les bits de façon déterministe ; il ne garantit ni l'unicité
des seeds entre partitions ni l'indépendance des séquences. La diversité repose
aussi sur les indices distincts, les décalages et les périodes propres à chaque
lumière. L'effet Forêt utilise directement les seeds, sans XOR.

## 🌲 Effet "Ambiance Forêt"

### Concept Artistique

Simulation d'une promenade en forêt avec jeux de lumière à travers le feuillage.

### Algorithme Technique

**Période** : 60-61 secondes (très lente pour un effet naturel)

```cpp
float slow = fmodf((t / 60000.0f) + ((seed & 0xFF) / 255.0f), 1.0f);
float branch = ((seed >> 8) & 0xFF) / 255.0f;
```

### Comment la Variable `slow` Affecte la Teinte

La variable `slow` est la **fonction temporelle normalisée** qui détermine la progression cyclique de la couleur :

**Composition de `slow`** :

- `t / 60000.0f` : Temps écoulé en minutes (60 000 ms = 1 minute, mais cycle de 60s car fmodf à 1.0)
- `(seed & 0xFF) / 255.0f` : Offset unique par LED (0.0 à 1.0)
- `fmodf(..., 1.0f)` : Maintient le résultat entre 0.0 et 1.0

**Effet combiné de `slow` et `zone_cycle`** :

```cpp
// Logique Forêt avec transitions
auto zone_cycle = fmod((seeds[i] + millis() * 0.00001) * 2.0, 3.0);
uint16_t hue;
if (zone_cycle < 1.0) {
  // Ambre (35°) → Mousse (80°)
  hue = 35 + (80-35) * zone_cycle;
} else if (zone_cycle < 2.0) {
  // Mousse (80°) → Vert (130°)  
  hue = 80 + (130-80) * (zone_cycle - 1.0);
} else {
  // Vert (130°) → Ambre (35°)
  hue = 130 + ((35 + 360) - 130) * (zone_cycle - 2.0);  // +360 pour transition circulaire
  if (hue >= 360) hue -= 360;
}
```

**Double temporalité** :

- **`slow` (60s)** : Modulation fine de chaque gamme de couleur
- **`zone_cycle` (3 min)** : Transition majeure entre les 3 zones colorimètriques
- **Modulation sinusoïdale** : le mélange des teintes suit les expressions définies dans le YAML

**Avantages pour peu de LEDs** :

- Chaque LED visite **toutes les gammes** au lieu d'être figée dans une seule
- **Variété de couleurs** même avec seulement deux bandeaux commandés globalement
- Transitions douces pour des changements esthétiques

**Système de Zones avec Transitions** :

L'effet utilise un **cycle de zones à 3 minutes** (`zone_cycle`) qui fait passer chaque LED à travers 3 gammes colorimètriques :

- **Zone 1 (0-33%)** : Amber → Mousse (30°-40° → 45°-50°)
- **Zone 2 (33-66%)** : Mousse → Vert (45°-50° → 80°-130°)  
- **Zone 3 (66-100%)** : Vert → Amber (80°-130° → 30°-40°, avec bouclage)

**Transitions fluides** : Utilisation de `sin()` pour créer des passages doux entre zones, évitant les sauts brusques de couleur.

**Variations dynamiques** :

- Saturation : 55% - 90% (fonction de la seed)
- Luminosité : Sinusoïde lente avec phase unique par LED

### Différence entre Période (60s) et Update Interval (300ms)

**Période de 60 secondes** :

- **Cycle complet** de l'algorithme de couleur
- Temps pour que `slow` passe de 0.0 → 1.0 → 0.0
- **Détermine la vitesse** du cycle global de teinte
- Plus longue = effet plus naturel et apaisant

**Update Interval de 300ms** :

- **Fréquence de recalcul** de l'effet lambda
- ESPHome exécute le code toutes les 300ms et applique **immédiatement** la couleur calculée
- **Aucun lissage automatique** - changements directs HSV→RGB
- Assez rapide pour paraître fluide mais économise le CPU

**Pourquoi l'effet paraît fluide** :

- La période de 60s fait que `slow` varie très peu entre deux updates (300ms)
- Variation de `slow` par update : 300 / 60 000 = 0,005 (soit 0,5 % du cycle)
- **Résultat** : L'œil perçoit un mouvement continu alors que ce sont des steps discrets très fins

### Résultat Visuel de la Forêt

**Promenade temporelle en forêt** : Chaque LED traverse lentement tous les aspects d'un environnement forestier - du sous-bois ambré aux mousses dorées, jusqu'au feuillage vert profond, puis retour cyclique.

**Adapté à deux bandeaux** : chaque bandeau parcourt les gammes colorimétriques
dans son ensemble, sans animation spatiale entre les LED physiques du ruban.

## 🌊 Effet "Ambiance Aquatique"

### Concept Aquatique

Évocation des profondeurs marines avec vagues lentes et bioluminescence occasionnelle.

### Algorithme Aquatique

**Période de base** : 45-47 secondes (houle marine lente)

```cpp
float base_hue = 160.0f + 80.0f * slow; // 160-240° (vert d'eau → bleu profond)
```

**Système de Randomisation Aquatique** :

L'effet utilise des **hash pseudo-aléatoires** pour créer des événements marins imprévisibles :

```cpp
uint32_t cyan_hash = (t + seed * 1664525 + i * 1013904223) ^ (seed >> 4);
uint32_t biolumi_hash = (t * 11 + seed * 2147483647 + i * 16807) ^ (seed >> 12);
```

**Paramètres configurables** (constants en en-tête) :

- **Pulsations cyan** : Intervalles 8-25s, durées 1.2-3.5s, teintes cyan variables (180° - 215°)
- **Éclats bioluminescents** : Intervalles 15-45s, durées 0.8-2s, teintes violettes (260° - 280°)
- **Variations par partition** : Paramètres légèrement différents pour désynchronisation

**Palette de couleurs dynamique** :

- **Base marine** (160° - 240°) : Houle vert d'eau → bleu profond
- **Pulsations cyan** (180° - 215°) : Sauts non-prévisibles vers cyan avec variations
- **Bioluminescence** (260° - 280°) : Éclats violets-magenta totalement aléatoires

**Effets spéciaux randomisés** :

- **Scintillement de base** : Modulation sinusoïdale (700ms) pour texture marine
- **Sauts cyan imprévisibles** : Transitions brutales depuis base marine vers cyan pulsé
- **Éclats bioluminescents** : Événements violets aux temporisations vraiment aléatoires
- **Saturation adaptative** : 50% - 95% selon l'événement en cours

### Résultat Aquatique

Houle marine naturelle (vert d'eau → bleu profond) ponctuée d'événements totalement imprévisibles : sauts soudains vers des cyans lumineux et éclats de bioluminescence violette, créant un écosystème aquatique vivant avec temporisations vraiment aléatoires.

## 🔥 Effet "Ambiance Cheminée"

### Concept Cheminée

Simulation réaliste de flammes avec variations thermiques et étincelles.

### Algorithme Cheminée

**Période** : 900-950ms (flicker rapide des flammes)

```cpp
float heat = 0.4f + 0.6f * (0.5f + 0.5f * sinf(phase));
if ((random_uint32() & 0x1F) == 0) heat = 1.0f; // Flicker aléatoire
```

**Modèle thermique** :

- **Chaleur de base** : Sinusoïde avec phase unique par LED
- **Flickers aléatoires** : Pics de chaleur (probabilité 1/32 à chaque frame)
- **Mapping couleur** : Plus chaud = plus jaune, moins saturé

**Palette de couleurs** :

- **Orange chaud** (15° - 40°) : Fonction inverse de la chaleur
- **Saturation thermique** : 75% → 30% (flamme → blanc chaud)
- **Luminosité** : 15% → 100% (braises → flammes)

**Étincelles** :

- **Déclenchement** : Probabilité 1/95-110 (variable selon la LED)
- **Couleur** : Blanc pur (hue 9°-10°, saturation 5%-6%)
- **Durée** : Une frame (120ms)

### Résultat Cheminée

Flammes dansantes avec variations rapides orange-rouge, ponctuées d'étincelles blanches brèves et imprévisibles.

## 🌲 Effet "Forêt Lumineuse"

### Concept Lumineux

Variation de l'ambiance forestière avec luminosité renforcée et teintes minérales-végétales diversifiées, ponctuée de rayons de soleil et d'éclats lumineux subtils.

### Algorithme Lumineux

**Période** : 75 secondes (cycle lent minéral)

```cpp
float mineral_cycle = fmodf((t / 120000.0f) + ((seed >> 8) & 0xFF) / 255.0f, 1.0f); // 2 minutes
```

**Système de Zones Minérales** :

L'effet utilise un **cycle minéral à 2 minutes** qui fait alterner chaque LED entre 3 gammes de teintes naturelles :

- **Zone 1 (0-33%)** : Terre sombre rougeâtre (15° - 25°)
- **Zone 2 (33-66%)** : Mousse profonde (55° - 70°)
- **Zone 3 (66-100%)** : Écorce grise-verte (35° - 55°)

**Effets spéciaux** :

- **Rayons de soleil** : Incursions dorées-vertes brèves (45° - 70°) sur 15% des LEDs
- **Éclats lumineux faibles** : Impulsions lumineuses fréquentes avec variation vers tons chauds
- **Luminosité garantie** : Seuil minimal de 65% pour visibilité constante

**Variations dynamiques** :

- Saturation : 45% - 70% (teintes minérales naturelles)
- Luminosité : 65% - 87% (base élevée + variations)
- Modulation temporelle : Oscillations lentes sur 18-27 secondes

### Résultat Lumineux

Ambiance forestière claire avec teintes terre-mousse-écorce, rehaussée par de brefs rayons dorés et des éclats subtils, maintenant une visibilité constante tout en conservant la richesse chromatique naturelle.

## ✨ Effet "Forêt Scintillante"

### Concept Scintillant

Évolution avancée de l'ambiance forestière combinant base végétale stable avec événements lumineux randomisés : flickers chauds et éclats brillants aux temporisations imprévisibles.

### Algorithme Scintillant

**Update très rapide** : 25ms (pour capturer les micro-événements)

**Base végétale** :

```cpp
float base_hue = 80.0f + 40.0f * (0.5f + 0.5f * sinf(base_oscillation * 2.0f * 3.14159f));
```

**Teinte végétale** : Vert profond à vert-jaune (80° - 120°) avec dérive lente

**Système de Randomisation Temporelle** :

L'effet utilise des **hash pseudo-aléatoires** pour générer des événements imprévisibles :

```cpp
uint32_t flicker_hash = (t + seed * 1103515245 + i * 12345) ^ (seed >> 8);
float flicker_random = (flicker_hash % 1000) / 1000.0f;
```

**Paramètres configurables** (constants en en-tête de chaque partition) :

- **Flickers chauds** : Intervalles 300-1200ms, durées 50-180ms, teintes rouge-orange (5° - 35°)
- **Éclats brillants** : Intervalles 4-15s, durées 70-220ms, teinte orange vif (20-25°)
- **Variations par partition** : Paramètres légèrement différents pour désynchronisation

**Calcul d'événements** :

Chaque LED/partition calcule indépendamment ses événements basés sur :

- Hash temporel unique (t + seed + multiplicateurs)
- Intervalles aléatoires dans plages définies
- Durées variables pour chaque événement
- Progression sinusoïdale de l'intensité

**Luminosité adaptative** :

- Base végétale : 50% - 65%
- Avec flickers : +25% max
- Avec éclats : 75% - 100%
- Seuil minimal maintenu à 50%

### Résultat Scintillant

Base végétale stable (verts naturels) ponctuée d'événements lumineux totalement imprévisibles : flickers chaleureux rouge-orange et éclats brillants orange vif, avec temporisations vraiment aléatoires créant un effet vivant et naturel sans répétitions perceptibles.

## 🔘 Interface de Contrôle

### Buttons Disponibles

| Button | Action | Description |
|--------|--------|-------------|
| **🌲 Ambiance Forêt** | `on_press` | Active l'effet forêt sur toutes les lumières |
| **🌊 Ambiance Aquatique** | `on_press` | Active l'effet aquatique sur toutes les lumières |
| **🔥 Ambiance Cheminée** | `on_press` | Active l'effet cheminée sur toutes les lumières |
| **🌲 Forêt Lumineuse** | `on_press` | Active l'effet forêt lumineux sur toutes les lumières |
| **✨ Forêt Scintillante** | `on_press` | Active l'effet forêt scintillant sur toutes les lumières |
| **⚫ Éteindre toutes les ambiances** | `on_press` | Éteint toutes les lumières |

### Avantages de l'Interface Button

- **Action instantanée** : Un clic = activation immédiate
- **Pas d'état persistant** : Évite la confusion des toggles
- **UX simplifiée** : Interface épurée dans Home Assistant
- **Compatibilité vocale** : "Appuie sur Ambiance Forêt"

## 🎯 Points Techniques Avancés

### Mécanismes de désynchronisation

**Mécanismes anti-synchronisation** :

1. **Seeds différentes** par LED (esp_random())
2. **Périodes légèrement décalées** (60s, 60.5s, 61s...)
3. **XOR de mélange** pour les partitions
4. **Phases initiales aléatoires** dans les calculs temporels

### Optimisations Performance

**Fréquences d'update configurées** :

- Forêt : 300ms (effet lent, économise le CPU)
- Aquatique : 25ms (pulsations et éclats)
- Cheminée : 120ms (flicker rapide nécessaire)
- Forêt Lumineuse : 200ms (rayons et éclats fréquents)
- Forêt Scintillante : 25ms (capture micro-événements randomisés)

**Calculs HSV optimisés** :

- Conversion HSV→RGB native ESPHome
- Précalcul des offsets seed
- Réutilisation des valeurs temporelles

## 🚀 Utilisation et Déploiement

### Compilation et Upload

```bash
# Compilation
podman run --rm -v $(pwd):/config -it ghcr.io/esphome/esphome:stable compile ledstrip-m5-atom-matrix.yaml

# Upload OTA
podman run --rm -v $(pwd):/config -it ghcr.io/esphome/esphome:stable upload ledstrip-m5-atom-matrix.yaml
```

### Intégration Home Assistant

1. Le device apparaît automatiquement via mDNS
2. Les 6 boutons sont exposés comme entités `button.*`
3. Les 3 lumières sont disponibles individuellement
4. Utilisable dans automations, scènes, et dashboards

## 📊 Spécifications Mémoire

**Utilisation Flash** : ~924 KB (50.4% du total)
**Utilisation RAM** : ~35.5 KB (10.8% du total)  
**Performance** : Compatible ESP32 standard, pas de surcharge CPU

## 🎨 Personnalisation Avancée

### Personnalisation des Palettes

Les couleurs peuvent être ajustées en modifiant les valeurs HSV dans les effets `addressable_lambda`.

### Ajout de Nouveaux Effets

1. Créer un nouvel `addressable_lambda` sur chaque partition (matrix, porte_bas, porte_haut)
2. Ajouter un bouton template correspondant
3. Utiliser une seed unique du tableau global avec XOR différent par partition
4. Pour effets randomisés : utiliser des hash temporels avec constantes configurables

### Réglage des Temporisations

Modifier les constantes de période dans les calculs `fmodf()` pour accélérer/ralentir les effets.

---

**Version** : ESPHome 2025.8.4  
**Dernière mise à jour** : Septembre 2025  
**Licence** : Open Source
