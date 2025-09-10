# LED Strip Matrix - Effets d'Ambiance Asynchrones

## 📋 Description du Projet

Ce projet ESPHome contrôle un ensemble de lumières RGB connectées à un ESP32 M5 Atom Matrix, avec des effets d'ambiance sophistiqués qui créent des atmosphères immersives tout en évitant la synchronisation monotone.

## 🔧 Configuration Matérielle

- **Microcontrôleur** : ESP32 M5 Atom Matrix (pico32)
- **Matrice LED** : 25 LEDs WS2812 (5x5) sur GPIO27
- **Strip LED** : 2 LEDs P9813 via SPI (MOSI: GPIO25, CLK: GPIO21)
  - LED 0 : "Porte bas"  
  - LED 1 : "Porte haut"
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

- **Matrice LED** : Indices 0-24 (une seed par LED)
- **Porte bas** : Indice 30 + XOR fixe
- **Porte haut** : Indice 31 + XOR différent

### Pourquoi Appliquer un XOR sur les Seeds ?

Le XOR (`^`) est appliqué sur les seeds des partitions pour **garantir l'unicité** des séquences pseudo-aléatoires :

```cpp
uint32_t seed = id(ambiance_seeds)[30] ^ 0xA55A5A5A;  // Porte bas
uint32_t seed = id(ambiance_seeds)[31] ^ 0x55AA55AA;  // Porte haut
```

**Problème sans XOR** : Si deux LEDs utilisent la même seed de base, elles produiraient des séquences identiques, créant une synchronisation indésirable.

**Solution XOR** :

- Mélange les bits de façon déterministe mais complexe
- Garantit que même des seeds proches donnent des résultats très différents
- Les constantes `0xA55A5A5A` et `0x55AA55AA` sont choisies pour maximiser la dispersion binaire
- Permet de réutiliser une seed de base tout en créant des variantes uniques

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
```cpp
// Logique Forêt avec transitions
auto zone_cycle = fmod((seeds[i] + millis() * 0.00001) * 2.0, 3.0);
uint16_t hue;
if (zone_cycle < 1.0) {
  // Ambre (35°) → Mousse (80°)
  hue = 35 + (80-35) * zone_cycle;
} else if (zone_cycle < 2.0) {
  // Mousse (80°) → Vert (130°)  
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
- **Transitions sinusoïdales** : `0.5f + 0.5f * sinf()` crée des passages fluides (courbe en S)

**Avantages pour peu de LEDs** :

- Chaque LED visite **toutes les gammes** au lieu d'être figée dans une seule
- **Variété maximisée** même avec 2-3 LEDs seulement
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
- Variation de `slow` par update : ≈ 0.0083 (soit ~0.8% du cycle)
- Sur la gamme de teinte (ex: 80°→130°), cela représente ≈ 0.4° par update
- **Résultat** : L'œil perçoit un mouvement continu alors que ce sont des steps discrets très fins

### Résultat Visuel de la Forêt

**Promenade temporelle en forêt** : Chaque LED traverse lentement tous les aspects d'un environnement forestier - du sous-bois ambré aux mousses dorées, jusqu'au feuillage vert profond, puis retour cyclique.

**Optimisé pour peu de LEDs** : Même avec 2-3 LEDs seulement, l'effet reste riche grâce aux transitions fluides entre toutes les gammes colorimètriques, évitant la monotonie des affectations fixes.

## 🌊 Effet "Ambiance Aquatique"

### Concept Aquatique

Évocation des profondeurs marines avec vagues lentes et bioluminescence occasionnelle.

### Algorithme Aquatique

**Période** : 45-47 secondes (houle marine)

```cpp
float hue = 160.0f + 80.0f * slow; // 160-240° (vert d'eau → bleu profond)
```

**Palette de couleurs** :

- **Vert d'eau → Bleu profond** (160° - 240°) : Couleur de base évolutive
- **Cyan pulsé** (190° - 220°) : Vagues et reflets (conditions spéciales)
- **Violet bioluminescent** (265°) : Éclats rares (probabilité 1/1000 sur 900ms)

**Effets spéciaux** :

- **Scintillement** : Modulation sinusoïdale haute fréquence (700ms)
- **Éclats violets** : Déclenchement aléatoire selon la seed (toutes les 18-20 secondes)
- **Saturation variable** : 50% - 90% selon la profondeur simulée

### Résultat Aquatique

Houle apaisante avec des variations de bleus, interrompue par de rares éclairs violets évoquant la bioluminescence planctonique.

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

### Désynchronisation Garantie

**Mécanismes anti-synchronisation** :

1. **Seeds différentes** par LED (esp_random())
2. **Périodes légèrement décalées** (60s, 60.5s, 61s...)
3. **XOR de mélange** pour les partitions
4. **Phases initiales aléatoires** dans les calculs temporels

### Optimisations Performance

**Fréquences d'update configurées** :

- Forêt : 300ms (effet lent, économise le CPU)
- Aquatique : 400ms (houle lente)
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
