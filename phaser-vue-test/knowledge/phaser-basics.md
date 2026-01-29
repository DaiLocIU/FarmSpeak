# Phaser 3 Knowledge Notes

## 1. Game Configuration

The game config is the foundation of your Phaser game:

```typescript
const config: Phaser.Types.Core.GameConfig = {
  type: Phaser.AUTO, // AUTO = WebGL with Canvas fallback
  width: 800, // Game canvas width
  height: 600, // Game canvas height
  parent: containerElement, // DOM element to mount the game
  backgroundColor: "#2d2d2d",
  pixelArt: true, // CRITICAL for pixel art games!
  scene: {
    preload,
    create,
    update,
  },
};
```

---

## 2. Pixel Art Rendering (IMPORTANT!)

### The Problem

By default, Phaser uses **bilinear filtering** when scaling images, which causes pixel art to look **blurry**.

### The Solution

Add `pixelArt: true` to your game config:

```typescript
pixelArt: true; // Uses nearest-neighbor interpolation
```

This preserves sharp pixel edges when scaling sprites.

---

## 3. Scene Lifecycle

### `preload()`

- Runs first, before anything is displayed
- Use to load all assets (images, audio, spritesheets)

```typescript
function preload(this: Phaser.Scene) {
  this.load.image("key", "/path/to/image.png");
}
```

### `create()`

- Runs once after preload completes
- Setup your game objects, sprites, interactions

```typescript
function create(this: Phaser.Scene) {
  const sprite = this.add.sprite(x, y, "key");
}
```

### `update()`

- Runs every frame (60fps typically)
- Game loop logic goes here

---

## 4. Sprites & Interactivity

### Creating a Sprite

```typescript
const tile = this.add.sprite(400, 300, "textureKey");
```

### Scaling

```typescript
tile.setScale(2); // 2x bigger
tile.setScale(0.5); // Half size
```

### Making Sprites Clickable

```typescript
tile.setInteractive(); // Enable input detection

tile.on("pointerdown", () => {
  // Handle click
});
```

### Changing Texture Dynamically

```typescript
tile.setTexture("newTextureKey");

// Check current texture
if (tile.texture.key === "dirt") {
  tile.setTexture("grass");
}
```

---

## 5. Vue Integration

### Setup Pattern (Vue 3.5+)

```vue
<script setup lang="ts">
import { onMounted, onUnmounted, useTemplateRef } from "vue";
import Phaser from "phaser";

// useTemplateRef is the preferred way for DOM element refs in Vue 3.5+
const gameContainer = useTemplateRef<HTMLDivElement>("gameContainer");
let gameInstance: Phaser.Game | null = null;

onMounted(() => {
  if (!gameContainer.value) return;

  const config = {
    /* ... */
  };
  gameInstance = new Phaser.Game(config);
});

onUnmounted(() => {
  if (gameInstance) {
    gameInstance.destroy(true); // Clean up!
  }
});
</script>

<template>
  <div ref="gameContainer"></div>
</template>
```

### `ref()` vs `useTemplateRef()`

| API                | Use Case                                  |
| ------------------ | ----------------------------------------- |
| `ref()`            | Reactive data (numbers, strings, objects) |
| `useTemplateRef()` | DOM element references (Vue 3.5+)         |

### Key Points:

- Use `useTemplateRef` for DOM element references (Vue 3.5+)
- Create game in `onMounted`
- **Always destroy game in `onUnmounted`** to prevent memory leaks

---

## 6. Asset Loading

### Image Paths

- Put assets in `/public` folder
- Reference with root path: `/image.png`

```typescript
this.load.image("dirt", "/dirt.png");
```

### Supported Formats

- Images: PNG, JPG, WebP
- Audio: MP3, OGG, WAV
- Spritesheets: PNG with JSON atlas

---

## 7. Common Gotchas

| Issue               | Solution                                   |
| ------------------- | ------------------------------------------ |
| Blurry pixel art    | Add `pixelArt: true` to config             |
| Game not showing    | Check `parent` element exists              |
| Assets not loading  | Check paths (use `/` for public folder)    |
| Memory leaks in Vue | Call `game.destroy(true)` in `onUnmounted` |
| Click not working   | Call `sprite.setInteractive()` first       |

---

## 8. Useful Resources

- [Phaser 3 Documentation](https://photonstorm.github.io/phaser3-docs/)
- [Phaser Examples](https://phaser.io/examples)
- [Phaser Discord](https://discord.gg/phaser)
