<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import Phaser from "phaser";

const gameContainer = ref<HTMLDivElement | null>(null);
let gameInstance: Phaser.Game | null = null;

// 1. PRELOAD: Load our two images into memory
function preload(this: Phaser.Scene) {
  this.load.image("dirt", "/dirt.png"); // Key: 'dirt'
  this.load.image("grass", "/grass.png"); // Key: 'grass'
}

// 2. CREATE: Setup the Magic Tile
function create(this: Phaser.Scene) {
  // Center of the screen (400, 300)
  // We start with 'dirt'
  const tile = this.add.sprite(400, 300, "dirt");

  // Make it visible (scale down since image is large)
  tile.setScale(0.5);

  // CRITICAL: Tell Phaser "This object can be clicked"
  tile.setInteractive();

  // The Interaction Logic
  tile.on("pointerdown", () => {
    // Check current texture key
    if (tile.texture.key === "dirt") {
      // Logic: If dirt, change to grass
      tile.setTexture("grass");
      console.log("Changed to Grass!");
    } else {
      // Logic: If grass, change back to dirt
      tile.setTexture("dirt");
      console.log("Changed to Dirt!");
    }
  });

  // Just some text to help you
  this.add.text(300, 500, "Click the ball!", {
    fontSize: "32px",
    color: "#fff",
  });
}

function update() {
  // Nothing needed here yet
}

// Vue Lifecycle
onMounted(() => {
  if (!gameContainer.value) return;

  const config: Phaser.Types.Core.GameConfig = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    parent: gameContainer.value,
    backgroundColor: "#2d2d2d", // Dark grey background
    pixelArt: true, // Disable anti-aliasing for crisp pixel art
    scene: {
      preload,
      create,
      update,
    },
  };

  gameInstance = new Phaser.Game(config);
});

onUnmounted(() => {
  if (gameInstance) {
    gameInstance.destroy(true);
  }
});
</script>

<template>
  <div ref="gameContainer"></div>
</template>
