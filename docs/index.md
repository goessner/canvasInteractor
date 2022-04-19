---
"lang": "en",
"title": "<code>canvasInteractor</code>",
"subtitle": "Make your HTML canvas Interactive",
"authors": ["Stefan Gössner<sup>1</sup>"],
"adresses": ["<sup>1</sup>Dortmund University of Applied Sciences. Department of Mechanical Engineering"],
"date": "April 2022",
"description": "How to make your canvas element interactive",
"tags": ["canvasInteractor","canvas","javascript","events","requestAnimationFrame","throttling","event loop"]
---

## 1. What is It ?

`canvasInteractor` is a micro-library (9.1 kB uncompressed) used to handle pointer events for simple geometry editing together with one or more HTML canvases.
It implements a global event loop based on `requestAnimationFrame` and supports throttling of `pointermove` and `wheel` events via its custom `tick` event for efficient animation. Cartesian coordinates with user defined origin are possible. It was primarily implemented for use by my students for web-kinematics projects.

`canvasInteractor` is the modern and more minimal successor of deprecated [canvas-area](https://github.com/goessner/canvas-area).


## 2. How to Initialize ?

For each HTML canvas in an HTML document an instance via `canvasInteractor.create` is required.

```html
<canvas id="c" width="601" height="401"></canvas>
<script src="https://cdn.jsdelivr.net/gh/goessner/canvasinteractor/canvasInteractor.js"></script>
<script>
    const ctx = document.getElementById('c').getContext('2d');
    const interactor = canvasInteractor.create(ctx, {x: 300, 
                                                     y: 200, 
                                                     cartesian: true});
    // ...
</script>
```
<figcaption>Listing 1: Constructor &ndash; cartesian coordinates with origin centered.</figcaption><br>

View coordinates provided by events can be controlled in the constructor by an additional view argument `{x=0,y=0,scl=1,cartesian=false}` beside the canvas `RenderingContext2D` object.

* `x,y` &hellip; view's origin location.
* `scl` &hellip;  view's scaling.
* `cartesian` &hellip; cartesian coordinate system (y-axis up).
<br><br><br><br><br><br>

## 3. Handling Events

Each `canvasInteractor` instance handles DOM pointer events and some custom events.

<figcaption> Table 1: Supported Events </figcaption>

| Event | Comment |
|:--|:--|
|`pointermove`  |Pointer moved.  |
|`pointerdown`  |Pointer device button pressed. |
|`pointerup`  |Pointer device button released. |
|`pointerenter`  |Pointer enters canvas. |
|`pointerleave`  |Pointer leaves canvas. |
|`pointercancel`  |DOM event forwarded. |
|`click`  | Pointer device button pressed and released at the same location. |
|`wheel`  |Pointer device wheel event. |
|`tick`  |Throttled timer event. At most every 60 milliseconds. |
|`pan`  |Pan by pointer device. Occuring only with (left) button pressed. |
|`drag`  |Drag by pointer device. Occuring only with (left) button pressed and something was signalled as 'hit'. |

<br>

Instead of registering events via well known `addEventListener`, an application registers events to a `canvasInteractor` instance using thats `on` method.

```js
// ...
interactor.on('pointermove', (e) => { /* do stuff */ })
          .on('tick', (e) => { /* do other stuff */ })
          .on('pan',  (e) => { /* do yet other stuff */ })
          .startTimer();
```
<figcaption>Listing 2: Registering events and starting tick timer.</figcaption><br>

The `pointermove` event in combination with (left) pointer device button pressed also notifies the application of a `drag` or `pan` event, whether an underlying element is `hit` or not. See the example how to control that behavior.

Callback functions registered via `on` recieve an extended event object `e`.


<figcaption> Table 2: Event object properties </figcaption>

| Property | Type | Comment |
|:--|:--:|:--|
|`type` | `string`  | Event type.  |
|`x`, `y` | `number` | Canvas coordinates with respect to upper left or lower left (`cartesian`) corner. |
|`dx`, `dy` | `number` | Pointer location displacement from previous position. |
|`xusr`, `yusr` | `number` | Coordinates with respect to user defined view origin and scaling. |
|`dxusr`, `dyusr` | `number` | Pointer location displacement with respect to user defined view scaling. |
|`btn`  | `number` |  Pointer device button identifier on button press<br> (left: `1`, right: `2`, middle: `4`). |
|`dbtn`  | `number` | Pointer device button difference on button release<br> (left: `-1`, right: `-2`, middle: `-4`). |
|`eps`  | `number` | Some pixel tolerance for selecting/hitting<br> (default = `5`). |
|`inside` | `boolean` | Is pointer currently inside canvas. |
|`delta` | `number` | Wheel delta. |
|`hit` | `boolean` | Needs to be set by application within `pointermove` event. Can be considered then within `tick` event. |

## 4. Example

This example shows how to use the `canvasInteractor`.
The `blue` circle can be `drag`ged, wheras the `orange` circle can not, which induces the `pan` event.

```html
<!doctype html>
<html>
<head>
    <title>canvasInteractor example</title>
</head>

<body>
    <h1>canvasInteractor example</h1>
    <canvas id="c" width="601" height="301" 
            style="border:1px solid black;background-color:snow"></canvas>

    <script src="https://cdn.jsdelivr.net/gh/goessner/canvasinteractor/canvasInteractor.js"></script>
    <script>
    const ctx = document.getElementById('c').getContext('2d');
    const interactor = canvasInteractor.create(ctx, {x:200,
                                                     y:100,
                                                     scl:1,
                                                     cartesian:true});
    const rec1 = {x:50,y:50,b:80,h:60,fs:'orange',lw:4};
    const rec2 = {x:150,y:50,b:100,h:40,fs:'cyan',lw:4};

    function render() {
        // ...
    }

    function hitRec(x,y,rec) {
        return (x > rec.x && x < rec.x + rec.b && y > rec.y && y < rec.y + rec.h);
    }
    interactor
        .on('tick', (e) => {
            render();
        })
        .on('pointermove', (e) => {
            rec1.sel = hitRec(e.xusr,e.yusr,rec1) ? true : false; 
            rec2.sel = hitRec(e.xusr,e.yusr,rec2) ? true : false;
            e.hit = rec1.sel || rec2.sel;
        })
``` 
```html
        .on('wheel',  (e) => {   // zooming about pointer location ...
            interactor.view.x = e.x + e.dscl*(interactor.view.x - e.x);
            interactor.view.y = e.y + e.dscl*(interactor.view.y - e.y);
            interactor.view.scl *= e.dscl;
        })
        .on('pan',  (e) => { 
            interactor.view.x += e.dx; 
            interactor.view.y += e.dy;
        })
        .on('drag', (e) => {
            if (rec1.sel) { rec1.x += e.dxusr; rec1.y += e.dyusr; }
            if (rec2.sel) { rec2.x += e.dxusr; rec2.y += e.dyusr; }
        })
        .startTimer();
    </script>
</body>
</html>
```
<figcaption>Listing 3: canvasInteractor example.</figcaption><br>

<h1>canvasInteractor example</h1>
<canvas id="c" width="601" height="301" style="border:1px solid black;background-color:snow"></canvas><br>
<span id="fps">fps: -</span> <b>|</b> 
<label><input id="cartesian" type="checkbox" onchange="interactor.view.cartesian = !interactor.view.cartesian"> cartesian</label> <b>|</b>
<span id="zoom">zoom-scale: 1</span> <b>|</b> 
<span id="coords">pos: ./.</span> <b>|</b> <span id="state">state: -</span> <b>|</b>

<script src="https://cdn.jsdelivr.net/gh/goessner/canvasinteractor/canvasInteractor.js"></script>
<script>
const ctx = document.getElementById('c').getContext('2d');
const interactor = canvasInteractor.create(ctx, {x:200,y:100,scl:1,cartesian:true});
const rec1 = {x:50,y:50,b:80,h:60,fs:'orange',lw:4};
const rec2 = {x:150,y:50,b:100,h:40,fs:'cyan',lw:4};
const info = {
    coords: document.getElementById('coords'),
    fps:    document.getElementById('fps'),
    state:  document.getElementById('state'),
    cartesian: document.getElementById('cartesian'),
    zoom: document.getElementById('zoom')
}

function render() {
    const w = ctx.canvas.width, h = ctx.canvas.height;
    const {x,y,scl,cartesian} = interactor.view;

    // init ...
    ctx.setTransform(1,0,0,cartesian?-1:1,0.5,cartesian?h-0.5:0.5);
    ctx.clearRect(0, 0, w, h);
    // draw grid
    if (scl > 0.2) {
        const sz = 20*scl;
        ctx.strokeStyle = "#ccc";
        ctx.lineWidth = 1;
        ctx.beginPath();
        for (let u=x%sz,nx=w+1; u<nx; u+=sz) { ctx.moveTo(u,0); ctx.lineTo(u,h); }
        for (let v=y%sz,ny=h+1; v<ny; v+=sz) { ctx.moveTo(0,v); ctx.lineTo(w,v); }
        ctx.stroke();
    }
    // view transformation
    ctx.setTransform(scl, 0, 0, cartesian ? -scl : scl, x+0.5, (cartesian ? h-y : y)+0.5);
    // draw origin ...
    ctx.strokeStyle = "black";
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.stroke(new Path2D('M0,40L0,0L40,0'));
    // ... and rectangles.
    drawRec(rec1);
    drawRec(rec2);
}

function drawRec(rec) {
    ctx.fillStyle = rec.fs;
    ctx.strokeStyle = rec.ls;
    ctx.lineWidth = rec.lw;
    ctx.shadowOffsetX = 0;
    ctx.shadowOffsetY = 0;
    ctx.shadowBlur = rec.sel === true ? 8 : 0;
    ctx.shadowColor = 'black';
    ctx.fillRect(rec.x,rec.y,rec.b,rec.h);
    ctx.shadowBlur = 0;
}

function hitRec(x,y,rec) {
    return (x > rec.x && x < rec.x + rec.b && y > rec.y && y < rec.y + rec.h);
}

function showinfo(e) {
    const prec = Math.max(Math.log(interactor.view.scl)/Math.log(2), 0);
    if (info.action !== 'pan')
        info.coords.innerHTML = `pos: ${(interactor.evt.xusr).toFixed(prec)} / ${(interactor.evt.yusr).toFixed(prec)}`;
    info.fps.innerHTML = "fps: " + canvasInteractor.fps;
    info.state.innerHTML = "action: " + info.action;
    info.zoom.innerHTML = "zoom-scale: " + interactor.view.scl.toFixed(2);
    info.cartesian.checked = interactor.view.cartesian;
}

interactor
    .on('tick', (e) => {
        render();
        showinfo(e);
        info.action = '-';
    })
    .on('pointermove', (e) => {
        rec1.sel = hitRec(e.xusr,e.yusr,rec1) ? true : false; 
        rec2.sel = hitRec(e.xusr,e.yusr,rec2) ? true : false;
        e.hit = rec1.sel || rec2.sel;
    })
    .on('wheel',  (e) => {   // zooming about pointer location ...
        interactor.view.x = e.x + e.dscl*(interactor.view.x - e.x);
        interactor.view.y = e.y + e.dscl*(interactor.view.y - e.y);
        interactor.view.scl *= e.dscl;
    })
    .on('pan',  (e) => { 
        interactor.view.x += e.dx; 
        interactor.view.y += e.dy;
        info.action = 'pan';
    })
    .on('drag', (e) => {
        if (rec1.sel) { rec1.x += e.dxusr; rec1.y += e.dyusr; }
        if (rec2.sel) { rec2.x += e.dxusr; rec2.y += e.dyusr; }
        info.action = 'drag';
    })
    .startTimer();
</script>


