# Vite Plugin Web Extension

An all in one plugin for developing browser extensions using [Vite](https://vitejs.dev/). It supports:

- Browsers
  - Chrome
  - Firefox
- Languages
  - Javascript
  - Typescript
- Frameworks
  - Vue
  - React
  - Svelte
  - And anything else with a plugin for Vite!

```ts
// vite.config.ts
import browserExtension from "vite-plugin-web-extension";

export default defineConfig({
  plugins: [
    browserExtension({
      manifest: () => require("./manifest.json"),
      assets: "assets",
    }),
  ],
});
```

And that's it! All entry points are read from the `manifest.json` (or the [`additionalInputs`](#additional-inputs) option)

```bash
npm i -D vite-plugin-web-extension
```

## Roadmap

- [x] Build for production &emsp; :tada: `v0.1.0` :tada:
- [ ] CSS inputs & generated files
- [ ] Dev mode with hot reload

## Setup and Usage

Lets say your project looks like this:

<pre>
<strong>dist/</strong>
   <i>build output...</i>
<strong>src/</strong>
   <strong>assets/</strong>
      <i>icon-16.png</i>
      <i>icon-48.png</i>
      <i>icon-128.png</i>
   <strong>background/</strong>
      <i>index.ts</i>
   <strong>popup/</strong>
      <i>index.html</i>
   <i>manifest.json</i>
<i>package.json</i>
<i>vite.config.ts</i>
<i>...</i>
</pre>

In your `vite.config.ts` and `src/manifest.json`, **make sure all paths are relative to your Vite `root`**!

In this example, we set our it to `"src"`. In the `vite.config.ts` file, the only field that is relative to the root is your `assets` directory.

```ts
// vite.config.ts
import browserExtension from "vite-plugin-web-extension";

export default defineConfig({
  root: "src",
  build: {
    // Configure our outputs - nothing special, this is normal vite config
    outDir: path.resolve(__dirname, "dist"),
    emptyOutDir: true,
  },
  plugins: [
    browserExtension({
      manifest: () => require("./src/manifest.json"),
      assets: "assets",
    }),
  ],
});
```

> The manifest `require` is just a plain JS import. There are no special rules for relative paths

The manifest has a bit more. You should use relative paths for all entry points (HTML, JS, or TS), such as `browser_action.default_popup` or `background.scripts`.

**Your relative paths should also be the real file extension**! If you're using typescript, have any scripts end with their usual `.ts` extension. The plugin will transform the file extensions for you.

```jsonc
// src/manifest.json
{
  "name": "Example",
  "version": "1.0.0",
  "icons": {
    "16": "assets/icon-16.png",
    "48": "assets/icon-48.png",
    "128": "assets/icon-128.png"
  },
  "browser_action": {
    "default_icon": "assets/icon/128.png",
    "default_popup": "popup/index.html"
  },
  "background": {
    "scripts": "background/index.ts"
  }
}
```

### Adding Frontend Framewoks

If you want to add a framework like Vue or React, just add their plugin!

```ts
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  ...
  plugins: [
    browserExtension({ ... }),
    vue(),
  ],
});
```

> The plugin order doesn't matter

## Advanced Features

### Additional Inputs

If you have have files that need to be included, but aren't listed in your `manifest.json`, you can add them via the `additionalInputs` option.

The paths should be relative to the Vite's `root`, just like the `assets` option.

```ts
// vite.config.ts
import browserExtension from "vite-plugin-web-extension";

export default defineConfig({
  root: "src",
  build: {
    // Configure our outputs - nothing special, this is normal vite config
    outDir: path.resolve(__dirname, "dist"),
    emptyOutDir: true,
  },
  plugins: [
    browserExtension({
      ...
      additionalInputs: [
        "content-scripts/injected-from-background.ts",
      ]
    }),
  ],
});
```

### CSS

For HTML entry points like popups or the options page, css is automatically output and refrenced in the built HTML. There's nothing you need to do!

#### Manifest `content_scripts`

TODO, but here's the plan:

For content scripts listed in your `manifest.json`, its a little more difficult. There are two ways to include CSS files:

1. You have a CSS, SCSS, or some other stylesheet on your disk
1. The stylesheet is generated by a framework like Vue or React

For the first case, that's simple! Make sure you have the relevant plugin installed to parse your stylesheet, and you're good!

```json
{
  "content_scripts": [
    {
      "matches": [...],
      "css": "content-scripts/some-style.css"
    }
  ]
}
```

For the second case, it's a little more involved. Say your content script at `content-scripts/overlay.ts`. This script is responsible for binding a Vue/React app to a webpage. When Vite compiles it, it will output two files: `dist/content-scripts/overlay.js` and `dist/content-scripts/overlay.css`.

In the content script section of your `manifest.json`, add add the path to this generated file, but prefix it with `generated:*`

```json
{
  "content_scripts": [
    {
      "matches": [...],
      "scripts": "content-scripts/overlay.ts",
      "css": "generated:content-scripts/overlay.css"
    }
  ]
}
```

This will tell the plugin that the file is already being generated for us, but that we still need it in our manifest so it is injected.

#### Browser API `tabs.executeScripts`

TODO, but here's the plan:

For content scripts injected programmatically, include path in the plugin's [`additionalInputs` option](#additional-inputs)

### Browser Specific Manifest Fields

TODO
