---
layout: post
title: GameLevelSeek CS 111 Breakdown
description: In-depth, non-table breakdown of how GameLevelSeek demonstrates CS 111 and CSSE concepts with a runnable GameRunner.
permalink: /seek-cs111-breakdown
hide: true
toc: true
toc_history: true
codemirror: true
---

**Designated GameRunner**

{% capture seek_runner_challenge %}
Run the Seek level here while reading the breakdown below. Move with WASD, press Q to open the sprite menu, choose a sprite, and collect every coin.
{% endcapture %}

{% capture seek_runner_code %}
import GameControl from '{{site.baseurl}}/assets/js/GameEnginev1/essentials/GameControl.js';
import GameObject from '{{site.baseurl}}/assets/js/GameEnginev1/essentials/GameObject.js';
import GameEnvBackground from '{{site.baseurl}}/assets/js/GameEnginev1/essentials/GameEnvBackground.js';
import Player from '{{site.baseurl}}/assets/js/GameEnginev1/essentials/Player.js';

function makeSpriteSheet(primary, accent) {
  const sheet = document.createElement('canvas');
  sheet.width = 144;
  sheet.height = 192;
  const ctx = sheet.getContext('2d');

  for (let row = 0; row < 4; row += 1) {
    for (let col = 0; col < 3; col += 1) {
      const x = col * 48;
      const y = row * 48;
      ctx.fillStyle = primary;
      ctx.fillRect(x + 12, y + 10, 24, 30);
      ctx.fillStyle = accent;
      ctx.fillRect(x + 15 + col, y + 4, 18, 12);
      ctx.fillStyle = '#111827';
      ctx.fillRect(x + 18, y + 18, 4, 4);
      ctx.fillRect(x + 27, y + 18, 4, 4);
      ctx.fillStyle = '#f8fafc';
      ctx.fillRect(x + 14, y + 38, 8, 6);
      ctx.fillRect(x + 27, y + 38, 8, 6);
    }
  }

  return sheet.toDataURL('image/png');
}

class SeekCoin extends GameObject {
  constructor(data, gameEnv) {
    super(gameEnv);
    this.position = data.position;
    this.radius = 16;
    this.collected = false;
    this.canvas = document.createElement('canvas');
    this.canvas.id = 'SeekCoin-' + Math.round(this.position.x) + '-' + Math.round(this.position.y);
    this.canvas.width = 36;
    this.canvas.height = 36;
    this.ctx = this.canvas.getContext('2d');
    gameEnv.container.appendChild(this.canvas);
  }

  update() {
    this.draw();
  }

  draw() {
    this.ctx.clearRect(0, 0, 36, 36);
    if (!this.collected) {
      this.canvas.style.display = 'block';
      this.ctx.fillStyle = '#f59e0b';
      this.ctx.beginPath();
      this.ctx.arc(18, 18, 15, 0, Math.PI * 2);
      this.ctx.fill();
      this.ctx.strokeStyle = '#fef3c7';
      this.ctx.lineWidth = 3;
      this.ctx.stroke();
    } else {
      this.canvas.style.display = 'none';
    }

    this.canvas.style.position = 'absolute';
    this.canvas.style.left = this.position.x + 'px';
    this.canvas.style.top = this.position.y + 'px';
    this.canvas.style.zIndex = '8';
  }

  resize() {}

  destroy() {
    this.canvas.remove();
  }
}

class GameLevelSeek {
  constructor(gameEnv) {
    this.gameEnv = gameEnv;
    const path = gameEnv.path;
    this.collected = 0;
    this.spriteOptions = [
      { label: 'Green Seeker', src: makeSpriteSheet('#22c55e', '#bbf7d0') },
      { label: 'Blue Seeker', src: makeSpriteSheet('#2563eb', '#bfdbfe') },
      { label: 'Rose Seeker', src: makeSpriteSheet('#e11d48', '#fecdd3') }
    ];
    this.playerData = {
      id: 'SeekPlayer',
      src: this.spriteOptions[0].src,
      SCALE_FACTOR: 10,
      STEP_FACTOR: 700,
      ANIMATION_RATE: 8,
      INIT_POSITION: { x: 80, y: 160 },
      pixels: { width: 144, height: 192 },
      orientation: { rows: 4, columns: 3 },
      down: { row: 0, start: 0, columns: 3 },
      right: { row: 1, start: 0, columns: 3 },
      left: { row: 2, start: 0, columns: 3 },
      up: { row: 3, start: 0, columns: 3 },
      hitbox: { widthPercentage: 0.25, heightPercentage: 0.15 },
      keypress: { up: 87, left: 65, down: 83, right: 68 }
    };
    const coinPositions = [
      { x: gameEnv.innerWidth * 0.28, y: gameEnv.innerHeight * 0.24 },
      { x: gameEnv.innerWidth * 0.55, y: gameEnv.innerHeight * 0.38 },
      { x: gameEnv.innerWidth * 0.78, y: gameEnv.innerHeight * 0.25 },
      { x: gameEnv.innerWidth * 0.43, y: gameEnv.innerHeight * 0.68 }
    ];
    const bgData = {
      name: 'seek-background',
      src: path + '/images/gamebuilder/bg/clouds.jpg',
      pixels: { height: 720, width: 1280 }
    };
    this.classes = [
      { class: GameEnvBackground, data: bgData },
      { class: Player, data: this.playerData },
      ...coinPositions.map((position) => ({ class: SeekCoin, data: { position } }))
    ];
  }

