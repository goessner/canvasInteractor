---
"lang": "en",
"title": "<code>canvasInteractor</code>",
"subtitle": "Make your HTML canvas Interactive",
"authors": ["Stefan Gössner<sup>1</sup>", "<a href='https://github.com/goessner/canvasInteractor'><svg height='16' width='16' viewBox='0 0 16 16'><path fill-rule='evenodd' fill='#1f3939' clip-rule='evenodd' d='M8 0C3.58 0 0 3.58 0 8C0 11.54 2.29 14.53 5.47 15.59C5.87 15.66 6.02 15.42 6.02 15.21C6.02 15.02 6.01 14.39 6.01 13.72C4 14.09 3.48 13.23 3.32 12.78C3.23 12.55 2.84 11.84 2.5 11.65C2.22 11.5 1.82 11.13 2.49 11.12C3.12 11.11 3.57 11.7 3.72 11.94C4.44 13.15 5.59 12.81 6.05 12.6C6.12 12.08 6.33 11.73 6.56 11.53C4.78 11.33 2.92 10.64 2.92 7.58C2.92 6.71 3.23 5.99 3.74 5.43C3.66 5.23 3.38 4.41 3.82 3.31C3.82 3.31 4.49 3.1 6.02 4.13C6.66 3.95 7.34 3.86 8.02 3.86C8.7 3.86 9.38 3.95 10.02 4.13C11.55 3.09 12.22 3.31 12.22 3.31C12.66 4.41 12.38 5.23 12.3 5.43C12.81 5.99 13.12 6.7 13.12 7.58C13.12 10.65 11.25 11.33 9.47 11.53C9.76 11.78 10.01 12.26 10.01 13.01C10.01 14.08 10 14.94 10 15.21C10 15.42 10.15 15.67 10.55 15.59C13.71 14.53 16 11.53 16 8C16 3.58 12.42 0 8 0Z'></path></svg></a>"],
"adresses": ["<sup>1</sup>Dortmund University of Applied Sciences. Department of Mechanical Engineering"],
"date": "April 2022",
"description": "How to make your canvas element interactive",
"tags": ["canvasInteractor","canvas","javascript","events","requestAnimationFrame","throttling","event loop"]
---

## 1. What is It ?

`canvasInteractor` is a JavaScript micro-library (4.7 kB compressed) used to handle pointer events for simple geometry editing together with one or more HTML canvases [[1]](#1).
It implements a global event loop based on `requestAnimationFrame` and supports throttling of `pointermove` and `wheel` events via its custom `tick` event for efficient animation [[2]](#2). Cartesian coordinates for scientific and engineering applications are supported. Pan, drag and zoom operations based on an user defined origin can be done. 

It was primarily implemented for use in engineering education and conference presentations.

`canvasInteractor` is the modern and more minimal successor of deprecated `canvas-area`  [[3]](#3).


## 2. How to Initialize ?

For each HTML canvas in an HTML document an instance via `canvasInteractor.create` is required.

```html
<canvas id="c" width="601" height="401"></canvas>
<script src="https://cdn.jsdelivr.net/gh/goessner/canvasinteractor/canvasInteractor.js"></script>
<script>
    const ctx = document.getElementById('c').getContext('2d');
    const interactor = canvasInteractor.create(ctx, {x: 300,  // view ...
                                                     y: 200,  // ... properties
                                                     cartesian: true});
    // ...
</script>
```
<figcaption>Listing 1: Constructor &ndash; cartesian coordinates with origin centered.</figcaption><br>

View coordinates provided by events can be controlled in the constructor by an additional view argument `{x=0,y=0,scl=1,cartesian=false}` beside the canvas `RenderingContext2D` object.

* `x,y` &hellip; view's origin location.
* `scl` &hellip;  view's scaling.
* `cartesian` &hellip; cartesian coordinate system (y-axis up).


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
|`hit` | `boolean` | Needs to be set by application within `pointermove` event. Can be treated then within `tick` event. |

<br>

## 4. Example

The example shows how to use `canvasInteractor`.
Rectangles can be `drag`ged, whereas the origin symbol can not, which induces the `pan` event. `zoom`ing is done by the pointer device' wheel operation.

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

<br>

### Example Code

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


## 5. Conclusion

`canvasInteractor` is a tiny JavaScript library enhancing and extending HTML canvas' event handling for performant animation and geometrical interaction.

A global event loop (singleton) based on `requestAnimationFrame` provides assistance with event throttling via its custom `tick` event. Cartesian coordinates preferred in scientific and engineering applications are supported. *Pan*, *drag* and *zoom* based on user defined origins can be done.

<br>

## References 

<span id="1">[1] Canvas API, [https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)</span>    
<span id="2">[2] 
D. Corbacho, Debouncing and Throttling Explained Through Examples    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[https://css-tricks.com/debouncing-throttling-explained-examples/](https://css-tricks.com/debouncing-throttling-explained-examples/)</span>    
 <span id="3">[3] S. Goessner, <code>canvas-area</code>, 
 [https://github.com/goessner/canvas-area](https://github.com/goessner/canvas-area)</span>    
 <span id="4">[4] S. Goessner, Make your HTML canvas Interactive, 
 [Researchgate, DOI: 10.13140/RG.2.2.31978.39367](https://www.researchgate.net/publication/360034117_Make_your_HTML_canvas_Interactive)</span>    
