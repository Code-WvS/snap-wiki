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

Your block needs to do something; you have these choices to define a function:
* add your function `foo` to `threads.js`, for example `Process.prototype.foo = function (bar) { console.log(bar); };`
  * if you want to access the stage from inside the function you can get it via `this.homeContext.receiver.parentThatIsA(StageMorph)`, `this.homeContext.receiver` is the caller of the block
* add your function `foo` to the sprite and/or the stage in `objects.js`, like `SpriteMorph.prototype.foo = function (bar) { console.log(bar); }; StageMorph.prototype.foo = SpriteMorph.prototype.foo;`

If you want to do your action continuously, use a `threads.js` function. There you can append
```javascript
this.pushContext('doYield');
this.pushContext();
```
and your function will be called on every tick. You can save something inside the context (`this.context.test = "Hello World!"`) and it will therefore be available in the following executions. A reporter can return something with `return`, this can even happen after multiple cycles.
The advantage of adding your function to a `SpriteMorph` or `StageMorph` is that you can access the stage/sprite directly with `this` and the functions for either of them can variate.

Drawing something on the stage
------------------------------

### Easy method
To get something instantly on the stage the easiest way is to overwrite the pen trails. `stage.trailsCanvas` is a canvas of the stage's width and height. It lies behind the sprites but on top of the stage's costume. Usually, the dimensions are 360 * 480px but you should not rely on that as it can be changed in the settings by the user.
Outside the deep insides of Snap*!* you need to get the stage object, for example like this: `var stage = world.children[0].stage`. In theory there could be more than one IDE (`world.children`) but most often this short assignment is enough. `trailsCanvas` behaves like a normal canvas and has a context you can draw on. Drawing should happen in `snap.html`'s `loop` but it does not need to; you can redraw with `stage.changed()` otherwise.

### Div method
Sometimes you need to have HTML as your 'stage' and a canvas is not enough. Luckily, modifying `StageMorph.prototype.drawOn` is very straightforward.
Get your element, preferrably a `div`. Resizing and positioning can be adopted from the original `drawOn` (`div` is your element):
```javascript
var div = document.getElementById('div');
// make sure to draw the pen trails canvas as well
var rectangle, area, delta, src, w, h, sl, st;
if (!this.isVisible) {
    return null;
}
rectangle = aRect || this.bounds;
area = rectangle.intersect(this.bounds).round();
if (area.extent().gt(new Point(0, 0))) {
    delta = this.position().neg();
    src = area.copy().translateBy(delta).round();
        
    sl = src.left();
    st = src.top();
    w = Math.min(src.width(), this.image.width - sl);
    h = Math.min(src.height(), this.image.height - st);

    if (w < 1 || h < 1) {
        return null;
    }
    div.style.width = w + 'px';
    div.style.height = h + 'px';
    div.style.left = area.left() + 'px';
    div.style.top = area.top() + 'px';
}
```