  initialize() {
    this.buildHud();
    this.onKeyDown = (event) => {
      if (event.key.toLowerCase() === 'q') this.toggleMenu();
    };
    document.addEventListener('keydown', this.onKeyDown);
    console.log('GameLevelSeek.js loaded:', new Date().toISOString());
  }

  buildHud() {
    this.hud = document.createElement('div');
    this.hud.style.cssText = 'position:absolute;left:12px;top:12px;z-index:40;padding:10px 12px;background:#111827;color:#f8fafc;border:1px solid #38bdf8;border-radius:6px;font:14px system-ui;';
    this.menu = document.createElement('div');
    this.menu.style.cssText = 'position:absolute;right:12px;top:12px;z-index:40;display:none;gap:8px;flex-direction:column;padding:10px;background:#0f172a;border:1px solid #f59e0b;border-radius:6px;';
    this.spriteOptions.forEach((option, index) => {
      const button = document.createElement('button');
      button.textContent = option.label;
      button.style.cssText = 'padding:7px 10px;border:0;border-radius:4px;background:#f8fafc;color:#0f172a;cursor:pointer;';
      button.addEventListener('click', () => this.switchSprite(index));
      this.menu.appendChild(button);
    });
    this.gameEnv.container.append(this.hud, this.menu);
    this.syncHud();
  }

  syncHud() {
    const total = (this.gameEnv?.gameObjects || []).filter((obj) => obj instanceof SeekCoin).length;
    this.hud.textContent = 'Seek coins: ' + this.collected + '/' + total + ' | Q: sprites | WASD: move';
  }

  toggleMenu() {
    this.menu.style.display = this.menu.style.display === 'none' ? 'flex' : 'none';
  }

  switchSprite(index) {
    const player = (this.gameEnv?.gameObjects || []).find((obj) => obj?.spriteData?.id === 'SeekPlayer');
    if (!player) return;
    const option = this.spriteOptions[index];
    player.spriteData.src = option.src;
    player.spriteSheet = new Image();
    player.spriteReady = false;
    player.spriteSheet.onload = () => {
      player.spriteReady = true;
      player.resize();
    };
    player.spriteSheet.src = option.src;
    console.log('Sprite switched:', option.label);
  }

  update() {
    const objects = this.gameEnv?.gameObjects || [];
    const player = objects.find((obj) => obj?.spriteData?.id === 'SeekPlayer');
    const coins = objects.filter((obj) => obj instanceof SeekCoin);
    for (const coin of coins) {
      if (!coin.collected && player && Math.hypot(player.position.x - coin.position.x, player.position.y - coin.position.y) < 55) {
        coin.collected = true;
        this.collected += 1;
        this.syncHud();
      }
    }
    if (coins.length && this.collected === coins.length) {
      this.hud.textContent = 'All Seek coins collected. Press Q to keep testing sprites.';
    }
  }

  destroy() {
    document.removeEventListener('keydown', this.onKeyDown);
    this.hud?.remove();
    this.menu?.remove();
  }
}

export const gameLevelClasses = [GameLevelSeek];
export { GameControl };
{% endcapture %}

{% include runners/game.html
   runner_id="seek-main-runner"
   challenge=seek_runner_challenge
   code=seek_runner_code
   height="520px"
   editor_height="360px"
%}

## Overview

