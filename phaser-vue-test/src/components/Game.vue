<script setup lang="ts">
import { onMounted, onUnmounted, useTemplateRef } from "vue";
import Phaser from "phaser";

// --- VUE STATE ---
const gameContainer = useTemplateRef<HTMLDivElement>("gameContainer");
let gameInstance: Phaser.Game | null = null;

// Tool State - using a simple variable that Phaser can read
let currentTool: "hand" | "hoe" | "water" = "hand";

// Function to update tool (called from template)
function setTool(tool: "hand" | "hoe" | "water") {
  currentTool = tool;
  activeTool.value = tool;

  // Update cursor sprite texture
  if (cursorSprite) {
    cursorSprite.setTexture(`cursor-${tool}`);
  }

  // Hide system cursor over canvas
  if (gameContainer.value) {
    gameContainer.value.style.cursor = "none";
    const canvas = gameContainer.value.querySelector("canvas");
    if (canvas) {
      (canvas as HTMLCanvasElement).style.cursor = "none";
    }
  }
}

// Reactive version for template display
import { ref } from "vue";
const activeTool = ref<"hand" | "hoe" | "water">("hand");

// --- PHASER ENGINE ---

// Store scene reference for cursor updates
let gameScene: Phaser.Scene | null = null;
let cursorSprite: Phaser.GameObjects.Sprite | null = null;

function preload(this: Phaser.Scene) {
  this.load.image("grass", "/grass.png");
  this.load.image("dirt", "/dirt.png");
  // Load cursor images
  this.load.image("cursor-hand", "/cursor-hand.png");
  this.load.image("cursor-hoe", "/cursor-hoe.png");
  this.load.image("cursor-water", "/cursor-water.png");
}

function create(this: Phaser.Scene) {
  const mapSize = 5;
  const tileSize = 64; // Size each tile should appear as
  const gap = 4; // Gap between tiles for visibility
  const cellSize = tileSize + gap;

  // Calculate grid dimensions to center it
  const gridWidth = mapSize * cellSize - gap;
  const gridHeight = mapSize * cellSize - gap;
  const startX = (800 - gridWidth) / 2 + tileSize / 2;
  const startY = (600 - gridHeight) / 2 + tileSize / 2;

  // Draw debug background grid
  const graphics = this.add.graphics();

  // Create the 5x5 Grid
  for (let row = 0; row < mapSize; row++) {
    for (let col = 0; col < mapSize; col++) {
      const pixelX = startX + col * cellSize;
      const pixelY = startY + row * cellSize;

      // Draw debug border for this cell
      graphics.lineStyle(2, 0xffff00, 0.5); // Yellow border
      graphics.strokeRect(
        pixelX - tileSize / 2,
        pixelY - tileSize / 2,
        tileSize,
        tileSize,
      );

      // Create tile (Default is Grass)
      const tile = this.add.sprite(pixelX, pixelY, "grass");

      // Scale tile to fit the cell (assuming source image is larger)
      // If your image is 512x512, scale = 64/512 = 0.125
      const originalSize = tile.width; // Get actual image size
      const scale = tileSize / originalSize;
      tile.setScale(scale);

      tile.setInteractive();

      // Store tile position for debugging
      tile.setData("isWet", false);
      tile.setData("gridPos", `[${col},${row}]`);

      // Add coordinate label for debugging
      this.add
        .text(pixelX, pixelY - 20, `${col},${row}`, {
          fontSize: "10px",
          color: "#888888",
        })
        .setOrigin(0.5);

      // INTERACTION LOGIC
      tile.on("pointerdown", () => {
        const tool = currentTool;
        const texture = tile.texture.key;
        const isWet = tile.getData("isWet");
        const gridPos = tile.getData("gridPos");

        console.log(
          `[${tool.toUpperCase()}] clicked tile ${gridPos} - ${texture}${isWet ? " (wet)" : ""}`,
        );

        // LOGIC A: Hoe - converts grass to dirt
        if (tool === "hoe" && texture === "grass") {
          tile.setTexture("dirt");
          tile.clearTint();
          tile.setData("isWet", false);

          // Dig animation
          this.tweens.add({
            targets: tile,
            angle: { from: -5, to: 5 },
            duration: 50,
            yoyo: true,
            repeat: 2,
            onComplete: () => {
              tile.angle = 0;
            },
          });
        }

        // LOGIC B: Water - wets dirt tiles
        else if (tool === "water" && texture === "dirt" && !isWet) {
          tile.setTint(0x666666); // Darken to show wetness
          tile.setData("isWet", true);

          // Water splash effect
          this.tweens.add({
            targets: tile,
            alpha: { from: 0.7, to: 1 },
            duration: 200,
          });
        }

        // LOGIC C: Hand - touch/inspect
        else if (tool === "hand") {
          // Squish animation
          this.tweens.add({
            targets: tile,
            scaleX: 0.9,
            scaleY: 0.9,
            duration: 100,
            yoyo: true,
            ease: "Bounce.easeOut",
          });
        }
      });

      // Hover effect
      tile.on("pointerover", () => {
        tile.setAlpha(0.8);
      });

      tile.on("pointerout", () => {
        tile.setAlpha(1);
      });
    }
  }

  // Add instruction text
  this.add
    .text(400, 550, "Select a tool and click the tiles!", {
      fontSize: "18px",
      color: "#aaaaaa",
    })
    .setOrigin(0.5);

  // Create cursor sprite that follows mouse
  gameScene = this;
  cursorSprite = this.add.sprite(0, 0, "cursor-hand");
  cursorSprite.setDepth(1000); // Always on top
  cursorSprite.setScale(1); // 32x32 already
  cursorSprite.setOrigin(0.5, 0.5);

  // Hide cursor when outside game
  this.input.on("pointerout", () => {
    if (cursorSprite) cursorSprite.setVisible(false);
  });

  this.input.on("pointerover", () => {
    if (cursorSprite) cursorSprite.setVisible(true);
  });
}

