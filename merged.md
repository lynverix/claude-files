## Merged Files List
- 1. events.txt (1.6 KB)
- 2. methods.txt (4.9 KB)
- 3. configuration.txt (3.1 KB)
- 4. headless-mode.txt (7.3 KB)
- 5. quickstart.txt (1.5 KB)


## 1. events.txt

```txt
# Events

Listen for events via callbacks in config, or with the `on`/`off` methods. Whichever fits your code.

## Available events

| Event | Payload | When it fires |
| --- | --- | --- |
| `ready` | none | SDK is connected, grid is loading. |
| `gameStart` | `{ id: string, name: string }` | A game is opening. |
| `gameEnd` | none | The game player was closed. |
| `error` | `Error` | Something went wrong. |

## Using callbacks in config

```
Lumin.init({  container: '#games',  theme: 'dark',
  onReady: () => {    document.getElementById('status').textContent = 'Ready';  },
  onGameStart: (game) => console.log('Now playing:', game.name),  onGameEnd: () => console.log('Game closed'),  onError: (err) => console.error('SDK error:', err.message),});
```

## Using on / off

```
function handleGameStart(game) {  console.log('Playing:', game.name);  analytics.track('game_started', { gameId: game.id });}
Lumin.on('gameStart', handleGameStart);Lumin.off('gameStart', handleGameStart);
```

Caution

`off()` requires the **exact same function reference** passed to `on()`. Anonymous functions won’t work.

## Common patterns

### Loading state

```
Lumin.init({  container: '#games',  theme: 'dark',  onReady: () => loader.style.display = 'none',  onError: () => loader.textContent = 'Failed to load games',});
```

### Pause background music during gameplay

```
Lumin.on('gameStart', () => bgMusic.pause());Lumin.on('gameEnd', () => bgMusic.play());
```

### Analytics

```
Lumin.on('gameStart', (game) => {  gtag('event', 'game_play', {    game_id: game.id,    game_name: game.name,  });});
```
```

## 2. methods.txt