`GameLevelSeek` is a strong CS 111 capstone example because it combines object-oriented design, arrays, loops, keyboard input, canvas rendering, collision-style distance checks, DOM output, and debugging-friendly structure in one playable level.

The core gameplay loop is simple:

- The player moves around the map with WASD.
- Coins are generated from an array of positions.
- The player collects each coin by getting close enough.
- Pressing Q opens a sprite menu.
- Clicking a menu button changes the active player sprite.

That simple loop gives one GameRunner a lot of real CS concepts to explain.

## Software Engineering Practices

The level is organized into focused classes and methods instead of one giant function.

```js
class GameLevelSeek {
  constructor(gameEnv) {
    this.gameEnv = gameEnv;
    this.collected = 0;
    this.spriteOptions = [ ... ];
  }

  initialize() { ... }
  update() { ... }
  destroy() { ... }
}
```

<a id="writing-classes"></a>
### Writing Classes

- `class GameLevelSeek` controls the full playable level.
- `class SeekCoin extends GameObject` creates a custom collectible.
- A class keeps related state and behavior together.
- The constructor runs when the engine creates the level or coin object.
- `GameLevelSeek` stores setup data such as the selected sprite, coin count, and level object configuration.

<a id="methods-and-parameters"></a>
### Methods and Parameters

```js
switchSprite(index) { ... }
makeSpriteSheet(primary, accent) { ... }
constructor(data, gameEnv) { ... }
```

- Methods are functions attached to a class or object.
- Parameters are values passed into those methods.
- `index` chooses which sprite option should be used.
- `primary` and `accent` control generated sprite colors.
- `data` and `gameEnv` let `SeekCoin` use engine-provided setup information.

<a id="single-responsibility"></a>
### Single Responsibility and Organization

```js
initialize() { ... }
buildHud() { ... }
syncHud() { ... }
toggleMenu() { ... }
switchSprite(index) { ... }
update() { ... }
destroy() { ... }
```

- `initialize()` starts level-specific setup.
- `buildHud()` creates visible DOM controls.
- `syncHud()` updates the current coin count display.
- `toggleMenu()` controls the sprite menu state.
- `switchSprite()` changes the player's sprite sheet.
- `update()` checks gameplay state every frame.
- `destroy()` removes event listeners and DOM elements.

## Object-Oriented Programming and Classes

<a id="object-oriented-programming"></a>
### Object-Oriented Programming

```js
class SeekCoin extends GameObject {
  constructor(data, gameEnv) {
    super(gameEnv);
    this.position = data.position;
    this.collected = false;
  }
}
```

- The coin is an object with its own position, canvas, state, and drawing behavior.
- `GameLevelSeek` is another object that owns the level rules.
- This is OOP because state and behavior live together inside objects.

<a id="instantiation-and-objects"></a>
### Instantiation and Objects

```js
this.classes = [
  { class: GameEnvBackground, data: bgData },
  { class: Player, data: this.playerData },
  ...coinPositions.map((position) => ({ class: SeekCoin, data: { position } }))
];
```

- `this.classes` is a configuration array for the GameEngine.
- Each object literal tells the engine what class to create and what data to pass in.
- The spread operator adds one `SeekCoin` definition for every generated coin position.
- This is data-driven design because the level is built from structured object data.

<a id="inheritance-basic"></a>
### Inheritance

```js
import GameObject from '{{site.baseurl}}/assets/js/GameEnginev1/essentials/GameObject.js';
import Player from '{{site.baseurl}}/assets/js/GameEnginev1/essentials/Player.js';

class SeekCoin extends GameObject { ... }
```

- `SeekCoin` inherits from the engine's `GameObject` base class.
- `Player` is imported from the local engine path in this repository.
- The engine also uses a class hierarchy where shared behavior lives in parent classes.
- Inheritance prevents every object from rewriting the same lifecycle pattern.

<a id="method-overriding"></a>
### Method Overriding

```js
update() {
  this.draw();
}

draw() {
  ...
}

destroy() {
  this.canvas.remove();
}
```

- `SeekCoin` provides its own versions of `update()`, `draw()`, `resize()`, and `destroy()`.
- Those methods match the lifecycle expected by `GameObject`.
- Overriding gives the coin custom behavior while still fitting the engine.

<a id="constructor-chaining"></a>
### Constructor Chaining

```js
constructor(data, gameEnv) {
  super(gameEnv);
  this.position = data.position;
}
```

- `super(gameEnv)` runs the parent `GameObject` constructor first.
- Then `SeekCoin` adds its own custom fields.
- This is constructor chaining because setup flows from parent class to child class.

