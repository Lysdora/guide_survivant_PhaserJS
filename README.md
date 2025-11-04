# 📚 COURS COMPLET : Créer un Roguelike avec Phaser.js

> **Guide ultra-pédagogique pour débutants**  
> Par une développeuse passionnée, pour les développeuses passionnées ! 💜  
> _Révision dans les transports - Édition 2025_

---

## 📖 Table des matières

1. [🎯 Introduction - Ce que tu vas apprendre](#introduction)
2. [📦 PARTIE 1 : Charger et afficher une tilemap](#partie1)
3. [🎭 PARTIE 2 : Créer un player provisoire](#partie2)
4. [💥 PARTIE 3 : Gérer les collisions](#partie3)
5. [🕹️ PARTIE 4 : Programmer les déplacements](#partie4)
6. [📸 PARTIE 5 : Configurer la caméra](#partie5)
7. [⚠️ PARTIE 6 : Pièges courants et solutions](#partie6)
8. [✅ Checklist complète](#checklist)
9. [🎓 Exercices de révision](#exercices)

---

## 🎯 Introduction - Ce que tu vas apprendre {#introduction}

### 🤔 Qu'est-ce qu'on va créer ?

Un **roguelike** en pixel art avec :
- 🗺️ Une carte créée dans Tiled
- 🎭 Un personnage contrôlable
- 🧱 Des murs qui bloquent le joueur
- 📸 Une caméra qui suit le joueur
- 🎮 Des contrôles fluides (ZQSD + flèches)

### 🛠️ Outils utilisés

- **Phaser.js** : Le moteur de jeu (version 3)
- **Vite** : L'outil de build (rapide et moderne)
- **Tiled** : L'éditeur de tilemap
- **VSCode** : L'éditeur de code

### 📁 Structure du projet

```
mon-roguelike/
├── public/
│   └── assets/
│       ├── maps/
│       │   └── donjon01.json    ← Ta map exportée de Tiled
│       └── tiles/
│           └── dungeon_tiles.png ← L'image du tileset
├── src/
│   └── game/
│       ├── main.js              ← Configuration Phaser
│       └── scenes/
│           ├── Preload.js       ← Chargement des assets
│           └── Game.js          ← Logique du jeu
└── package.json
```

**🔑 Règle d'or :**
- `public/` → Tous les fichiers non-code (images, sons, JSON)
- `src/` → Tout le code JavaScript

---

## 📦 PARTIE 1 : Charger et afficher une tilemap {#partie1}

### 🎓 Comprendre le cycle de chargement

**Phaser fonctionne en 3 étapes dans cet ordre :**

```
1. PRELOAD     2. CREATE      3. UPDATE
   ⬇️             ⬇️              ⬇️
Charger les   Initialiser    Boucle de jeu
assets        le jeu         (60 fois/sec)
```

**Analogie de la cuisine 🍳 :**
1. **Preload** = Sortir tous les ingrédients du frigo
2. **Create** = Préparer le plat (couper, mélanger)
3. **Update** = Surveiller la cuisson en continu

---

### 📥 Étape 1 : Charger les assets (Scène Preload)

**📍 Fichier : `src/game/scenes/Preload.js`**

```javascript
import { Scene } from "phaser";

export class Preload extends Scene {
  constructor() {
    super("Preload");
  }

  preload() {
    console.log("📦 Début du chargement des assets...");
    
    // 🎨 Charger l'IMAGE du tileset
    this.load.image("tiles", "assets/tiles/dungeon_tiles.png");
    //               ↑ Nom      ↑ Chemin depuis public/
    
    console.log("🎨 Chargement des tiles : assets/tiles/dungeon_tiles.png");
    
    // 🗺️ Charger le fichier JSON de la tilemap
    this.load.tilemapTiledJSON("map", "assets/maps/donjon01.json");
    //                          ↑ Nom  ↑ Chemin depuis public/
    
    console.log("🗺️ Chargement de la tilemap : assets/maps/donjon01.json");
  }

  create() {
    console.log("✅ Assets chargés avec succès !");
    console.log("🚀 Démarrage de la scène Game...");
    
    // Passer à la scène Game
    this.scene.start("Game");
  }
}
```

#### 🔍 Explication ligne par ligne

##### `this.load.image("tiles", "assets/tiles/dungeon_tiles.png");`

**À quoi ça sert ?**  
Charge une image et lui donne un nom pour l'utiliser plus tard.

**Les paramètres :**
- `"tiles"` → Le **nom** qu'on va utiliser dans le code (tu choisis ce que tu veux)
- `"assets/tiles/dungeon_tiles.png"` → Le **chemin** du fichier (relatif à `public/`)

**💡 Analogie :** C'est comme mettre une étiquette "Farine" sur un paquet dans ton placard !

##### `this.load.tilemapTiledJSON("map", "assets/maps/donjon01.json");`

**À quoi ça sert ?**  
Charge un fichier JSON de tilemap exporté depuis Tiled.

**Les paramètres :**
- `"map"` → Le **nom** pour référencer cette tilemap
- `"assets/maps/donjon01.json"` → Le **chemin** du fichier JSON

**⚠️ ATTENTION :**  
Le fichier JSON doit être exporté depuis Tiled au format **JSON** (pas TMX) !

##### `this.scene.start("Game");`

**À quoi ça sert ?**  
Lance la scène "Game" une fois que tout est chargé.

**Ordre d'exécution :**
```
preload() → create() → La scène Game démarre !
```

---

### 🏗️ Étape 2 : Créer la tilemap (Scène Game)

**📍 Fichier : `src/game/scenes/Game.js`**

```javascript
import { Scene } from "phaser";

export class Game extends Scene {
  constructor() {
    super("Game");
  }

  create() {
    console.log("📦 Création de la scène Game...");
    
    // === ÉTAPE 1 : Créer l'objet tilemap ===
    const map = this.make.tilemap({ key: "map" });
    //                                    ↑ Le nom qu'on a donné dans Preload
    console.log("🗺️ Tilemap créée :", map);
    
    // === ÉTAPE 2 : Associer l'image au tileset ===
    const tileset = map.addTilesetImage("donjon-level", "tiles");
    //                                   ↑ Nom dans JSON  ↑ Nom de l'image
    console.log("🎨 Tileset ajoutée :", tileset);
    
    // === ÉTAPE 3 : Créer les layers ===
    const groundLayer = map.createLayer("Ground", tileset, 0, 0);
    //                                   ↑ Nom du layer dans Tiled
    console.log("🌍 Layer Ground créé :", groundLayer);
    
    const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
    console.log("🧱 Layer Walls créé :", wallsLayer);
    
    console.log("✅ Tilemap affichée !");
  }
}
```

#### 🔍 Explication détaillée

##### `const map = this.make.tilemap({ key: "map" });`

**À quoi ça sert ?**  
Crée un objet Tilemap à partir du JSON chargé dans Preload.

**Que contient cet objet ?**
- La taille de la map (largeur, hauteur en tuiles)
- Les layers (sol, murs, etc.)
- Les propriétés de chaque tuile
- Les objets (spawn points, etc.)

**💡 Analogie :** C'est comme ouvrir un plan IKEA avant de monter le meuble !

##### `const tileset = map.addTilesetImage("donjon-level", "tiles");`

**À quoi ça sert ?**  
Associe l'image des tuiles au tileset défini dans le JSON.

**Les paramètres :**
- `"donjon-level"` → Le **nom du tileset** dans le fichier JSON Tiled
- `"tiles"` → Le **nom de l'image** chargée dans Preload

**🚨 PIÈGE ULTRA FRÉQUENT :**

Le nom `"donjon-level"` **DOIT** correspondre **EXACTEMENT** au nom dans ton JSON !

**Comment vérifier ?**
1. Ouvre ton fichier `donjon01.json`
2. Cherche la ligne `"name":"xxx"`
3. Utilise ce nom **exact** dans ton code !

**Exemple de JSON :**
```json
{
  "tilesets": [{
    "name": "donjon-level",  ← CE NOM ICI !
    "image": "dungeon_tiles.png"
  }]
}
```

**Dans le code :**
```javascript
const tileset = map.addTilesetImage("donjon-level", "tiles");
                                    //       ↑ Doit être identique !
```

**❌ Si les noms ne correspondent pas :**
```
Error: Tileset 'donjon-level' not found
```

##### `const groundLayer = map.createLayer("Ground", tileset, 0, 0);`

**À quoi ça sert ?**  
Crée et affiche un layer (une couche) de la tilemap.

**Les paramètres :**
- `"Ground"` → Le **nom du layer** dans Tiled (vérifie dans l'interface Tiled !)
- `tileset` → Le tileset créé juste avant
- `0, 0` → La position X, Y où afficher le layer (généralement 0, 0)

**🎨 Les layers, c'est quoi ?**

Imagine un sandwich 🥪 :
```
┌─────────────┐
│   DÉCOR     │ ← Layer 3 (optionnel)
├─────────────┤
│   MURS      │ ← Layer 2 (Walls)
├─────────────┤
│   SOL       │ ← Layer 1 (Ground)
└─────────────┘
```

Chaque layer est une couche transparente qu'on empile !

**💡 Pourquoi plusieurs layers ?**
- **Ground** : Le sol (herbe, dalles, etc.)
- **Walls** : Les murs et obstacles
- **Decor** : Les décorations par-dessus (optionnel)

**Ordre d'affichage :**
Le premier layer créé est affiché en dessous, le dernier au-dessus !

---

### 🎯 Récapitulatif PARTIE 1

**Ce qu'on a fait :**
1. ✅ Chargé l'image du tileset dans **Preload**
2. ✅ Chargé le JSON de la map dans **Preload**
3. ✅ Créé l'objet tilemap dans **Game**
4. ✅ Associé l'image au tileset
5. ✅ Créé et affiché les layers Ground et Walls

**Ce qu'il faut ABSOLUMENT retenir :**
- 🔑 Le nom du tileset dans le code **DOIT** correspondre au JSON
- 🔑 Le nom des layers dans le code **DOIT** correspondre à Tiled
- 🔑 Les chemins des fichiers sont relatifs à `public/`

---

## 🎭 PARTIE 2 : Créer un player provisoire {#partie2}

### 🎯 Objectif

Créer un sprite temporaire pour représenter le joueur avant d'avoir une vraie image.

### 🤔 Pourquoi un sprite provisoire ?

**Au début, tu n'as pas encore :**
- Un vrai sprite de personnage
- Des animations
- Une spritesheet

**Donc on crée un carré coloré temporaire !** 🟩

**💡 Avantage :** Tu peux tester la logique de jeu sans attendre les graphismes !

---

### 🎨 Créer une texture temporaire

**📍 Dans `Game.js`, fonction `preload()` :**

```javascript
preload() {
  // 🎨 Créer une texture carrée verte
  const graphics = this.add.graphics();
  //     ↑ Outil de dessin Phaser
  
  graphics.fillStyle(0x00ff00, 1);
  //                 ↑ Couleur  ↑ Opacité (1 = opaque)
  
  graphics.fillRect(0, 0, 16, 16);
  //                ↑X ↑Y ↑Largeur ↑Hauteur
  
  graphics.generateTexture("player", 16, 16);
  //                        ↑ Nom     ↑ Taille
  
  graphics.destroy();
  //       ↑ Important : libérer la mémoire !
}
```

#### 🔍 Explication ligne par ligne

##### `const graphics = this.add.graphics();`

**À quoi ça sert ?**  
Crée un objet "Graphics" qui permet de dessiner des formes (carrés, cercles, lignes, etc.).

**💡 Analogie :** C'est comme sortir un pinceau et une toile vierge !

##### `graphics.fillStyle(0x00ff00, 1);`

**À quoi ça sert ?**  
Définit la couleur et l'opacité du remplissage.

**Les paramètres :**
- `0x00ff00` → Couleur en **hexadécimal** (ici vert vif)
- `1` → Opacité (0 = transparent, 1 = opaque)

**🎨 Quelques couleurs courantes :**
```javascript
0xff0000  // Rouge
0x00ff00  // Vert
0x0000ff  // Bleu
0xffff00  // Jaune
0xff00ff  // Magenta
0x00ffff  // Cyan
0xffffff  // Blanc
0x000000  // Noir
```

##### `graphics.fillRect(0, 0, 16, 16);`

**À quoi ça sert ?**  
Dessine un rectangle rempli avec la couleur définie.

**Les paramètres :**
- `0, 0` → Position X, Y (coin haut-gauche)
- `16, 16` → Largeur et hauteur en pixels

**💡 Pourquoi 16x16 ?**  
C'est une taille standard pour les jeux en pixel art ! (Les tiles font souvent 16x16 ou 32x32)

**📐 Autres tailles courantes :**
- 8x8 → Très petit (style NES)
- 16x16 → Classique (style SNES)
- 32x32 → Plus grand (style GBA)

##### `graphics.generateTexture("player", 16, 16);`

**À quoi ça sert ?**  
Transforme le dessin en une **texture réutilisable** !

**Les paramètres :**
- `"player"` → Le **nom** de la texture (pour l'utiliser plus tard)
- `16, 16` → La taille de la texture

**💡 Analogie :** C'est comme prendre une photo de ton dessin pour pouvoir l'utiliser partout !

##### `graphics.destroy();`

**À quoi ça sert ?**  
Supprime l'objet Graphics de la mémoire.

**⚠️ IMPORTANT :**  
Une fois la texture générée, on n'a plus besoin de l'objet Graphics. **Toujours le détruire** pour éviter les fuites mémoire !

**❌ Si tu oublies :**  
Ton jeu prendra de plus en plus de RAM au fil du temps !

---

### 🏃 Créer le sprite du joueur

**📍 Dans `Game.js`, fonction `create()` :**

```javascript
create() {
  // ... (code de la tilemap)
  
  // 🏃 Créer le sprite du joueur
  this.player = this.physics.add.sprite(400, 300, "player");
  //    ↑ Variable  ↑ Avec physique  ↑ Position  ↑ Texture
  
  // 🚧 Empêcher de sortir des limites du monde
  this.player.setCollideWorldBounds(true);
}
```

#### 🔍 Explication détaillée

##### `this.player = this.physics.add.sprite(400, 300, "player");`

**À quoi ça sert ?**  
Crée un sprite avec **physique Arcade** activée.

**Les paramètres :**
- `400` → Position X (centre de l'écran si width = 800)
- `300` → Position Y (centre de l'écran si height = 600)
- `"player"` → Le **nom de la texture** créée dans `preload()`

**🤔 Différence entre `add.sprite` et `physics.add.sprite` ?**

| `add.sprite` | `physics.add.sprite` |
|--------------|----------------------|
| Sprite simple | Sprite + physique |
| Pas de vélocité | A une vélocité |
| Pas de collisions | Peut collisionner |
| Pour décors | Pour personnages/ennemis |

**💡 Règle simple :**
- Si ça **bouge** ou **collisionne** → `physics.add.sprite`
- Si c'est **statique** (décor) → `add.sprite`

##### `this.player.setCollideWorldBounds(true);`

**À quoi ça sert ?**  
Empêche le joueur de sortir des limites du monde de jeu.

**Sans cette ligne :**
```
🏃→→→→ [Écran] →→→→ 💨 (le joueur disparaît !)
```

**Avec cette ligne :**
```
🏃→→→ [Écran]🚧 (le joueur rebondit sur le bord !)
```

**🎯 Résultat :**  
Le joueur reste toujours visible à l'écran !

---

### 🎯 Récapitulatif PARTIE 2

**Ce qu'on a fait :**
1. ✅ Créé un objet Graphics pour dessiner
2. ✅ Dessiné un carré vert 16x16
3. ✅ Transformé le dessin en texture nommée "player"
4. ✅ Libéré la mémoire avec `destroy()`
5. ✅ Créé un sprite avec physique
6. ✅ Bloqué le joueur aux limites du monde

**Ce qu'il faut ABSOLUMENT retenir :**
- 🔑 Toujours `destroy()` un Graphics après `generateTexture()`
- 🔑 Utiliser `physics.add.sprite` pour les objets qui bougent
- 🔑 `setCollideWorldBounds(true)` pour éviter que le joueur disparaisse

---

## 💥 PARTIE 3 : Gérer les collisions {#partie3}

### 🎯 Objectif

Faire en sorte que le joueur ne puisse **pas traverser les murs** !

### 🤔 Comment ça marche ?

**Phaser a un système de collisions intégré !**

```
Joueur 🏃  →  [Mur 🧱]
              ↓
         COLLISION !
              ↓
         Joueur bloqué ❌
```

**Pour que ça marche, il faut 3 choses :**
1. ✅ Dire quelles tuiles sont solides
2. ✅ Activer la physique sur ces tuiles
3. ✅ Créer la collision entre le joueur et ces tuiles

---

### 🧱 Étape 1 : Marquer les tuiles solides dans Tiled

**🗺️ Dans Tiled (AVANT d'exporter le JSON) :**

1. Sélectionne une tuile de mur dans le tileset
2. Clique droit → "Tile Properties" (Propriétés de la tuile)
3. Ajoute une propriété personnalisée :
   - **Nom** : `collides`
   - **Type** : `bool` (booléen)
   - **Valeur** : `true` (coché ✅)
4. Répète pour **toutes les tuiles de murs**
5. Sauvegarde et **exporte en JSON** !

**💡 Astuce :**  
Tu peux sélectionner plusieurs tuiles à la fois (Ctrl + clic) et ajouter la propriété sur toutes d'un coup !

**📸 À quoi ça ressemble dans Tiled :**
```
Custom Properties
┌──────────────┐
│ collides  ✓  │  ← Coché
└──────────────┘
```

---

### ⚙️ Étape 2 : Activer les collisions dans Phaser

**📍 Dans `Game.js`, fonction `create()` :**

```javascript
create() {
  // ... (code de création de la tilemap)
  
  const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
  
  // 💥 Activer les collisions sur les tuiles avec la propriété "collides"
  wallsLayer.setCollisionByProperty({ collides: true });
  //          ↑ Méthode magique !    ↑ Nom de la propriété
  
  console.log("💥 Collisions activées sur le layer Walls !");
}
```

#### 🔍 Explication

##### `wallsLayer.setCollisionByProperty({ collides: true });`

**À quoi ça sert ?**  
Active la physique de collision sur **toutes les tuiles** qui ont la propriété `collides: true` dans Tiled.

**Comment ça marche ?**
1. Phaser lit le JSON de la tilemap
2. Il regarde chaque tuile du layer "Walls"
3. Si la tuile a `collides: true` → elle devient **solide** 🧱
4. Sinon → elle reste transparente aux collisions 👻

**💡 Analogie :**  
C'est comme mettre un mur invisible sur certaines tuiles !

**🎯 Autres méthodes de collision :**

```javascript
// Méthode 1 : Par propriété (RECOMMANDÉ !)
wallsLayer.setCollisionByProperty({ collides: true });

// Méthode 2 : Toutes les tuiles sauf les vides
wallsLayer.setCollisionByExclusion([-1]);

// Méthode 3 : Par numéros de tuiles spécifiques
wallsLayer.setCollision([5, 6, 7, 8]); // Tuiles 5,6,7,8 sont solides

// Méthode 4 : Par intervalle de tuiles
wallsLayer.setCollisionBetween(10, 20); // Tuiles 10 à 20 sont solides
```

**🏆 Pourquoi `setCollisionByProperty` est le meilleur ?**
- ✅ Flexible : tu changes dans Tiled, pas dans le code
- ✅ Clair : tu vois directement dans Tiled quelles tuiles collisionnent
- ✅ Scalable : tu peux ajouter d'autres propriétés (`damage`, `door`, etc.)

---

### 🤝 Étape 3 : Créer la collision Player ↔ Murs

**📍 Dans `Game.js`, fonction `create()` :**

```javascript
create() {
  // ... (code de création du player et du wallsLayer)
  
  // 🤝 Créer la collision entre le player et les murs
  this.physics.add.collider(this.player, wallsLayer);
  //                        ↑ Objet 1    ↑ Objet 2
  
  console.log("🤝 Collision player ↔ walls créée !");
}
```

#### 🔍 Explication

##### `this.physics.add.collider(this.player, wallsLayer);`

**À quoi ça sert ?**  
Dit à Phaser : "Ces deux objets ne doivent **PAS** se traverser !"

**Les paramètres :**
- `this.player` → Le sprite du joueur
- `wallsLayer` → Le layer des murs

**🎯 Résultat :**  
Phaser va automatiquement :
1. Détecter quand le joueur touche un mur
2. Empêcher le joueur de passer à travers
3. Faire "rebondir" le joueur légèrement

**💡 Analogie :**  
C'est comme mettre un garde du corps qui pousse le joueur quand il essaie de traverser un mur !

**🔄 Ordre d'exécution important :**

```javascript
// ❌ FAUX - Collision avant la création des objets
this.physics.add.collider(this.player, wallsLayer); // Erreur !
const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
this.player = this.physics.add.sprite(400, 300, "player");

// ✅ BON - Collision après la création des objets
const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
wallsLayer.setCollisionByProperty({ collides: true });
this.player = this.physics.add.sprite(400, 300, "player");
this.physics.add.collider(this.player, wallsLayer); // ✅
```

**🚨 ORDRE CRUCIAL :**
```
1. Créer le layer Walls
2. Activer les collisions sur le layer
3. Créer le player
4. Créer la collision player ↔ layer
```

---

### 🐛 Debug : Voir les zones de collision

**Pour VOIR les zones de collision (utile pour débugger) :**

```javascript
// 🐛 Ajouter après setCollisionByProperty
wallsLayer.renderDebug(this.add.graphics(), {
  tileColor: null,  // Pas de couleur sur toutes les tuiles
  collidingTileColor: new Phaser.Display.Color(243, 134, 48, 255), // Orange
  faceColor: new Phaser.Display.Color(40, 39, 37, 255) // Contour sombre
});
```

**🎯 Résultat :**  
Tu vois des rectangles oranges sur les tuiles qui collisionnent ! 🟧

**⚠️ À enlever en production :**  
Une fois que tout marche, commente ou supprime ce code !

---

### 🎯 Récapitulatif PARTIE 3

**Ce qu'on a fait :**
1. ✅ Ajouté la propriété `collides: true` dans Tiled
2. ✅ Activé les collisions avec `setCollisionByProperty()`
3. ✅ Créé la collision avec `physics.add.collider()`

**Ce qu'il faut ABSOLUMENT retenir :**
- 🔑 Toujours ajouter la propriété dans Tiled AVANT d'exporter
- 🔑 `setCollisionByProperty` AVANT `physics.add.collider`
- 🔑 L'ordre de création est crucial !

**🎉 Si tout marche :**  
Ton joueur rebondit maintenant sur les murs ! 🧱🏃

---

## 🕹️ PARTIE 4 : Programmer les déplacements {#partie4}

### 🎯 Objectif

Faire bouger le joueur avec le clavier (ZQSD + flèches) de manière fluide et réactive !

### 🤔 Comment ça marche ?

**Phaser fonctionne avec une boucle de jeu :**

```
update()  →  update()  →  update()  →  update()  →  ...
  ↓           ↓           ↓           ↓
60 fois par seconde ! (60 FPS)
```

**À chaque frame, on :**
1. Vérifie si une touche est enfoncée
2. Applique une vélocité (vitesse) au joueur
3. Phaser déplace automatiquement le joueur !

**💡 Analogie :**  
C'est comme pousser une voiture : tu donnes une force (vélocité), et elle avance !

---

### ⌨️ Étape 1 : Configurer les touches

**📍 Dans `Game.js`, fonction `create()` :**

```javascript
create() {
  // ... (code précédent)
  
  // ⌨️ Activer les touches fléchées par défaut
  this.cursors = this.input.keyboard.createCursorKeys();
  //    ↑ Variable    ↑ Méthode magique qui crée ⬆️⬇️⬅️➡️
  
  // 🎮 Ajouter des touches personnalisées (ZQSD)
  this.keys = this.input.keyboard.addKeys({
    up: "Z",      // ⬆️
    down: "S",    // ⬇️
    left: "Q",    // ⬅️
    right: "D",   // ➡️
  });
  
  // 🏃 Définir la vitesse de déplacement
  this.speed = 200; // pixels par seconde
  
  console.log("⌨️ Contrôles configurés : Flèches + ZQSD");
}
```

#### 🔍 Explication ligne par ligne

##### `this.cursors = this.input.keyboard.createCursorKeys();`

**À quoi ça sert ?**  
Crée automatiquement des références vers les touches fléchées.

**Ce que ça crée :**
```javascript
this.cursors = {
  up: Key,      // Flèche haut ⬆️
  down: Key,    // Flèche bas ⬇️
  left: Key,    // Flèche gauche ⬅️
  right: Key,   // Flèche droite ➡️
  space: Key,   // Barre espace
  shift: Key    // Shift
};
```

**💡 C'est un raccourci :**  
Au lieu de créer chaque touche manuellement, Phaser le fait pour toi !

##### `this.keys = this.input.keyboard.addKeys({ ... });`

**À quoi ça sert ?**  
Ajoute des touches personnalisées (ZQSD, WASD, etc.).

**Structure :**
```javascript
this.keys = this.input.keyboard.addKeys({
  nomDansLeCode: "TOUCHE_CLAVIER",
  //    ↑ Le nom que tu utilises  ↑ La touche physique
});
```

**🎮 Exemple avec WASD (version anglophone) :**
```javascript
this.wasd = this.input.keyboard.addKeys({
  up: "W",
  down: "S",
  left: "A",
  right: "D",
});
```

**🌍 Tu peux ajouter plein de touches :**
```javascript
this.keys = this.input.keyboard.addKeys({
  jump: "SPACE",
  attack: "X",
  dash: "SHIFT",
  inventory: "I",
});
```

**📋 Liste de touches courantes :**
```javascript
"A" à "Z"          // Lettres
"ZERO" à "NINE"    // Chiffres du clavier principal
"NUMPAD_ZERO"...   // Chiffres du pavé numérique
"SPACE"            // Barre espace
"SHIFT"            // Shift
"CTRL"             // Ctrl
"ALT"              // Alt
"ENTER"            // Entrée
"ESC"              // Échap
"UP", "DOWN"...    // Flèches (mais createCursorKeys() le fait déjà !)
```

##### `this.speed = 200;`

**À quoi ça sert ?**  
Définit la vitesse de déplacement en **pixels par seconde**.

**🎯 Différentes vitesses :**
```javascript
this.speed = 100;  // Lent (escargot 🐌)
this.speed = 200;  // Normal (marche 🚶)
this.speed = 400;  // Rapide (course 🏃)
this.speed = 800;  // Très rapide (dash ⚡)
```

**💡 Astuce :**  
Tu peux changer la vitesse dynamiquement !

```javascript
// Normal
this.speed = 200;

// Si Shift enfoncé → Course !
if (this.cursors.shift.isDown) {
  this.speed = 400;
} else {
  this.speed = 200;
}
```

---

### 🔄 Étape 2 : La boucle de déplacement (update)

**📍 Dans `Game.js`, fonction `update()` :**

```javascript
update() {
  // === ÉTAPE 1 : Arrêter le player par défaut ===
  this.player.setVelocity(0);
  //           ↑ Remet la vélocité à zéro
  
  // === ÉTAPE 2 : Mouvement HORIZONTAL ===
  if (this.cursors.left.isDown || this.keys.left.isDown) {
    // ⬅️ Aller à gauche
    this.player.setVelocityX(-this.speed);
    //                       ↑ Négatif = gauche
  } 
  else if (this.cursors.right.isDown || this.keys.right.isDown) {
    // ➡️ Aller à droite
    this.player.setVelocityX(this.speed);
    //                       ↑ Positif = droite
  }
  
  // === ÉTAPE 3 : Mouvement VERTICAL ===
  if (this.cursors.down.isDown || this.keys.down.isDown) {
    // ⬇️ Aller en bas
    this.player.setVelocityY(this.speed);
    //                       ↑ Positif = bas
  } 
  else if (this.cursors.up.isDown || this.keys.up.isDown) {
    // ⬆️ Aller en haut
    this.player.setVelocityY(-this.speed);
    //                       ↑ Négatif = haut
  }
  
  // === ÉTAPE 4 : Arrondir la position (anti-tremblement) ===
  this.player.x = Math.round(this.player.x);
  this.player.y = Math.round(this.player.y);
}
```

#### 🔍 Explication ligne par ligne

##### `this.player.setVelocity(0);`

**À quoi ça sert ?**  
Remet la vélocité à zéro **à chaque frame** !

**🤔 Pourquoi ?**  
Sinon, le joueur continue de glisser même quand on lâche les touches !

**Sans cette ligne :**
```
[Appuie sur →] → 🏃💨💨💨 (le joueur glisse à l'infini !)
```

**Avec cette ligne :**
```
[Appuie sur →] → 🏃
[Lâche →] → 🧍 (le joueur s'arrête immédiatement)
```

**💡 Analogie :**  
C'est comme remettre le frein à main à chaque instant !

##### `if (this.cursors.left.isDown || this.keys.left.isDown)`

**À quoi ça sert ?**  
Vérifie si la touche gauche (flèche OU Q) est **actuellement enfoncée**.

**Structure :**
```javascript
if (touche.isDown) {
  // Code exécuté tant que la touche est enfoncée
}
```

**🎯 Méthodes disponibles :**
```javascript
touche.isDown    // La touche est enfoncée maintenant
touche.isUp      // La touche est relâchée maintenant
touche.duration  // Depuis combien de temps elle est enfoncée (en ms)
```

**🔗 L'opérateur `||` (OU logique) :**

```javascript
if (condition1 || condition2) {
  // Exécuté si AU MOINS UNE condition est vraie
}
```

**Exemples :**
```javascript
true  || false  →  true   // Au moins une vraie
false || true   →  true   // Au moins une vraie
true  || true   →  true   // Les deux vraies
false || false  →  false  // Aucune vraie
```

**Dans notre cas :**
```javascript
if (this.cursors.left.isDown || this.keys.left.isDown) {
  // Si flèche gauche OU Q est enfoncée
}
```

**🎯 Résultat :**  
Les deux systèmes de contrôle marchent en même temps !

##### `this.player.setVelocityX(-this.speed);`

**À quoi ça sert ?**  
Applique une vélocité (vitesse) **horizontale** au joueur.

**Les paramètres :**
- **Négatif** (`-200`) → Aller à **gauche** ⬅️
- **Positif** (`200`) → Aller à **droite** ➡️
- **Zéro** (`0`) → Ne pas bouger horizontalement

**📐 Système de coordonnées Phaser :**
```
        0, 0
         ↓
    ⬅️ NEG | POS ➡️  (Axe X)
         |
        POS
         ⬇️
       (Axe Y)
```

**🎯 Résumé :**
```javascript
setVelocityX(-200)  // ⬅️ Gauche
setVelocityX(200)   // ➡️ Droite
setVelocityY(-200)  // ⬆️ Haut
setVelocityY(200)   // ⬇️ Bas
```

**⚠️ ATTENTION : Y est inversé !**

En maths, Y positif va **vers le haut** ⬆️  
En Phaser, Y positif va **vers le bas** ⬇️

C'est comme ça dans **tous** les moteurs de jeu 2D !

##### `Math.round(this.player.x);`

**À quoi ça sert ?**  
Arrondit la position du joueur à l'entier le plus proche.

**🤔 Pourquoi ?**  
Sans ça, le joueur peut avoir des positions décimales (ex: `100.2345678`), ce qui cause des tremblements visuels !

**Exemples :**
```javascript
Math.round(100.2)  →  100
Math.round(100.7)  →  101
Math.round(100.5)  →  101 (arrondi au supérieur si .5)
```

**🎯 Résultat :**  
Le sprite reste net et stable, sans micro-tremblements !

---

### 🎮 Variante : Mouvement style Zelda (une seule direction)

**Dans les Zelda, tu ne peux aller que dans UNE direction à la fois !**

**💡 Solution : Utiliser `else if` pour tout :**

```javascript
update() {
  this.player.setVelocity(0);
  
  // HORIZONTAL prioritaire
  if (this.cursors.left.isDown || this.keys.left.isDown) {
    this.player.setVelocityX(-this.speed);
  } 
  else if (this.cursors.right.isDown || this.keys.right.isDown) {
    this.player.setVelocityX(this.speed);
  }
  // SINON on vérifie le VERTICAL
  else if (this.cursors.up.isDown || this.keys.up.isDown) {
    this.player.setVelocityY(-this.speed);
  } 
  else if (this.cursors.down.isDown || this.keys.down.isDown) {
    this.player.setVelocityY(this.speed);
  }
}
```

**🔑 Le truc :** Tout est dans un seul `if/else if` !

**🎯 Résultat :**  
Pas de diagonales, comme dans les vieux Zelda ! ⚔️

---

### 🎯 Récapitulatif PARTIE 4

**Ce qu'on a fait :**
1. ✅ Configuré les touches (flèches + ZQSD)
2. ✅ Défini une vitesse de déplacement
3. ✅ Programmé la boucle de mouvement dans `update()`
4. ✅ Arrondi les positions pour éviter les tremblements

**Ce qu'il faut ABSOLUMENT retenir :**
- 🔑 `setVelocity(0)` au début de `update()` pour arrêter le glissement
- 🔑 `||` (OU) pour supporter plusieurs touches
- 🔑 Négatif = gauche/haut, Positif = droite/bas
- 🔑 `Math.round()` pour éviter les tremblements

**🎉 Si tout marche :**  
Ton joueur bouge maintenant avec fluidité ! 🏃✨

---

## 📸 PARTIE 5 : Configurer la caméra {#partie5}

### 🎯 Objectif

Faire en sorte que la caméra **suive le joueur** et **zoom** pour une meilleure visibilité !

### 🤔 Pourquoi configurer la caméra ?

**Par défaut :**
- La caméra est fixe (ne bouge pas)
- Le zoom est à 1x (trop petit pour du pixel art)
- Le joueur peut sortir de l'écran

**Avec la config :**
- La caméra suit le joueur partout 📸
- Le zoom x2 rend tout plus gros 🔍
- Le joueur reste toujours visible ✨

---

### 🔍 Configuration complète de la caméra

**📍 Dans `Game.js`, fonction `create()` (APRÈS la création du player) :**

```javascript
create() {
  // ... (code précédent)
  
  // === CAMÉRA : Configuration complète ===
  
  // 🔍 Zoom x2 (agrandit tout)
  this.cameras.main.setZoom(2);
  
  // 📸 La caméra suit le joueur
  this.cameras.main.startFollow(this.player);
  
  // 🎯 Zone morte (deadzone) pour un suivi plus fluide
  this.cameras.main.setDeadzone(50, 50);
  
  // 🌊 Lissage du mouvement (lerp)
  this.cameras.main.setLerp(0.1, 0.1);
  
  // 📐 Arrondir les pixels (anti-tremblement)
  this.cameras.main.roundPixels = true;
  
  console.log("📸 Caméra configurée : Zoom x2 + Suivi fluide");
}
```

#### 🔍 Explication ligne par ligne

##### `this.cameras.main.setZoom(2);`

**À quoi ça sert ?**  
Agrandit tout ce qui est affiché à l'écran.

**Les paramètres :**
- `1` → Taille normale (100%)
- `2` → Deux fois plus grand (200%)
- `0.5` → Deux fois plus petit (50%)

**🎨 Comparaison visuelle :**
```
Zoom 1x          Zoom 2x
┌────────┐       ┌────────┐
│ 🏃     │       │        │
│        │  →    │   🏃   │  (Plus gros !)
│    🧱  │       │        │
└────────┘       └────────┘
```

**💡 Pourquoi x2 pour le pixel art ?**  
Les sprites 16x16 sont trop petits sur les écrans modernes ! Le zoom x2 les rend bien visibles !

**🎯 Autres zooms possibles :**
```javascript
this.cameras.main.setZoom(1.5);  // x1.5
this.cameras.main.setZoom(3);    // x3 (très gros)
this.cameras.main.setZoom(4);    // x4 (énorme)
```

##### `this.cameras.main.startFollow(this.player);`

**À quoi ça sert ?**  
Fait suivre le joueur par la caméra automatiquement !

**Paramètres optionnels :**
```javascript
// Suivi simple
this.cameras.main.startFollow(this.player);

// Suivi avec options
this.cameras.main.startFollow(this.player, true, 0.1, 0.1);
//                                         ↑      ↑     ↑
//                                      round  lerpX lerpY
```

**Sans startFollow :**
```
🏃→→→ [Écran fixe] ... (le joueur sort de l'écran !)
```

**Avec startFollow :**
```
[Écran] → 🏃 ← (la caméra suit le joueur !)
```

##### `this.cameras.main.setDeadzone(50, 50);`

**À quoi ça sert ?**  
Crée une "zone morte" au centre où le joueur peut bouger sans que la caméra ne bouge.

**Les paramètres :**
- `50, 50` → Largeur et hauteur de la zone morte (en pixels)

**🎯 Comportement :**
```
Sans deadzone : La caméra bouge dès que le joueur bouge
                → Sensation de "caméra collée"

Avec deadzone : Le joueur peut bouger un peu avant que la caméra ne suive
                → Sensation plus fluide et naturelle
```

**📐 Visualisation :**
```
┌────────────────┐
│                │
│   ┌──────┐     │
│   │ 🏃  │     │ ← Zone morte (50x50)
│   └──────┘     │
│                │
└────────────────┘
  Écran caméra
```

**💡 Règle :**
- **Petite deadzone** (20-50) → Caméra réactive
- **Grande deadzone** (100-150) → Caméra plus stable

##### `this.cameras.main.setLerp(0.1, 0.1);`

**À quoi ça sert ?**  
Lisse le mouvement de la caméra (elle "rattrape" le joueur progressivement).

**Les paramètres :**
- `0.1, 0.1` → Vitesse de rattrapage (X et Y)

**🎯 Valeurs possibles :**
- `0` → La caméra ne bouge jamais
- `0.05` → Très lent (caméra élastique)
- `0.1` → Lent et fluide (RECOMMANDÉ)
- `0.5` → Moyen
- `1` → Instantané (pas de lissage)

**📊 Comportement :**
```
Lerp 1 (instantané) :
🏃 → [Caméra suit instantanément]

Lerp 0.1 (lissé) :
🏃 → ... → [Caméra rattrape doucement]
```

**💡 Analogie :**  
C'est comme une caméra tenue par un caméraman qui suit le sujet, au lieu d'être fixée rigidement !

##### `this.cameras.main.roundPixels = true;`

**À quoi ça sert ?**  
Arrondit les positions de la caméra pour éviter les pixels "flous" ou tremblants.

**🎨 Effet visuel :**
```
roundPixels = false :  roundPixels = true :
┌─────┐                ┌─────┐
│░░░█░│  (flou)        │░░██░│  (net !)
│░███░│                │░███░│
└─────┘                └─────┘
```

**⚠️ CRUCIAL pour le pixel art !**  
Sans ça, les pixels peuvent apparaître flous ou trembler !

---

### 🌍 Configuration globale (alternative)

**Au lieu de configurer dans `create()`, tu peux aussi le faire dans `main.js` :**

**📍 Dans `src/game/main.js` :**

```javascript
const config = {
  // ... autres options ...
  
  pixelArt: true,       // 🎯 Optimisé pour le pixel art
  roundPixels: true,    // 📐 Arrondir les pixels globalement
  antialias: false,     // 🚫 Pas d'antialiasing
  
  scale: {
    zoom: 2,            // 🔍 Zoom global x2
    mode: Phaser.Scale.FIT,
    autoCenter: Phaser.Scale.CENTER_BOTH,
  }
};
```

**🤔 Quelle méthode choisir ?**

| Dans `create()` | Dans `main.js` |
|-----------------|----------------|
| Contrôle précis | Plus simple |
| Par scène | Global au jeu |
| Peut changer dynamiquement | Fixe au démarrage |

**💡 Recommandation :**  
Utilise `main.js` pour la config de base, et `create()` pour les ajustements spécifiques !

---

### 🎯 Récapitulatif PARTIE 5

**Ce qu'on a fait :**
1. ✅ Activé le zoom x2
2. ✅ Fait suivre le joueur par la caméra
3. ✅ Ajouté une deadzone pour plus de fluidité
4. ✅ Lissé le mouvement avec lerp
5. ✅ Arrondi les pixels pour éviter les tremblements

**Ce qu'il faut ABSOLUMENT retenir :**
- 🔑 `setZoom(2)` pour agrandir (essentiel en pixel art)
- 🔑 `startFollow()` pour que la caméra suive le joueur
- 🔑 `roundPixels = true` pour éviter les pixels flous
- 🔑 `setLerp()` pour un mouvement fluide et naturel

**🎉 Si tout marche :**  
La caméra suit maintenant le joueur de manière fluide et agréable ! 📸✨

---

## ⚠️ PARTIE 6 : Pièges courants et solutions {#partie6}

### 🐛 Problème 1 : La tilemap ne s'affiche pas

#### ❌ Symptômes
- Écran noir
- Aucune erreur dans la console
- Ou erreur : `Tileset 'xxx' not found`

#### ✅ Solutions

**Solution A : Vérifier le nom du tileset**

1. Ouvre ton fichier `donjon01.json`
2. Cherche la ligne avec `"name"`
3. Note le nom **exact** (avec majuscules !)

**Exemple dans le JSON :**
```json
{
  "tilesets": [{
    "name": "donjon-level",  ← CE NOM !
    ...
  }]
}
```

**Dans le code :**
```javascript
const tileset = map.addTilesetImage("donjon-level", "tiles");
                                    //       ↑ Doit correspondre !
```

**Solution B : Vérifier le chemin du fichier**

```javascript
// ❌ Faux
this.load.tilemapTiledJSON("map", "maps/donjon01.json");

// ✅ Bon (si dans public/assets/)
this.load.tilemapTiledJSON("map", "assets/maps/donjon01.json");
```

**Solution C : Vérifier le nom des layers**

Dans Tiled, tes layers s'appellent `"Ground"` et `"Walls"` ?

```javascript
// Dans le code, utilise les MÊMES noms !
const groundLayer = map.createLayer("Ground", tileset, 0, 0);
const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
```

---

### 🐛 Problème 2 : Le player traverse les murs

#### ❌ Symptômes
- Le joueur passe à travers les murs comme un fantôme 👻
- Aucune erreur dans la console

#### ✅ Solutions

**Solution A : Vérifier la propriété dans Tiled**

1. Ouvre ta map dans Tiled
2. Sélectionne une tuile de mur
3. Vérifie qu'elle a la propriété `collides: true` (cochée ✓)

**Solution B : Vérifier l'ordre du code**

```javascript
// ✅ BON ORDRE
const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
wallsLayer.setCollisionByProperty({ collides: true });  // D'ABORD
this.player = this.physics.add.sprite(400, 300, "player");
this.physics.add.collider(this.player, wallsLayer);     // ENSUITE

// ❌ MAUVAIS ORDRE
this.physics.add.collider(this.player, wallsLayer);     // Trop tôt !
wallsLayer.setCollisionByProperty({ collides: true });  // Trop tard !
```

**Solution C : Vérifier que le player a la physique**

```javascript
// ❌ Faux (pas de physique)
this.player = this.add.sprite(400, 300, "player");

// ✅ Bon (avec physique)
this.player = this.physics.add.sprite(400, 300, "player");
```

---

### 🐛 Problème 3 : La caméra tremble

#### ❌ Symptômes
- Le personnage ou la map "vibre" légèrement
- Les pixels semblent flous ou décalés

#### ✅ Solutions

**Solution A : Activer roundPixels**

```javascript
// Dans create()
this.cameras.main.roundPixels = true;

// ET dans main.js
const config = {
  pixelArt: true,
  roundPixels: true,
  // ...
};
```

**Solution B : Arrondir les positions dans update()**

```javascript
update() {
  // ... mouvement ...
  
  // Arrondir la position du player
  this.player.x = Math.round(this.player.x);
  this.player.y = Math.round(this.player.y);
}
```

**Solution C : Désactiver l'antialiasing**

```javascript
const config = {
  antialias: false,  // Important pour le pixel art !
  // ...
};
```

---

### 🐛 Problème 4 : Les assets ne chargent pas

#### ❌ Symptômes
- Erreur `Failed to load image`
- Erreur `Failed to load tilemapTiledJSON`

#### ✅ Solutions

**Solution A : Vérifier la structure des dossiers**

```
public/
├── assets/        ← ATTENTION : Tu as ce dossier ?
│   ├── maps/
│   │   └── donjon01.json
│   └── tiles/
│       └── dungeon_tiles.png
```

**Solution B : Vérifier les chemins**

```javascript
// Si les fichiers sont dans public/assets/
this.load.image("tiles", "assets/tiles/dungeon_tiles.png");
this.load.tilemapTiledJSON("map", "assets/maps/donjon01.json");

// Si les fichiers sont directement dans public/
this.load.image("tiles", "tiles/dungeon_tiles.png");
this.load.tilemapTiledJSON("map", "maps/donjon01.json");
```

**Solution C : Vider le cache**

Appuie sur `Ctrl + Shift + R` dans le navigateur pour forcer le rechargement !

---

### 🐛 Problème 5 : Le player ne bouge pas

#### ❌ Symptômes
- Les touches ne font rien
- Le player reste immobile

#### ✅ Solutions

**Solution A : Vérifier que les touches sont configurées**

```javascript
// Dans create()
this.cursors = this.input.keyboard.createCursorKeys();
this.keys = this.input.keyboard.addKeys({ ... });
```

**Solution B : Vérifier la fonction update()**

```javascript
// La fonction update() existe bien ?
update() {
  // ... code de mouvement ...
}
```

**Solution C : Vérifier les collisions**

Le player est peut-être coincé dans un mur ! Change sa position de départ :

```javascript
// Essaie une autre position
this.player = this.physics.add.sprite(200, 200, "player");
```

---

### 🐛 Problème 6 : Erreur "Cannot read property 'xxx' of undefined"

#### ❌ Symptômes
- Erreur dans la console
- Le jeu crash

#### ✅ Solutions

**Solution : Vérifier l'ordre de création**

```javascript
// ❌ Faux - utilisation avant création
this.physics.add.collider(this.player, wallsLayer);  // player n'existe pas encore !
this.player = this.physics.add.sprite(400, 300, "player");

// ✅ Bon - création avant utilisation
this.player = this.physics.add.sprite(400, 300, "player");
this.physics.add.collider(this.player, wallsLayer);  // ✓
```

---

### 🎯 Récapitulatif PARTIE 6

**Les erreurs les plus fréquentes :**
1. 🔴 Nom du tileset incorrect
2. 🔴 Nom des layers incorrect
3. 🔴 Mauvais ordre de création
4. 🔴 Oubli de `setCollisionByProperty()`
5. 🔴 Chemins de fichiers incorrects

**Les solutions les plus efficaces :**
1. ✅ Toujours vérifier les noms dans le JSON
2. ✅ Respecter l'ordre de création
3. ✅ Utiliser `console.log()` pour débugger
4. ✅ Activer `debug: true` dans la config physique

---

## ✅ Checklist complète {#checklist}

### 📦 Avant de commencer

- [ ] Node.js et npm installés
- [ ] Projet Vite + Phaser créé
- [ ] Tiled installé
- [ ] VSCode (ou autre éditeur) prêt

---

### 🗺️ Création de la tilemap dans Tiled

- [ ] Tileset importé (image PNG)
- [ ] Map créée avec bonne taille de tuiles
- [ ] Layer "Ground" créé et peint
- [ ] Layer "Walls" créé et peint
- [ ] Propriété `collides: true` ajoutée sur les murs
- [ ] Map exportée en JSON dans `public/assets/maps/`

---

### 📁 Structure des fichiers

- [ ] `public/assets/maps/donjon01.json` existe
- [ ] `public/assets/tiles/dungeon_tiles.png` existe
- [ ] `src/game/scenes/Preload.js` existe
- [ ] `src/game/scenes/Game.js` existe
- [ ] `src/game/main.js` configuré

---

### 📥 Scène Preload

- [ ] `this.load.image("tiles", "assets/tiles/xxx.png")` ✓
- [ ] `this.load.tilemapTiledJSON("map", "assets/maps/xxx.json")` ✓
- [ ] `this.scene.start("Game")` dans create() ✓
- [ ] Console logs ajoutés pour debug ✓

---

### 🎮 Scène Game - Partie Tilemap

- [ ] `const map = this.make.tilemap({ key: "map" })` ✓
- [ ] `const tileset = map.addTilesetImage("nom-exact", "tiles")` ✓
- [ ] `const groundLayer = map.createLayer("Ground", tileset, 0, 0)` ✓
- [ ] `const wallsLayer = map.createLayer("Walls", tileset, 0, 0)` ✓
- [ ] Nom du tileset correspond au JSON ✓
- [ ] Noms des layers correspondent à Tiled ✓

---

### 🎭 Scène Game - Partie Player

- [ ] Texture temporaire créée dans preload() ✓
- [ ] `graphics.destroy()` appelé après generateTexture() ✓
- [ ] `this.player = this.physics.add.sprite(x, y, "player")` ✓
- [ ] `this.player.setCollideWorldBounds(true)` ✓

---

### 💥 Scène Game - Partie Collisions

- [ ] `wallsLayer.setCollisionByProperty({ collides: true })` ✓
- [ ] `this.physics.add.collider(this.player, wallsLayer)` ✓
- [ ] Ordre correct : layer → collision → player → collider ✓

---

### 🕹️ Scène Game - Partie Contrôles

- [ ] `this.cursors = this.input.keyboard.createCursorKeys()` ✓
- [ ] `this.keys = this.input.keyboard.addKeys({ ... })` ✓
- [ ] `this.speed = 200` défini ✓
- [ ] Fonction `update()` existe ✓
- [ ] `this.player.setVelocity(0)` au début de update() ✓
- [ ] Logique de mouvement H + V implémentée ✓
- [ ] Positions arrondies avec `Math.round()` ✓

---

### 📸 Scène Game - Partie Caméra

- [ ] `this.cameras.main.setZoom(2)` ✓
- [ ] `this.cameras.main.startFollow(this.player)` ✓
- [ ] `this.cameras.main.setDeadzone(50, 50)` ✓
- [ ] `this.cameras.main.setLerp(0.1, 0.1)` ✓
- [ ] `this.cameras.main.roundPixels = true` ✓

---

### ⚙️ Configuration globale (main.js)

- [ ] `pixelArt: true` ✓
- [ ] `roundPixels: true` ✓
- [ ] `antialias: false` ✓
- [ ] `scene: [Preload, Game]` (dans le bon ordre !) ✓
- [ ] Physique Arcade activée ✓
- [ ] `debug: true` (pour tester) ✓

---

### 🧪 Tests

- [ ] Le serveur démarre sans erreur (`npm run dev`) ✓
- [ ] La console n'affiche aucune erreur rouge ✓
- [ ] La tilemap s'affiche correctement ✓
- [ ] Le player apparaît ✓
- [ ] Le player bouge avec les touches ✓
- [ ] Le player ne traverse pas les murs ✓
- [ ] La caméra suit le player ✓
- [ ] Pas de tremblements visuels ✓

---

### 🎉 Bravo !

Si tu as coché toutes les cases, **ton roguelike fonctionne !** 🏆

---

## 🎓 Exercices de révision {#exercices}

### 📝 Exercice 1 : Questions théoriques

**Réponds sans regarder le cours :**

1. Quelle est la différence entre `add.sprite` et `physics.add.sprite` ?
2. Dans quel ordre s'exécutent `preload()`, `create()` et `update()` ?
3. Que fait `setCollisionByProperty({ collides: true })` ?
4. Pourquoi utilise-t-on `Math.round()` sur les positions ?
5. À quoi sert `setLerp(0.1, 0.1)` sur la caméra ?

**Réponses :**

1. `add.sprite` = sprite simple, `physics.add.sprite` = sprite avec physique (vélocité, collisions)
2. `preload()` → `create()` → `update()` (60 fois/sec)
3. Active les collisions sur les tuiles qui ont la propriété `collides: true` dans Tiled
4. Pour éviter les positions décimales qui causent des tremblements visuels
5. Lisse le mouvement de la caméra pour un suivi plus fluide

---

### 🛠️ Exercice 2 : Correction de code

**Trouve les erreurs dans ce code :**

```javascript
create() {
  // Création du player
  this.player = this.add.sprite(400, 300, "player");
  
  // Création de la tilemap
  const map = this.make.tilemap({ key: "map" });
  const tileset = map.addTilesetImage("tiles", "tiles");
  const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
  
  // Collisions
  this.physics.add.collider(this.player, wallsLayer);
  wallsLayer.setCollisionByProperty({ collides: true });
}
```

**Erreurs :**
1. ❌ `this.add.sprite` → devrait être `this.physics.add.sprite`
2. ❌ `addTilesetImage("tiles", "tiles")` → le premier paramètre doit correspondre au nom dans le JSON
3. ❌ Ordre incorrect : `setCollisionByProperty` doit être AVANT `physics.add.collider`

**Code corrigé :**
```javascript
create() {
  // Création de la tilemap
  const map = this.make.tilemap({ key: "map" });
  const tileset = map.addTilesetImage("donjon-level", "tiles");
  const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
  
  // Collisions d'ABORD
  wallsLayer.setCollisionByProperty({ collides: true });
  
  // Player ENSUITE
  this.player = this.physics.add.sprite(400, 300, "player");
  
  // Collider en DERNIER
  this.physics.add.collider(this.player, wallsLayer);
}
```

---

### 💻 Exercice 3 : Ajouter une fonctionnalité

**Objectif : Ajouter une touche SHIFT pour courir**

**Solution :**

```javascript
update() {
  // Vitesse normale ou course
  const speed = this.cursors.shift.isDown ? 400 : 200;
  //            ↑ Si SHIFT enfoncé     ↑ Course ↑ Marche
  
  this.player.setVelocity(0);
  
  if (this.cursors.left.isDown) {
    this.player.setVelocityX(-speed);  // Utilise la vitesse variable
  } 
  // ... reste du code
}
```

**💡 Explication :**
- `condition ? valeurSiVrai : valeurSiFaux` = opérateur ternaire
- Si SHIFT enfoncé → `speed = 400`
- Sinon → `speed = 200`

---

### 🎨 Exercice 4 : Changer la couleur du player

**Objectif : Créer un player rouge au lieu de vert**

**Solution :**

```javascript
preload() {
  const graphics = this.add.graphics();
  graphics.fillStyle(0xff0000, 1);  // Rouge au lieu de 0x00ff00
  graphics.fillRect(0, 0, 16, 16);
  graphics.generateTexture("player", 16, 16);
  graphics.destroy();
}
```

**🎨 Autres couleurs :**
```javascript
0xff0000  // 🔴 Rouge
0x00ff00  // 🟢 Vert
0x0000ff  // 🔵 Bleu
0xffff00  // 🟡 Jaune
0xff00ff  // 🟣 Magenta
```

---

### 🚀 Exercice 5 : Challenge final

**Objectif : Ajouter un système de téléportation**

Quand le joueur appuie sur **ESPACE**, il se téléporte à une position aléatoire !

**Solution :**

```javascript
update() {
  // ... code de mouvement normal ...
  
  // Téléportation avec ESPACE
  if (Phaser.Input.Keyboard.JustDown(this.cursors.space)) {
    // Position aléatoire
    const randomX = Phaser.Math.Between(100, 700);
    const randomY = Phaser.Math.Between(100, 500);
    
    // Téléporter le player
    this.player.setPosition(randomX, randomY);
    
    console.log(`🌀 Téléportation ! Nouvelle position : ${randomX}, ${randomY}`);
  }
}
```

**💡 Explications :**
- `JustDown()` détecte l'appui (pas maintenu)
- `Phaser.Math.Between(min, max)` génère un nombre aléatoire
- `setPosition(x, y)` téléporte instantanément

---

## 🎉 Conclusion

### 🏆 Ce que tu as appris

**Félicitations !** Tu sais maintenant :

- ✅ Charger et afficher une tilemap Tiled dans Phaser
- ✅ Créer un sprite avec physique
- ✅ Gérer les collisions entre le joueur et les murs
- ✅ Programmer des déplacements fluides (ZQSD + flèches)
- ✅ Configurer une caméra qui suit le joueur
- ✅ Débugger les problèmes courants

**Tu as les bases solides pour créer un roguelike complet !** 🎮✨

---

### 🚀 Pour aller plus loin

**Prochaines étapes (par ordre de difficulté) :**

1. 🎨 **Remplacer le carré par un vrai sprite**
   - Trouver ou créer un sprite de personnage
   - Charger une spritesheet
   - Ajouter des animations (marche dans les 4 directions)

2. 👾 **Ajouter des ennemis**
   - Créer un sprite ennemi
   - Programmer une IA simple (patrouille, poursuite)
   - Gérer les collisions player ↔ ennemi

3. ⚔️ **Système de combat**
   - Ajouter des points de vie (HP)
   - Créer un système d'attaque
   - Afficher une barre de vie

4. 🎒 **Inventaire et objets**
   - Créer des objets à ramasser (potions, clés)
   - Programmer un système d'inventaire
   - Créer une UI pour afficher l'inventaire

5. 🚪 **Changement de niveau**
   - Créer plusieurs maps dans Tiled
   - Ajouter des portes/escaliers
   - Charger dynamiquement la nouvelle map

6. 💾 **Sauvegarde du jeu**
   - Utiliser `localStorage` pour sauvegarder
   - Sauvegarder position, HP, inventaire
   - Charger la sauvegarde au démarrage

7. 🎵 **Sons et musiques**
   - Charger des fichiers audio
   - Jouer des bruitages (pas, attaque, objet)
   - Ajouter une musique de fond

---

### 📚 Ressources utiles

**Documentation officielle :**
- 📖 [Phaser 3 Docs](https://photonstorm.github.io/phaser3-docs/)
- 🎓 [Phaser 3 Examples](https://labs.phaser.io/)
- 🗺️ [Tiled Documentation](https://doc.mapeditor.org/)

**Tutoriels vidéo :**
- 🎥 [Chaîne YouTube Phaser officielle](https://www.youtube.com/@PhaserGameDev)
- 🎥 [Ourcade (excellents tutos Phaser)](https://www.youtube.com/@ourcadehq)

**Ressources graphiques gratuites :**
- 🎨 [OpenGameArt](https://opengameart.org/)
- 🎨 [Itch.io Game Assets](https://itch.io/game-assets/free)
- 🎨 [Kenney (assets gratuits)](https://www.kenney.nl/)

**Communautés :**
- 💬 [Phaser Discord](https://discord.gg/phaser)
- 💬 [HTML5 Game Devs Forum](https://www.html5gamedevs.com/)

---

### 💪 Message de motivation

**Tu as franchi une étape importante !** 🎉

Créer un jeu, c'est comme construire une maison :
- 🏗️ Tu as posé les **fondations** (la tilemap)
- 🧱 Tu as monté les **murs** (les collisions)
- 🚪 Tu as installé la **porte** (le player)
- 📸 Tu as mis les **fenêtres** (la caméra)

**Maintenant, tu peux décorer, meubler, personnaliser !** 🎨

**N'aie pas peur d'expérimenter !** Casse des trucs, teste, recommence. C'est comme ça qu'on apprend ! 💡

**Chaque bug résolu est une victoire.** 🏆  
**Chaque feature ajoutée est un niveau gagné.** 🎮  
**Chaque ligne de code écrite te rapproche de ton jeu de rêve.** ✨

---

### 🙏 Derniers conseils

1. **Code régulièrement** (même 15 min/jour)
2. **Sauvegarde souvent** (Git, cloud, USB)
3. **Commente ton code** (ton futur toi te remerciera)
4. **Teste après chaque ajout** (pas tout d'un coup)
5. **Demande de l'aide** (communautés, forums, Discord)
6. **Amuse-toi !** (c'est le plus important)

---

**🎮 Bon dev et à bientôt dans les donjons ! ⚔️✨**

---

_📅 Cours créé le 4 novembre 2025_  
_💜 Avec amour et pédagogie_  
_🚇 Pour réviser dans les transports !_

**Bon courage pour ton taf ! Tu gères ! 💪🔥**