```txt
# Methods

After calling `Lumin.init()`, the global `Lumin` object exposes these methods. All async methods return Promises.

## init

Initialize the SDK and render the game grid.

```
await Lumin.init({  container: '#games',  theme: 'dark',});
```

| Param | Type | Description |
| --- | --- | --- |
| `config` | `object` | See [Configuration](/configuration/) for all options. |

**Returns:** `Promise<void>`. Resolves when connected and loading.

Calling `init()` again destroys the previous instance and starts fresh.

* * *

## search

Search games by name. The grid updates automatically with results.

```
const results = await Lumin.search('snake');console.log(results.total);   // matching countconsole.log(results.games);   // game objects
```

| Param | Type | Description |
| --- | --- | --- |
| `query` | `string` | Search text. Empty string returns all games. |

**Returns:** `Promise<{ games, total, page, pages }>`

* * *

## loadGame

Open a specific game by ID in the fullscreen player.

```
await Lumin.loadGame('space-invaders');
```

| Param | Type | Description |
| --- | --- | --- |
| `gameId` | `string` | Game ID from `search()` or `getGames()` results. |

**Returns:** `Promise<void>`. Resolves when the game is loaded.

* * *

## getGames

Fetch a page of games without updating the UI. Useful for building custom displays.

```
const page1 = await Lumin.getGames({ page: 1, limit: 20 });
```

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| `page` | `number` | `1` | Page number. |
| `limit` | `number` | your `gamesPerPage` | Games per page. |
| `q` | `string` | `''` | Optional search query. |

**Returns:** `Promise<{ games, total, page, pages }>`

Note

Returns data only. It does not update the rendered grid. Use `search()` for that.

* * *

## getRandomGames

Get a random selection of games.

```
const random = await Lumin.getRandomGames(12);
```

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| `count` | `number` | `12` | How many random games to return. |

**Returns:** `Promise<{ games }>`

* * *

## getCategories

Fetch all available game categories.

```
const { categories } = await Lumin.getCategories();console.log(categories); // ['Action', 'Puzzle', 'Racing', ...]
```

**Returns:** `Promise<{ categories: string[] }>`

Note

Works in both standard and [headless](/headless/) mode.

* * *

## getGameUrl

Get the playable iframe URL for a game without opening the built-in player. Use this when you want to embed the game in your own UI.

```
const { url, meta } = await Lumin.getGameUrl('space-invaders');
// Embed in your own iframeconst iframe = document.createElement('iframe');iframe.src = url;iframe.allow = 'autoplay; fullscreen; pointer-lock; gamepad';document.body.appendChild(iframe);
```

| Param | Type | Description |
| --- | --- | --- |
| `gameId` | `string` | Game ID from `getGames()`, `search()`, or `getRandomGames()` results. |

**Returns:** `Promise<{ url: string, meta: { id, name, category, ... } }>`

Caution

The returned URL contains a single-use token. Create a new URL (call `getGameUrl` again) each time you want to load the game.

* * *

## getImageUrl

Resolve a game’s image token to a usable blob URL. Each game object from `getGames()` / `search()` / `getRandomGames()` includes an `image_token` field.

```
const { games } = await Lumin.getGames({ page: 1, limit: 10 });
for (const game of games) {  const src = await Lumin.getImageUrl(game.image_token);  // src is a blob URL you can set on an <img>}
```

| Param | Type | Description |
| --- | --- | --- |
| `token` | `string` | The `image_token` from a game object. |

**Returns:** `Promise<string>`. A blob URL usable as an `img.src`.

Note

In standard (non-headless) mode, the SDK resolves images automatically for the grid. This method is primarily useful in [headless mode](/headless/) where you render your own UI.

* * *

## endGame

Close the currently playing game. No-op if nothing is open.

```
Lumin.endGame();
```

* * *

## destroy

Tear down the SDK. Removes the UI, disconnects, and cleans up listeners.

```
Lumin.destroy();
```

You can call `init()` again after this, but you’ll need a fresh container element (the old one has a shadow root attached).

* * *

## on

Subscribe to an SDK event. See [Events](/events/) for event names.

```
Lumin.on('gameStart', (game) => {  console.log('Playing:', game.name);});
```

| Param | Type | Description |
| --- | --- | --- |
| `event` | `string` | Event name. |
| `callback` | `function` | Handler function. |

* * *

## off

Unsubscribe from an SDK event. Must pass the same function reference used in `on()`.

```
Lumin.off('gameStart', handler);
```

| Param | Type | Description |
| --- | --- | --- |
| `event` | `string` | Event name. |
| `callback` | `function` | The function reference passed to `on()`. |
```

## 3. configuration.txt

```txt
# Configuration

Tip

All the configuration here lets you configure the default Lumin UI rendered in your `container`. If you want maximum control over your UI, check out unless using [headless mode](/headless/).

```
Lumin.init({  container: '#games',    // required (unless headless)  theme: 'dark',          // defaults to 'auto'  columns: 8,             // defaults to 8  rows: 4,                // defaults to 3});
```

## All options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `container` | `string | HTMLElement` | — | **Required** (unless `headless: true`). CSS selector or DOM element for the game grid. |
| `headless` | `boolean` | `false` | Skip all UI rendering. Connects the worker and exposes data/game-loading methods only. See [Headless Mode](/headless/). |
| `theme` | `'dark' | 'light' | 'auto'` | `'auto'` | Color theme. `auto` follows system preference. |
| `columns` | `number` | `8` | Number of grid columns. |
| `rows` | `number` | `3` | Number of rows per page. |
| `gamesPerPage` | `number` | `columns * rows` | Override the auto-calculated games per page. |
| `showSearch` | `boolean` | `true` | Show the search bar. |
| `showCategories` | `boolean` | `true` | Show the category filter bar. |
| `showRandom` | `boolean` | `true` | Show the “Random” shuffle button. |
| `onReady` | `() => void` | — | Fires when the SDK is connected and the grid is loading. |
| `onGameStart` | `(game) => void` | — | Fires when a game opens. Receives `{ id, name }`. |
| `onGameEnd` | `(game) => void` | — | Fires when the game player closes. |
| `onError` | `(err) => void` | — | Fires on errors. Receives an `Error` object. |

## Grid sizing

`columns` and `rows` control tile count. `gamesPerPage` is auto-calculated as `columns * rows` unless you override it.

```
// Auto calculates games per page// 8 columns x 4 rows = 32 games per pageLumin.init({ container: '#games', columns: 8, rows: 4 });
// Only return 50 games regardless of grid shapeLumin.init({ container: '#games', columns: 10, gamesPerPage: 50 });
```

Note

The grid is responsive. Even with `columns: 8`, it collapses to fewer columns on smaller screens.

## Themes

-   [Dark](#tab-panel-18)
-   [Light](#tab-panel-19)
-   [Auto](#tab-panel-20)

```
Lumin.init({ container: '#games', theme: 'dark' });
```

Dark background, light text.

```
Lumin.init({ container: '#games', theme: 'light' });
```

Light background, dark text.

```
Lumin.init({ container: '#games', theme: 'auto' });
```

Matches OS preference. Switches when the system does.

## Hiding UI elements

```
Lumin.init({  container: '#games',  theme: 'dark',  showSearch: false,  showCategories: false,  showRandom: false,});
```

## Events

You can configure event callbacks to listen for events during the SDK’s lifecycle. For more info, see See [Events](/events/).

```
Lumin.init({  container: '#games',  theme: 'dark',
  onReady: () => console.log('SDK ready'),
  onGameStart: (game) => {    console.log('User launched', game.name);  },
  onGameEnd: () => console.log('Game closed'),
  onError: (err) => console.error('Error:', err.message),});
```
```

## 4. headless-mode.txt

```txt
# Headless Mode

Headless mode gives you access to the full game catalog, search, categories, and game loading without rendering any built-in UI. You build the interface; Lumin handles the data and game infrastructure.

Headless mode gives you full access to Lumin’s game catalog and functionality for you to render in your own applications, if you don’t want to use Lumin’s default, embedded UI.

```
await Lumin.init({ headless: true });
```

This initializes the Lumin SDK and lets you use further API methods to fetch game data.

## Why headless?

-   You want to get the full benefits of Lumin’s SDK, but use it with your existing site design
-   You’re building in a UI library/framework (React, Svelte, Solid, etc.) and want to use your own or reusable components
-   You only need a subset of the UI (e.g., a featured game section, not a full grid)

## Quick start

1.  **Add the script**
    
    ```
    <!-- For static HTML sites --><script src="https://cdn.jsdelivr.net/gh/luminsdk/script@latest/lumin.min.js"></script>
    ```
    
    ```
    // For bundled sites, like with Viteimport "https://cdn.jsdelivr.net/gh/luminsdk/script@latest/lumin.min.js"
    ```
    
2.  **Initalize with Headless mode**
    
    ```
    await Lumin.init({ headless: true });
    ```
    
3.  **Fetch and display games with your own UI**
    
    ```
    const { games, total, pages } = await Lumin.getGames({ page: 1, limit: 20 });
    for (const game of games) {  const imgUrl = await Lumin.getImageUrl(game.image_token);  // Render your own card with game.name, game.id, imgUrl, etc.}
    ```
    
4.  **Launch a game**
    
    ```
     // Either load game game directly in the configured container await Lumin.loadGame('space-invaders');
     // Or, get the URL to put in your own <iframe> const { url } = await Lumin.getGameUrl('space-invaders');
    ```
    

## Available methods

All standard data methods work in headless mode:

| Method | Description |
| --- | --- |
| [`getGames(opts)`](/methods/#getgames) | Paginated game list with optional search/filter |
| [`getRandomGames(count)`](/methods/#getrandomgames) | Random selection of games |
| [`search(query)`](/methods/#search) | Search by name (data only in headless mode, no grid update) |
| [`getCategories()`](/methods/#getcategories) | All available category names |
| [`getGameUrl(gameId)`](/methods/#getgameurl) | Playable iframe URL without opening the player |
| [`getImageUrl(token)`](/methods/#getimageurl) | Resolve an image token to a blob URL |
| [`loadGame(gameId)`](/methods/#loadgame) | Open a game in the built-in fullscreen player |
| [`endGame()`](/methods/#endgame) | Close the fullscreen player |

See [Methods](/methods/) for full parameter details and return types.

## Game objects

Methods like `getGames()` and `search()` return game objects with this shape:

```
{  id: 'space-invaders',       // unique ID, pass to loadGame() or getGameUrl()  name: 'Space Invaders',     // display name  image_token: 'abc123...',   // pass to getImageUrl() to get the thumbnail  category: 'Arcade',         // optional category}
```

## Loading images

Game objects return an `image_token` property instead of returning a direct image URL - you can use `getImageUrl()` to resolve it to a URL.

```
const { games } = await Lumin.getGames({ page: 1, limit: 12 });
const images = await Promise.all(  games.map(g => Lumin.getImageUrl(g.image_token)));
games.forEach((game, i) => {  // game.name  // images[i]  console.log(game.name, images[i]);});
```

Tip

Batch your `getImageUrl()` calls with `Promise.all()` to resolve image URLs concurrently.

## Using the built-in player

Even when headless mode is enabled, `loadGame()` will still use the default functionality of loading games in fullscreen.

```
await Lumin.loadGame('snake-classic');// Player opens fullscreen. User presses ESC or clicks the close button to exit.
Lumin.on('gameEnd', () => {  console.log('Player closed the game');});
```

## Using your own player

If you want full control over the game’s iframe, use `getGameUrl()` instead:

```
const { url, meta } = await Lumin.getGameUrl('snake-classic');
const iframe = document.querySelector('#player');iframe.src = url;
```

```
<iframe id="player" allow="autoplay; fullscreen; pointer-lock; gamepad" style="" allowfullscreen></iframe>
```

Caution

Game URLs are a one-time use, so you need to call `getGameUrl()` every time a user loads into a game. You can’t cache or reuse the URLs you get from this method.

## Framework examples

-   [React](#tab-panel-24)
-   [Vue](#tab-panel-25)
-   [Vanilla JS](#tab-panel-26)

```
import { useEffect, useState } from 'react';
function GameGrid() {  const [games, setGames] = useState([]);  const [images, setImages] = useState({});
  useEffect(() => {    Lumin.init({ headless: true }).then(async () => {      const { games } = await Lumin.getGames({ page: 1, limit: 12 });      setGames(games);
      const imgs = {};      await Promise.all(games.map(async (g) => {        imgs[g.id] = await Lumin.getImageUrl(g.image_token);      }));      setImages(imgs);    });    // important: safely cleanup the SDK    return () => Lumin.destroy();  }, []);
  return (    <div className="grid">      {games.map(game => (        <div key={game.id} onClick={() => Lumin.loadGame(game.id)}>          <img src={images[game.id]} alt={game.name} />          <span>{game.name}</span>        </div>      ))}    </div>  );}
```

```
<template>  <div class="grid">    <div v-for="game in games" :key="game.id" @click="play(game.id)">      <img :src="images[game.id]" :alt="game.name" />      <span>{{ game.name }}</span>    </div>  </div></template>
<script setup>import { ref, onMounted, onUnmounted } from 'vue';
const games = ref([]);const images = ref({});
async function play(id) {  await Lumin.loadGame(id);}
onMounted(async () => {  await Lumin.init({ headless: true });  const result = await Lumin.getGames({ page: 1, limit: 12 });  games.value = result.games;
  const imgs = {};  await Promise.all(result.games.map(async (g) => {    imgs[g.id] = await Lumin.getImageUrl(g.image_token);  }));  images.value = imgs;});// important: safely cleanup the SDKonUnmounted(() => Lumin.destroy());</script>
```

```
<div id="my-games"></div><script src="https://cdn.jsdelivr.net/gh/luminsdk/script@latest/lumin.min.js"></script><script>  (async () => {    await Lumin.init({ headless: true });
    const { games } = await Lumin.getGames({ page: 1, limit: 12 });    const container = document.getElementById('my-games');
    for (const game of games) {      const img = await Lumin.getImageUrl(game.image_token);      const card = document.createElement('div');      card.innerHTML = `<img src="${img}" alt="${game.name}"><p>${game.name}</p>`;      card.onclick = () => Lumin.loadGame(game.id);      container.appendChild(card);    }  })();</script>
```

For a complete standalone example, see [Headless Example](/examples/headless/).

## Events

All events work normally in headless mode:

```
Lumin.on('ready', () => console.log('Connected'));Lumin.on('gameStart', (game) => console.log('Playing:', game.name));Lumin.on('gameEnd', () => console.log('Game closed'));Lumin.on('error', (err) => console.error(err));
```

See [Events](/events/) for the full list.
```

## 5. quickstart.txt

```txt
# Quickstart

Three steps. No accounts, no API keys, no npm install.

1.  **Add the script**
    
    Drop this before `</body>`:
    
    ```
    <script src="https://cdn.jsdelivr.net/gh/luminsdk/script@latest/lumin.min.js"></script>
    ```
    
2.  **Add a container**
    
    ```
    <div id="games"></div>
    ```
    
3.  **Initialize**
    
    ```
    <script>  Lumin.init({    container: '#games',    theme: 'dark'  });</script>
    ```
    
    The grid loads automatically. Click any tile to play fullscreen, press **ESC** to close.
    

## Full copy-paste example

A complete standalone HTML page:

```
<!DOCTYPE html><html lang="en"><head>  <meta charset="utf-8">  <meta name="viewport" content="width=device-width, initial-scale=1">  <title>My Game Page</title>  <style>    body {      margin: 0;      background: #0a0a0a;    }  </style></head><body>  <div id="games"></div>
  <script src="https://cdn.jsdelivr.net/gh/luminsdk/script@latest/lumin.min.js"></script>  <script>    Lumin.init({      container: '#games',      theme: 'dark',      onReady: () => console.log('LuminSDK is ready'),      onError: (err) => console.error('LuminSDK error:', err),    });  </script></body></html>
```

The grid includes search, category filters, and pagination by default. You can turn any of those off in [Configuration](/configuration/).

Note

Works on any website: static HTML, WordPress, React, Next.js, Vue, Svelte. If it runs in a browser, it works.
```