## Data Types and Object Literals

<a id="numbers"></a>
### Numbers

```js
this.radius = 16;
this.canvas.width = 36;
SCALE_FACTOR: 10;
STEP_FACTOR: 700;
ANIMATION_RATE: 8;
```

- Numbers control sprite scale, movement speed, animation rate, coin radius, and canvas size.
- These values make the game measurable and tunable.

<a id="strings"></a>
### Strings

```js
id: 'SeekPlayer',
label: 'Green Seeker',
src: path + '/images/gamebuilder/bg/clouds.jpg'
```

- Strings store IDs, labels, paths, HUD text, and console messages.
- IDs let the program find specific objects later.
- Paths connect the runner to assets in this repository.

<a id="booleans"></a>
### Booleans

```js
this.collected = false;
player.spriteReady = false;
```

- Booleans store true-or-false state.
- `collected` decides whether a coin should be visible.
- `spriteReady` tracks whether the new sprite image has loaded.

<a id="arrays"></a>
### Arrays

```js
this.spriteOptions = [
  { label: 'Green Seeker', src: makeSpriteSheet('#22c55e', '#bbf7d0') },
  { label: 'Blue Seeker', src: makeSpriteSheet('#2563eb', '#bfdbfe') },
  { label: 'Rose Seeker', src: makeSpriteSheet('#e11d48', '#fecdd3') }
];
```

- Arrays store ordered groups of related values.
- `spriteOptions` stores the possible player sprites.
- `coinPositions` stores every collectible location.
- `this.classes` stores every object the engine should instantiate.

<a id="objects-json"></a>
### Objects and JSON-Style Data

```js
const bgData = {
  name: 'seek-background',
  src: path + '/images/gamebuilder/bg/clouds.jpg',
  pixels: { height: 720, width: 1280 }
};
```

- This is a JSON-style object literal.
- One object groups related properties together.
- `pixels` is a nested object.
- The engine reads these structured objects to build the level.

## Operators and Mathematical Reasoning

<a id="mathematical-operators"></a>
### Mathematical Operators

```js
Math.hypot(player.position.x - coin.position.x, player.position.y - coin.position.y) < 55
```

- Subtraction finds the horizontal and vertical distance between player and coin.
- `Math.hypot(...)` converts those two distances into one total distance.
- `< 55` checks whether the player is close enough to collect the coin.

<a id="string-operations"></a>
### String Operations

```js
this.hud.textContent = 'Seek coins: ' + this.collected + '/' + total + ' | Q: sprites | WASD: move';
```

- String concatenation combines labels with live values.
- The HUD updates as the coin count changes.
- This turns internal game state into readable output.

<a id="boolean-expressions"></a>
### Boolean Expressions

```js
if (!coin.collected && player && Math.hypot(...) < 55) {
  coin.collected = true;
}
```

- `!coin.collected` checks that the coin has not already been collected.
- `player` checks that the player object exists.
- `&&` requires every condition to be true before collection happens.

## Control Structures and State Management

<a id="conditionals"></a>
### Conditionals

```js
if (!player) return;
if (coins.length && this.collected === coins.length) {
  this.hud.textContent = 'All Seek coins collected. Press Q to keep testing sprites.';
}
```

- `if` statements decide when code should run.
- The sprite switch stops safely if the player cannot be found.
- The win message only appears after every coin has been collected.

<a id="iteration"></a>
### Iteration

```js
for (const coin of coins) {
  if (!coin.collected && player && Math.hypot(...) < 55) {
    coin.collected = true;
    this.collected += 1;
    this.syncHud();
  }
}
```

- The loop repeats collision-check logic for every coin.
- This is more scalable than writing one separate `if` statement per coin.
- The same code still works if more coin positions are added.

<a id="state-management"></a>
### State Management

```js
this.collected = 0;
this.spriteOptions = [ ... ];
this.menu.style.display = this.menu.style.display === 'none' ? 'flex' : 'none';
```

- State is saved information about what is happening now.
- `collected` tracks progress through the level.
- The menu display value tracks whether the sprite menu is open or closed.
- The current player sprite is stored in `player.spriteData.src`.

## Input, Output, and Rendering

<a id="keyboard-input"></a>
### Keyboard Input

```js
document.addEventListener('keydown', this.onKeyDown);
```

```js
this.onKeyDown = (event) => {
  if (event.key.toLowerCase() === 'q') this.toggleMenu();
};
```

