Getting started
===============

First, you'll need the source code. Learn how to use Git and clone the official repository: https://github.com/jmoenig/Snap--Build-Your-Own-Blocks

You probably want to add your own 'native' block now.

Adding a block to an existing category
--------------------------------------

Take a look at `objects.js`. Scroll down to `SpriteMorph.prototype.initBlocks = function()`. To keep the source code clean, you should search the appropriate category (indicated by a comment, e.g. `// Motion`). Add your specification there. It looks like this:
```
foo: {
  dev: true,
  only: SpriteMorph,
  type: 'command',
  category: 'motion',
  spec: 'do something funny with %s',
  defaults: 'Snap',
}
```
* `foo` is the function that will be called later.
* `dev` (boolean, default `false`) means that the block is visible in the developer mode only (shift-click the logo in the top-left corner to enter it).
* `only` (`SpriteMorph` or `StageMorph`, default both) lets the block appear either in the sprite's block list or the stage's block list only.
* `type` (`hat`, `command`, `reporter`, `predicate`, required) sets the shape of the block.
* `category` (`motion`, `looks`, ... other, required) will change the color of the block. It does not let it appear in that category yet!
* `spec` (string, required) sets the text of the block. `%s`, `%n` and others (see `blocks.js SyntaxElementMorph.prototype.labelPart` for more) create inputs. `%s` would create a string input and `%n` a number one. The text will be localized if possible.
* `defaults` (array, default `[]`) sets the default values of the inputs.

StageMorph and SpriteMorph share the same `blocks`.

To let your block appear in a specific category, look at `SpriteMorph.prototype.blockTemplates` or `StageMorph.prototype.blockTemplates` (important: if you want your block to appear in both, add it to *every* `blockTemplates`, this is not that obvious because the source for both look similar). After `if (cat === 'motion') {`, where `motion` is the desired category, you can do a `blocks.push(block('foo'));` where `foo` is the name from above. `blocks.push('-')` adds some space between two blocks and `blocks.push('=')` even more. To add a block only in development mode, you can check `this.world().isDevMode`. For a reporter you have the option to add a `blocks.push(watcherToggle('yourblock'))` before the actual block definition so the user is able to add activate a watcher.
