
# Ambiances WLED pour Salle de Bain — Spots GU10 RGB

Ce dépôt contient :
- un fichier `presets_and_playlists.json`
- une configuration complète de **presets**, **playlists**, **palettes personnalisées**
- une logique de **flicker intelligent basé sur la symbolique de chaque couleur**

Il est optimisé pour un **éclairage d’ambiance dans une salle de bain**, composé de :
- **2 spots encastrés GU10 RGB**
- chacun **pré‑flashé avec WLED** (2 instances indépendantes)
- même fichier de presets/playlists déployé sur les deux

L'objectif :  
Créer une ambiance **relaxante, immersive, chaleureuse**, avec :
- des **transitions lentes (7s)** entre les états
- des **changements de couleur espacés (20 à 45s)** via les playlists
- un **flicker subtil**, adapté à la *matière* que la couleur évoque (feu, eau, terre…)
- des **couleurs désynchronisées** entre les deux spots pour un rendu organique

---

# 🔄 Architecture : 2 instances WLED indépendantes

Chaque spot GU10 a sa propre instance WLED. Pour obtenir un effet **désynchronisé mais cohérent** :

1. **Même playlist** sur les deux instances → palette de couleurs cohérente
2. **Shuffle activé** → ordre des presets aléatoire par instance
3. **Durées variables** → les changements ne tombent jamais en même temps
4. **Pas de DDP/Sync** → chaque spot évolue indépendamment

### Synchronisation du changement de playlist

Pour changer de playlist simultanément sur les deux spots :

**Option 1 : Home Assistant** (recommandé)
```yaml
script:
  wled_playlist_foret:
    sequence:
      - service: light.turn_on
        target:
          entity_id:
            - light.wled_sdb_spot1
            - light.wled_sdb_spot2
        data:
          effect: "Playlist: Foret Tropicale"
```

**Option 2 : API WLED directe**
```bash
# Appliquer la playlist 10 (Forêt) sur les deux spots
curl "http://192.168.1.X/win&PL=10"
curl "http://192.168.1.Y/win&PL=10"
```

**Option 3 : Bouton physique**
Configurer un GPIO sur un des spots pour envoyer une requête HTTP à l'autre.

---

# 🧠 Logique du flicker contextuel

Chaque couleur est associée à une *matière* :
- 🔥 feu → flicker fort  
- 🌊 eau → flicker moyen à fort  
- 🌿 végétation → flicker léger  
- 🌳 terre / bois → flicker quasi nul  
- 🍬 couleurs “candy / néon” → flicker léger  
- 🌈 couleurs spa → variable

Le flicker est réalisé via l’effet **WLED Candle Multi (fx = 102)**,  
car chaque LED reçoit son propre pattern indépendant — parfait pour deux pixels distincts.

Paramètres :
- `sx` = vitesse du flicker  
- `ix` = intensité du flicker  

---

# 🌱 Ambiance 1 : Forêt tropicale

| Preset | Couleur | Matière | Flicker |
|--------|---------|----------|---------|
| 101 | Vert sombre | Feuillage dense | Léger (sx2 ix1) |
| 102 | Vert clair | Feuillage éclairé | Léger (sx2 ix2) |
| 103 | Bleu‑vert | Eau stagnante | Moyen (sx3 ix2) |
| 104 | Turquoise | Eau en mouvement | Moyen-fort (sx4 ix3) |
| 105 | Brun | Terre / tronc | Très faible (sx1 ix0) |

---

# 🍬 Ambiance 2 : Bonbons & sucreries

| Preset | Couleur | Matière symbolique | Flicker |
|--------|---------|--------------------|---------|
| 201 | Rose sucré | Lumière néon | Léger |
| 202 | Bleu pastel | Glow froid | Léger-moyen |
| 203 | Jaune doux | Bonbon translucide | Léger |
| 204 | Menthe | Liquide mentholé | Moyen |
| 205 | Violet candy | LED décorative | Léger |

---

# 🔥 Ambiance 3 : Feu de cheminée

| Preset | Couleur | Matière | Flicker |
|--------|---------|----------|---------|
| 301 | Ambre vif | Flamme vive | Fort |
| 302 | Rouge sombre | Braise | Fort |
| 303 | Ambre clair | Flamme moyenne | Fort |
| 304 | Rouge brun | Charbon | Moyen |
| 305 | Ambre orangé | Flamme modérée | Fort |

---

# 🌈 Ambiance 4 : Rouge — Ambre — Violet — Turquoise

| Preset | Couleur | Matière | Flicker |
|--------|---------|----------|---------|
| 401 | Rouge | Chaleur | Moyen-fort |
| 402 | Ambre | Flamme | Fort |
| 403 | Violet | Lumière artificielle | Faible |
| 404 | Turquoise | Eau lumineuse | Fort |
| 405 | Magenta | Lumière décorative | Moyen |

---

# ▶️ Playlists

| ID | Nom | Presets |
|----|-----|---------|
| 10 | Forêt tropicale | 101–105 |
| 11 | Bonbons | 201–205 |
| 12 | Feu de cheminée | 301–305 |
| 13 | Rouge Ambre Violet Turquoise | 401–405 |

Chaque ambiance a sa propre playlist :
- durée par preset : **20–45s** (en dixièmes de seconde dans le JSON)
- transitions : **7s** (`transition: 700`)
- boucle infinie activée (`repeat: 0`, `r: true`)

---

# 📦 Fichier JSON

Le fichier `presets_and_playlists.json` utilise le **format natif WLED** :
- clés numériques = IDs des presets (`"101"`, `"102"`, etc.)
- playlists aux IDs `10–13`
- couleurs en **RGBW** `[R, G, B, W]` (W=0 pour RGB pur)
- effet Candle Multi (`fx: 102`) avec `sx` (vitesse) et `ix` (intensité)

### Import dans WLED (à faire sur chaque spot)

1. Aller dans **WLED → Config → Security & Updates**
2. Section **Backup & Restore**
3. Cliquer sur **Restore Presets** et sélectionner le fichier JSON

Alternativement, via l'API :
```bash
# Déployer sur les deux spots
curl -X POST "http://<IP_SPOT1>/presets.json" -d @presets_and_playlists.json
curl -X POST "http://<IP_SPOT2>/presets.json" -d @presets_and_playlists.json
```

---

# 🛁 Conclusion

Cette configuration transforme tes **deux spots GU10 WLED** en un **système lumineux d’ambiance haut de gamme**, parfait pour :

- la relaxation sous la douche
- les ambiances naturelles
- la chromathérapie légère
- les jeux de couleurs doux et évolutifs

Elle combine :
- réalisme
- minimalisme
- naturalité
- douceur
- automatisation totale

Bon bain lumineux 🌈🛁✨