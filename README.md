# 🎮 Cours Phaser.js - Création d'un Roguelike

> **Guide complet pour créer un jeu roguelike avec Phaser.js**  
> Basé sur le projet mon-roguelike d'apprentissage

---

## 📚 Table des matières

1. [🗺️ Instanciation et gestion des Maps](#maps)
2. [🎭 Création d'un Player provisoire](#player)
3. [💥 Gestion des collisions](#collisions)
4. [🕹️ Gestion des déplacements](#deplacements)
5. [📸 Configuration de la caméra](#camera)
6. [⚠️ Points d'attention importants](#attention)

---

## 🗺️ Instanciation et gestion des Maps {#maps}

### 📦 1. Chargement des assets (Scène Preload)

```javascript
preload() {
    console.log("📦 Début du chargement des assets...");
    
    // 🎨 Chargement de la spritesheet des tiles
    this.load.image("tiles", "assets/tiles/dungeon_tiles.png");
    console.log("🎨 Chargement des tiles : assets/tiles/dungeon_tiles.png");

    // 🗺️ Chargement du fichier JSON de la tilemap
    this.load.tilemapTiledJSON("map", "assets/maps/donjon01.json");
    console.log("🗺️ Chargement de la tilemap : assets/maps/donjon01.json");
}
```

**🔑 Points clés :**
- Le nom `"tiles"` sera utilisé plus tard pour référencer l'image
- Le nom `"map"` sera utilisé pour référencer la tilemap JSON
- Les chemins sont relatifs au dossier `public/`

### 🏗️ 2. Création de la tilemap (Scène Game)

```javascript
create() {
    // 🗺️ Créer l'objet tilemap à partir du JSON chargé
    const map = this.make.tilemap({ key: "map" });
    console.log("🗺️ Tilemap créée :", map);
    
    // 🎨 Associer l'image des tiles au tileset
    // ATTENTION : "donjon-level" doit correspondre au nom dans le JSON !
    const tileset = map.addTilesetImage("donjon-level", "tiles");
    console.log("🎨 Tileset ajoutée :", tileset);
    
    // 🌍 Créer le layer du sol
    const groundLayer = map.createLayer("Ground", tileset, 0, 0);
    console.log("🌍 Layer Ground créé :", groundLayer);
    
    // 🧱 Créer le layer des murs
    const wallsLayer = map.createLayer("Walls", tileset, 0, 0);
    console.log("🧱 Layer Walls créé :", wallsLayer);
}
```

**⚠️ PIÈGE FRÉQUENT :**
- Le nom dans `addTilesetImage("donjon-level", "tiles")` DOIT correspondre exactement au nom du tileset dans votre fichier JSON Tiled !
- Vérifiez dans votre JSON : `"name":"donjon-level"`

### 📁 3. Structure des fichiers

```
public/
├── assets/
│   ├── maps/
│   │   └── donjon01.json    ← Fichier JSON exporté de Tiled
│   └── tiles/
│       └── dungeon_tiles.png ← Image des tiles
```

---

## 🎭 Création d'un Player provisoire {#player}

### 🎨 1. Création d'une texture temporaire

```javascript
preload() {
    // 🎭 Créer une texture pour le player uniquement
    const graphics = this.add.graphics();
    graphics.fillStyle(0x00ff00, 1);  // Vert vif
    graphics.fillRect(0, 0, 16, 16);  // Carré 16x16 pixels
    graphics.generateTexture("player", 16, 16);
    graphics.destroy();  // ⚠️ Important : nettoyer l'objet graphics
}
```

**🔍 Explication :**
- `0x00ff00` = couleur verte en hexadécimal
- `16, 16` = taille du sprite en pixels
- `"player"` = nom de la texture pour l'utiliser plus tard
- `destroy()` libère la mémoire

### 🏃 2. Instanciation du player

```javascript
create() {
    // 🏃 Créer le sprite du joueur avec physique
    this.player = this.physics.add.sprite(400, 300, "player");
    
    // 🚧 Empêcher le player de sortir des limites du monde
    this.player.setCollideWorldBounds(true);
}
```

**📍 Paramètres :**
- `400, 300` = position X, Y de départ
- `"player"` = nom de la texture créée précédemment

---

## 💥 Gestion des collisions {#collisions}

### 🧱 1. Activation des collisions sur les tiles

```javascript
create() {
    // 💥 Activer les collisions sur les tuiles avec la propriété "collides"
    wallsLayer.setCollisionByProperty({ collides: true });
}
```

**🔧 Comment ça marche :**
- Dans Tiled, vous devez ajouter une propriété `collides: true` sur les tiles qui bloquent
- Phaser activera automatiquement les collisions sur ces tiles

### 🤝 2. Collision player ↔ murs

```javascript
create() {
    // 🤝 Créer la collision entre le player et les murs
    this.physics.add.collider(this.player, wallsLayer);
}
```

**🎯 Résultat :**
- Le player ne peut plus traverser les murs
- Phaser gère automatiquement la physique de collision

### 🏗️ 3. Configuration de Tiled pour les collisions

Dans **Tiled Editor** :
1. 🎯 Sélectionner une tile de mur
2. ➕ Ajouter une propriété personnalisée
3. 📝 Nom : `collides`, Type : `bool`, Valeur : `true`
4. 💾 Sauvegarder et exporter en JSON

---

## 🕹️ Gestion des déplacements {#deplacements}

### ⌨️ 1. Configuration des touches

```javascript
create() {
    // ⌨️ Touches fléchées par défaut
    this.cursors = this.input.keyboard.createCursorKeys();

    // 🎮 Touches personnalisées (ZQSD)
    this.keys = this.input.keyboard.addKeys({
        up: "Z",
        down: "S", 
        left: "Q",
        right: "D",
    });
    
    // 🏃 Vitesse de déplacement
    this.speed = 200; // pixels par seconde
}
```

### 🔄 2. Boucle de déplacement (update)

```javascript
update() {
    // 🛑 Arrêter le player par défaut
    this.player.setVelocity(0);
    
    // ⬅️➡️ Mouvement horizontal
    if (this.cursors.left.isDown || this.keys.left.isDown) {
        this.player.setVelocityX(-this.speed);
    } else if (this.cursors.right.isDown || this.keys.right.isDown) {
        this.player.setVelocityX(this.speed);
    }
    
    // ⬆️⬇️ Mouvement vertical
    if (this.cursors.down.isDown || this.keys.down.isDown) {
        this.player.setVelocityY(this.speed);
    } else if (this.cursors.up.isDown || this.keys.up.isDown) {
        this.player.setVelocityY(-this.speed);
    }
    
    // 📐 Arrondir la position pour éviter les tremblements
    this.player.x = Math.round(this.player.x);
    this.player.y = Math.round(this.player.y);
}
```

**🧠 Logique :**
1. **Réinitialiser** la vélocité à 0
2. **Vérifier** les touches enfoncées
3. **Appliquer** la vélocité correspondante
4. **Arrondir** les positions pour la stabilité

---

## 📸 Configuration de la caméra {#camera}

### 🔍 1. Zoom et suivi

```javascript
create() {
    // 🔍 Configuration de la caméra avec zoom
    this.cameras.main.setZoom(2); // Zoom x2
    this.cameras.main.startFollow(this.player); // La caméra suit le joueur
    
    // 🎯 Configuration pour éviter les tremblements
    this.cameras.main.setDeadzone(50, 50); // Zone morte pour un suivi plus fluide
    this.cameras.main.setLerp(0.1, 0.1); // Lissage du mouvement de la caméra
    this.cameras.main.roundPixels = true; // Arrondir les pixels
}
```

**🎛️ Paramètres expliqués :**
- **Zoom** : `2` = affichage 2x plus grand
- **Deadzone** : `50, 50` = zone de 50px où la caméra ne bouge pas
- **Lerp** : `0.1` = vitesse de rattrapage de la caméra (0 = immédiat, 1 = très lent)

### 🌍 2. Configuration globale

```javascript
// Dans main.js
const config = {
    pixelArt: true,        // 🎯 Optimisé pour le pixel art
    roundPixels: true,     // 📐 Arrondir les pixels globalement
    antialias: false,      // 🚫 Pas d'antialiasing pour le pixel art
    scale: {
        zoom: 2,           // 🔍 Zoom global x2
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH,
    }
};
```

---

## ⚠️ Points d'attention importants {#attention}

### 🔴 1. Erreurs fréquentes

| ❌ Erreur | ✅ Solution |
|-----------|-------------|
| Tilemap ne s'affiche pas | Vérifier le nom du tileset dans JSON vs code |
| Player traverse les murs | Ajouter `setCollisionByProperty({ collides: true })` |
| Caméra tremble | Ajouter `roundPixels: true` et `setLerp()` |
| Assets ne chargent pas | Vérifier les chemins relatifs à `public/` |

### 🎯 2. Ordre d'exécution crucial

```javascript
// 📝 ORDRE IMPORTANT dans create() :
1. Créer la tilemap
2. Ajouter le tileset  
3. Créer les layers
4. Configurer les collisions sur les layers
5. Créer le player
6. Configurer les collisions player ↔ layers
7. Configurer la caméra
```

### 🧪 3. Debug et tests

```javascript
// 🔍 Ajouter des logs pour débugger
console.log("🗺️ Tilemap créée :", map);
console.log("🎨 Tileset ajoutée :", tileset);
console.log("🧱 Layer Walls créé :", wallsLayer);

// 📊 Activer le debug physique
physics: {
    default: "arcade",
    arcade: {
        debug: true, // 👈 Affiche les hitboxes
    },
}
```

### 💡 4. Optimisations

```javascript
// 🚀 Optimisations pour de meilleures performances
- Utiliser `pixelArt: true` pour les jeux en pixel art
- Arrondir les positions avec `Math.round()`
- Limiter les appels dans update() aux actions nécessaires
- Précharger tous les assets dans Preload
```

---

## 🎓 Récapitulatif

### 📋 Checklist pour créer un roguelike

- [ ] 📦 **Preload** : Charger tilemap JSON + image tiles
- [ ] 🗺️ **Tilemap** : Créer tilemap et tileset avec bon nom
- [ ] 🏗️ **Layers** : Créer layers Ground et Walls
- [ ] 🎭 **Player** : Créer texture temporaire + sprite
- [ ] 💥 **Collisions** : Activer collisions tiles + player↔walls
- [ ] 🕹️ **Déplacements** : Configurer touches + boucle update
- [ ] 📸 **Caméra** : Zoom + suivi + anti-tremblement

### 🏆 Résultat final

Un roguelike fonctionnel avec :
- ✅ Map affichée avec collisions
- ✅ Player controllable (ZQSD + flèches)
- ✅ Caméra qui suit le joueur
- ✅ Zoom x2 pour une meilleure visibilité
- ✅ Physique réaliste

---

## 🚀 Pour aller plus loin

**Prochaines étapes possibles :**
- 👾 Ajouter des ennemis
- 🎒 Système d'inventaire
- 🗝️ Objets à ramasser
- 🚪 Changement de niveau
- 🎵 Sons et musiques
- 💾 Sauvegarde du jeu

---

*📅 Cours créé le 4 novembre 2025*  
*🎮 Basé sur le projet mon-roguelike avec Phaser.js*

**Bon voyage dans les transports ! 🚇✨**