- The browser listens for keyboard input.
- Pressing Q toggles the sprite menu.
- WASD movement comes from the imported `Player` class and its `keypress` configuration.

<a id="canvas-rendering"></a>
### Canvas Rendering

```js
this.ctx.fillStyle = '#f59e0b';
this.ctx.beginPath();
this.ctx.arc(18, 18, 15, 0, Math.PI * 2);
this.ctx.fill();
```

- `ctx` is the canvas drawing context.
- `arc(...)` draws the circular coin shape.
- `fillStyle` chooses the coin color.
- `fill()` paints the coin on the screen.

<a id="gameenv-configuration"></a>
### GameEnv Configuration

```js
const path = gameEnv.path;
{ x: gameEnv.innerWidth * 0.28, y: gameEnv.innerHeight * 0.24 }
this.gameEnv.container.append(this.hud, this.menu);
```

- `gameEnv` provides shared engine information.
- `path` is used for repository asset paths.
- `innerWidth` and `innerHeight` make coin positions responsive to the game area.
- `container` tells the level where DOM elements should be attached.

## Collision Detection, Hitboxes, and Gameplay Logic

<a id="collision-detection"></a>
### Collision Detection

```js
if (!coin.collected && player && Math.hypot(player.position.x - coin.position.x, player.position.y - coin.position.y) < 55) {
  coin.collected = true;
}
```

- The player and coin are compared by distance.
- If the distance is below a threshold, the coin is collected.
- This is a simple circular collision strategy.

<a id="hit-box-visualization"></a>
### Hitbox Configuration

```js
hitbox: { widthPercentage: 0.25, heightPercentage: 0.15 }
```

- The player data includes a hitbox configuration used by the engine.
- Smaller hitboxes can make interactions feel fairer than using the full sprite rectangle.
- This connects visual sprites to gameplay collision behavior.

<a id="dynamic-sprite-system"></a>
### Dynamic Sprite System

```js
player.spriteData.src = option.src;
player.spriteSheet = new Image();
player.spriteReady = false;
player.spriteSheet.onload = () => {
  player.spriteReady = true;
  player.resize();
};
```

- The selected sprite source is swapped at runtime.
- The code creates a new `Image` object.
- The load callback marks the sprite ready and resizes it.
- This demonstrates state changes, browser image loading, and object property updates.

## API-Style I/O, Persistence, and Debugging Evidence

<a id="async-io"></a>
### Asynchronous I/O Pattern

```js
player.spriteSheet.onload = () => {
  player.spriteReady = true;
  player.resize();
};
```

- Loading an image does not finish instantly.
- The browser calls `onload` after the image is available.
- This is an asynchronous event pattern because the callback runs later.

<a id="console-debugging"></a>
### Console Debugging

```js
console.log('GameLevelSeek.js loaded:', new Date().toISOString());
console.log('Sprite switched:', option.label);
```

- Console logs give proof that level setup and sprite switching happened.
- The timestamp helps confirm when the level loaded.
- These messages can be inspected in browser DevTools.

<a id="element-inspection"></a>
### Element Inspection

```js
this.hud = document.createElement('div');
this.menu = document.createElement('div');
this.canvas = document.createElement('canvas');
```

- The level creates DOM elements while the game runs.
- The HUD, menu, buttons, and coin canvases can be inspected in the Elements panel.
- This gives clear debugging evidence for output and layout.

<a id="gameplay-testing"></a>
### Gameplay Testing

- Run the GameRunner and verify the background, player, HUD, and coins appear.
- Move with WASD and confirm the player can reach every coin.
- Collect a coin and confirm the HUD count increases.
- Press Q and confirm the sprite menu opens and closes.
- Click each sprite option and confirm the player image changes.
- Collect all coins and confirm the completion message appears.

## Final CS 111 Alignment

`GameLevelSeek` demonstrates the major CS 111 and CSSE objectives in a direct, playable way.

- It uses custom classes, inheritance, constructor chaining, and method overriding.
- It uses numbers, strings, booleans, arrays, and object literals throughout the level.
- It uses conditionals, loops, boolean expressions, and state management for real gameplay.
- It uses keyboard input, canvas rendering, DOM output, and `GameEnv` configuration.
- It shows asynchronous image loading, console debugging, element inspection, and testable game behavior.

What makes the file especially strong is that the concepts are not isolated practice exercises. They all support one playable level, so each programming concept connects to something the player can actually see and test.
