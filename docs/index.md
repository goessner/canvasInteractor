---
"layout": "page",
"title": "canvasInteractor-Main page",
"header": "canvasInteractor",
"date": "2020-03-25",
"description": "microjam for canvasInteractor",
"permalink": "#",
"tags": []
---

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

<script src="https://cdn.jsdelivr.net/gh/goessner/g2/bin/canvasInteractor.js"></script>
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