function update(this: Phaser.Scene) {
  // Make cursor sprite follow the mouse
  if (cursorSprite && this.input.activePointer) {
    cursorSprite.setPosition(
      this.input.activePointer.x,
      this.input.activePointer.y,
    );
  }
}

// --- INIT & DESTROY ---
onMounted(() => {
  if (!gameContainer.value) return;

  const config: Phaser.Types.Core.GameConfig = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    parent: gameContainer.value,
    backgroundColor: "#1a1a2e",
    pixelArt: true,
    scene: { preload, create, update },
  };

  gameInstance = new Phaser.Game(config);
});

onUnmounted(() => {
  if (gameInstance) gameInstance.destroy(true);
});
</script>

<template>
  <div class="game-wrapper">
    <!-- Tool Selection UI -->
    <div class="ui-panel">
      <h3>
        Tool: <span class="tool-name">{{ activeTool }}</span>
      </h3>

      <div class="button-group">
        <button
          @click="setTool('hand')"
          :class="{ active: activeTool === 'hand' }"
          title="Touch/Inspect tiles"
        >
          <img src="/hand.png" alt="Hand" class="tool-icon" />
          <span>Hand</span>
        </button>

        <button
          @click="setTool('hoe')"
          :class="{ active: activeTool === 'hoe' }"
          title="Convert grass to dirt"
        >
          <img src="/hoe.png" alt="Hoe" class="tool-icon" />
          <span>Hoe</span>
        </button>

        <button
          @click="setTool('water')"
          :class="{ active: activeTool === 'water' }"
          title="Water dirt tiles"
        >
          <img src="/water.png" alt="Water" class="tool-icon" />
          <span>Water</span>
        </button>
      </div>
    </div>

    <div ref="gameContainer"></div>
  </div>
</template>

<style scoped>
.game-wrapper {
  position: relative;
  width: 800px;
  margin: 20px auto;
}

.ui-panel {
  position: absolute;
  top: 10px;
  left: 10px;
  background: linear-gradient(
    135deg,
    rgba(0, 0, 0, 0.85),
    rgba(20, 20, 40, 0.9)
  );
  padding: 15px 20px;
  border-radius: 12px;
  color: white;
  z-index: 10;
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
}

.ui-panel h3 {
  margin: 0 0 12px 0;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: #888;
}

.tool-name {
  color: #4caf50;
  text-transform: capitalize;
}

.button-group {
  display: flex;
  gap: 8px;
}

button {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 10px 14px;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.05);
  color: #aaa;
  border: 2px solid transparent;
  border-radius: 8px;
  transition: all 0.2s ease;
  font-size: 12px;
}

button:hover {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  transform: translateY(-2px);
}

button.active {
  background: rgba(76, 175, 80, 0.2);
  border-color: #4caf50;
  color: #4caf50;
  box-shadow: 0 0 15px rgba(76, 175, 80, 0.3);
}

.tool-icon {
  width: 32px;
  height: 32px;
  image-rendering: pixelated;
}
</style>
