
# Ambiances WLED pour Salle de Bain — Spots GU10 RGB

Ce dépôt contient :
- un fichier `presets_and_playlists.json`
- une configuration complète de **presets**, **playlists**, **palettes personnalisées**
- une logique de **flicker intelligent basé sur la symbolique de chaque couleur**

Il est optimisé pour un **éclairage d’ambiance dans une salle de bain**, composé de :
- **2 spots encastrés GU10 RGB**
- chacun **pré‑flashé avec WLED**
- les deux contrôlés via **une instance WLED principale**, grâce à **DDP**
- 2 segments (un par spot), permettant à chaque spot de prendre une couleur différente

L’objectif :  
Créer une ambiance **relaxante, immersive, chaleureuse**, avec :
- des **transitions lentes (7s)**
- des **changements de couleur espacés (30 à 45s)**
- un **flicker subtil**, adapté à la *matière* que la couleur évoque (feu, eau, terre…)

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

Chaque ambiance a sa propre playlist :
- durée par preset : **30–45s**
- transitions : **7s**
- shuffle activé → rend chaque spot **indépendant et organique**

---

# 📦 Fichier JSON

Le fichier `presets_and_playlists.json` contient :
- toutes les ambiances
- toutes les playlists
- les flickers adaptés
- les transitions lentes
- la configuration complète des couleurs

Il est directement importable dans :

> WLED → Presets → "Edit JSON"

et

> WLED → Playlists → "Edit JSON"

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