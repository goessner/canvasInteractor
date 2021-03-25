---
"layout": "page",
"title": "canvasInteractor-Main page",
"header": "canvasInteractor",
"date": "2020-03-25",
"description": "microjam for canvasInteractor",
"permalink": "#",
"tags": []
---

## canvasInteractor

canvasInteractor is a micro-library used to handle events.
Instead of everything registering a new event `addEventListener`, a unified instance handles them.
Components then register themself to a `canvasInteractor` instance.

To create an instance `canvasInteractor.create` is issued.
`create` returns an object which implements the prototype of `canvasInteractor` (applying submitted properties on costruction).

To create a `canvasInteractor` instance for a canvas, the `RenderingContext2D` can be submitted.

```html
<canvas id="c"></canvas>
<script>
    const ctx = document.getElementById('c').getContext('2d');
    const interactor = canvasInteractor.create(ctx, {x: 200, y: 300, cartesian: true});
```

Events can be registered using `on`:

```js
interactor
    .on('pointerdown', (e) => { /* do stuff */ })
    .on('tick', (e) => { /* do other stuff */ })
    .on('pan',  (e) => { /* do yet other stuff */ })
    .startTimer();
```

`on` uses the key of an eventhandler as string and a function which is executed whenever said event is occuring.

Besides commonly used events like:

 - pointerdown
 - pointerup
 - pointerleave
 - pointerenter
 - pointercancel
 - keydown
 - wheel

Custom events like `tick` are used to unify the usage of `requestAnimationFrame`.
`pointermove` is distinguished between `drag` and `pan`, whether an underlying element is `hit` or not.

### Example

This example shows how to use the `canvasInteractor` with [g2](https://github.com/goessner/g2).
The `blue` circle can be `drag`ged, wheras the `orange` circle can not, which induces the `pan` event.

```js
const selector = g2.selector(interactor.evt);  // sharing 'evt' object ... !
const g = g2().clr()                           // important with 'interaction'
              .view(interactor.view)           // view sharing ... !
              .grid()
              .cir({x:0,y:0,r:30,fs:'blue',draggable:true})
              .cir({x:-100,y:0,r:30,fs:'orange'});

interactor
    .on('tick', (e) => { g.exe(selector).exe(ctx); })
    .on('pan',  (e) => { interactor.view.x += e.dx; interactor.view.y += e.dy; })
    .on('drag', (e) => { 
        if (selector.selection && selector.selection.drag) {
            selector.selection.drag({x:e.xusr,y:e.yusr,dx:e.dxusr,dy:e.dyusr,mode:'drag'});
        }
    });
```

<script src="https://cdn.jsdelivr.net/gh/goessner/canvasInteractor@wip/key_events/canvasInteractor.js"></script>
<canvas id="c" width="601" height="401" style="border:1px solid black;"></canvas>
<script>
const ctx = document.getElementById('c').getContext('2d');
const interactor = canvasInteractor.create(ctx, {x:300,y:200,cartesian:true});
const selector = g2.selector(interactor.evt);  // sharing 'evt' object ... !
const g = g2().clr()                           // important with 'interaction'
              .view(interactor.view)           // view sharing ... !
              .grid()
              .cir({x:0,y:0,r:30,fs:'blue',draggable:true})
              .cir({x:-100,y:0,r:30,fs:'orange'});

interactor
    .on('tick', (e) => { g.exe(selector).exe(ctx); })
    .on('pan',  (e) => { interactor.view.x += e.dx; interactor.view.y += e.dy; })
    .on('drag', (e) => { 
        if (selector.selection && selector.selection.drag) {
            selector.selection.drag({x:e.xusr,y:e.yusr,dx:e.dxusr,dy:e.dyusr,mode:'drag'});
        }
    })
    .startTimer();
</script>

Key events can be implemented using the `keydown` event.
Please keep in mind that `keydown` events are not registered to the `canvas` where the `CanvasRenderingContext2D` is defined, but on `globalThis`.

```js
const circle = g.commands[g.commands.length - 1].a;
    function move(sel, code, stepsize = 10) {
        switch (code) {
            case 37:
                sel.x -= stepsize;
                break;
            case 38:
                sel.y += stepsize;
                break;
            case 39:
                sel.x += stepsize;
                break;
            case 40:
                sel.y -= stepsize;
                break;
            default:
                break;
        }
    }

interactor
    .on('tick', (e) => {
        if (interactor.keys && interactor.keys.includes(16)) {
            interactor.keys.forEach(key => {
                move(circle, key);
            });
        }
        console.log(interactor.keys)
        g.exe(selector).exe(ctx);
    })
    .on('keydown', (e) => {
        if (interactor.keys && !interactor.keys.includes(16)) {
            move(circle, interactor.keys[0]);
        }
    })
    .startTimer();
```

<canvas id="c2" width="601" height="401" style="border:1px solid black;"></canvas>
<script>
    const ctx2 = document.getElementById('c2').getContext('2d');
    const interactor2 = canvasInteractor.create(ctx2, { x: 300, y: 200, cartesian: true });
    const selector2 = g2.selector(interactor2.evt);  // sharing 'evt' object ... !
    const gg = g2().clr()                           // important with 'interaction'
        .view(interactor2.view)           // view sharing ... !
        .grid()
        .cir({ x: 0, y: 0, r: 30, fs: 'orange', draggable: true })

    const circle = gg.commands[gg.commands.length - 1].a;
    function move(sel, code, stepsize = 10) {
        switch (code) {
            case 37:
                sel.x -= stepsize;
                break;
            case 38:
                sel.y += stepsize;
                break;
            case 39:
                sel.x += stepsize;
                break;
            case 40:
                sel.y -= stepsize;
                break;
            default:
                break;
        }
    }

    interactor2
        .on('tick', (e) => {
            if (interactor2.keys && interactor2.keys.includes(16)) {
                interactor2.keys.forEach(key => {
                    move(circle, key);
                });
            }
            gg.exe(selector2).exe(ctx2);
        })
        .on('keydown', (e) => {
            if (interactor2.keys && !interactor2.keys.includes(16)) {
                move(circle, interactor2.keys[0]);
            }
        })
        .startTimer();
</script>

