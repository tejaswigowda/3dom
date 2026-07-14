# 3D Object Model

**jQuery for 3D.** Address and edit *any* three.js scene with CSS-like selectors,
deterministic auto-labelling, and undoable ops. All in one line:

```js
$S('.wheel').recolor('#111').scale(1.2);
$S.undo();
```

<img src="https://raw.githubusercontent.com/tejaswigowda/3dom/main/examples/demo.gif" alt="3DOM: auto-labelling an imported 3MF model, then selecting and editing it with $S()" width="100%">

No editor. No framework. No build step required. Just three.js (a **peer** dependency) and this.


▶ **Documentation:** http://tejaswigowda.com/3dom/  
▶ **Repository:** https://github.com/tejaswigowda/3dom  
▶ **Live demo:** [bare.html](http://tejaswigowda.com/3dom/examples/bare.html) · [bare-3mf.html](http://tejaswigowda.com/3dom/examples/bare-3mf.html)


---

## Features

◆ **Selectors:** Query the scene graph like the DOM: `mesh`, `.red`, `#Body`, `.wheel:visible`, `light, camera`.

◆ **Auto-labelling:** Derive stable classes from geometry, colour, material and name. So `.red`, `.box`, `.wheel` *just work* on scenes you didn't author.

◆ **Undoable ops:** Every mutation goes through a **Host**. Out of the box you get a built-in undo/redo stack. Drop in your app's command system to reuse it.

◆ **Tiny:** ~24 kB minified. three.js stays external.


---

## Installation

Straight from a CDN. No build, no publish step. jsDelivr serves the repository's `dist/` directly from GitHub. `three` is a **peer dependency**, so include it via the same import map.

Copy this into your page's `<head>`:

```html
<script type="importmap">
{ "imports": {
  "three": "https://cdn.jsdelivr.net/gh/mrdoob/three.js@r160/build/three.module.js",
  "@tejaswigowda/3dom": "https://cdn.jsdelivr.net/gh/tejaswigowda/3dom@v1.0.0/dist/3dom.esm.min.js"
} }
</script>
<script type="module">
  import { createS, autoLabel } from '@tejaswigowda/3dom';
  // autoLabel(scene); const $S = createS(scene);
</script>
```

Pinned to a tag (as above) for an immutable, long-cached URL. `@main` also resolves but is mutable and cached for up to a week. Alternative CDNs use the same path shape:

**jsDelivr:**
```
https://cdn.jsdelivr.net/gh/tejaswigowda/3dom@v1.0.0/dist/3dom.esm.min.js
```

**Statically:**
```
https://cdn.statically.io/gh/tejaswigowda/3dom/v1.0.0/dist/3dom.esm.min.js
```

> **Note:** Strata editor consumes this library exactly this way. Its import map maps `@tejaswigowda/3dom` to the pinned jsDelivr URL, then `strata3dom.js` imports it by bare specifier.

---

## Quick Start

Here's a 60-second example using bare three.js:

```js
import * as THREE from 'three';
import { createS, autoLabel } from '@tejaswigowda/3dom';

const scene = new THREE.Scene();
// ... add your meshes ...

autoLabel( scene );            // derive .red / .box / .wheel / … classes
const $S = createS( scene );   // bind selector, with its own undo history

$S('.wheel').recolor('#ff3b3b').scale(1.2);
$S('mesh').rotate('y', 30);
$S('.roof').setVisible(false);

$S.undo();   // built-in, reversible
$S.redo();

$S.listSelectors();  // [{ selector: '.wheel', count: 4 }, …]
```

**Live demo:** Open [bare.html](http://tejaswigowda.com/3dom/examples/bare.html), orbit/pan/zoom the scene, click selectors and ops, and type your own `$S(...)` commands in the live shell. Source: [`examples/bare.html`](examples/bare.html). No editor, no bundler required.

---

## Selectors

Query the scene graph using familiar CSS-like syntax:

| Selector          | Matches                                          |
|:------------------|:-------------------------------------------------|
| `mesh` `light` …  | By object type: `mesh`, `group`, `light`, `camera`, `sprite`, `line`, `points`, `bone`, `object3d` |
| `.red` `.wheel`   | By class (author-set or auto-labelled)           |
| `#Body`           | By id: `userData.label`, falling back to `.name` |
| `.a.b`            | Compound (AND)                                   |
| `A B`             | Descendant combinator                            |
| `A > B`           | Child combinator                                 |
| `*`               | All objects                                      |
| `:selected`       | Host app's live selection (editor-bound)         |

See [SPEC.md](SPEC.md) for the full grammar and auto-label rules.

---

## API Chain

All operations return `this` for chaining, and are undoable by default.

**Reading (non-mutating):**  
`.nodes` · `.length` · `.count` · `.exists` · `.names` · `.first` · `.last` · `.classes()` · `.each(fn)` · `.toArray()`

**Traversal (new set):**  
`.not(sel)` · `.parent()` · `.children()` · `.filter(pred)`

**Mutation (undoable):**  
`.recolor()` · `.scale()` · `.move()` · `.rotate()` · `.delete()` · `.duplicate()` · `.setMaterial()` · `.setOpacity()` · `.setVisible()` · `.wireframe()` · `.castShadow()` · `.receiveShadow()` · `.renderOrder()` · `.metalness()` · `.roughness()`

**Labeling:**  
`.addClass()` · `.removeClass()` · `.editID(name)`

**Introspection:**  
`$S.listSelectors()`

**JSON ops:**  
`.op(json)` · `.ops([...])`

---

## Custom Undo (Host Integration)

The ops layer doesn't import a command class. Instead, it calls **Host** factories. This design allows you to bring your own undo/redo system or use the built-in `DefaultHost`:

**Built-in (default):**
```js
const $S = createS(scene);  // uses DefaultHost internally
```

**Custom host (for editor integration):**
```js
const $S = createS({
  scene,
  execute: ( cmd ) => editor.execute( cmd ),   // your command bus
  undo:    () => editor.undo(),
  redo:    () => editor.redo(),
  notify:  ( kind ) => signals[ kind ]?.dispatch(),
  // command factories → your real commands
  setMaterialColor: ( obj, hex ) => new SetMaterialColorCommand( editor, obj, hex ),
  // ...
});
```

This is exactly how the **Strata editor** consumes 3DOM: the library is the model, Strata is the host.

---

## License

MIT © Tejaswi Gowda  
three.js is a peer dependency under its own license.
