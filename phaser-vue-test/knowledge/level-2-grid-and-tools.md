# Level 2: The Grid & The Tool

## Key Concepts Learned

### 1. State Management in Games

This level introduces the pattern:

```
onClick + Condition → Action
```

Instead of simple `onClick → Action`, we check the current tool **before** performing the action.

### 2. Nested Loops for Tile Grids

Create grids in Phaser using nested loops in `create()`:

```typescript
const mapSize = 5;
const tileSize = 64;

for (let y = 0; y < mapSize; y++) {
  for (let x = 0; x < mapSize; x++) {
    const pixelX = startX + x * tileSize;
    const pixelY = startY + y * tileSize;

    const tile = this.add.sprite(pixelX, pixelY, "grass");
    tile.setInteractive();
  }
}
```

### 3. Storing Tile State with `setData()`

Use Phaser's data system to store custom properties:

```typescript
// Set data
tile.setData("isWet", false);

// Get data
const isWet = tile.getData("isWet");
```

### 4. Tool Logic Pattern

```typescript
tile.on("pointerdown", () => {
  const tool = currentTool;
  const texture = tile.texture.key;

  // Check tool + current state
  if (tool === "hoe" && texture === "grass") {
    tile.setTexture("dirt");
  } else if (tool === "water" && texture === "dirt") {
    tile.setTint(0x666666);
    tile.setData("isWet", true);
  } else if (tool === "hand") {
    // Squish animation
  }
});
```

### 5. Visual Feedback with `setTint()`

Change sprite color without new assets:

```typescript
tile.setTint(0x666666); // Darken (wet effect)
tile.clearTint(); // Remove tint
```

### 6. Phaser Tweens for Animations

Add juice with simple animations:

```typescript
this.tweens.add({
  targets: tile,
  scaleX: 0.9,
  scaleY: 0.9,
  duration: 100,
  yoyo: true,
  ease: "Bounce.easeOut",
});
```

---

## Vue ↔ Phaser Communication

**Problem**: Vue reactive refs and Phaser don't always sync well.

**Solution**: Use a regular variable that Phaser reads:

```typescript
// Regular variable for Phaser to read
let currentTool: "hand" | "hoe" | "water" = "hand";

// Reactive ref for Vue template display
const activeTool = ref("hand");

// Update both when changing tools
function setTool(tool) {
  currentTool = tool; // For Phaser
  activeTool.value = tool; // For Vue template
}
```

---

## Files Created

| File                      | Purpose             |
| ------------------------- | ------------------- |
| `public/hand.png`         | Hand tool icon      |
| `public/hoe.png`          | Hoe tool icon       |
| `public/water.png`        | Water tool icon     |
| `src/components/Game.vue` | Main game component |

---

## Next Steps (Level 3 Preview)

- Add crops that can be planted on wet dirt
- Implement growth stages (timer-based)
- Connect to English learning: answer question → get seeds
