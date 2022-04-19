# `canvasInteractor`

## What is it ... ?

`canvasInteractor` is a micro-library (9.1 kB uncompressed) used to handle pointer events for simple geometry editing together with one or more HTML canvases.
It implements a global event loop based on `requestAnimationFrame` and supports throttling of `pointermove` and `wheel` events via its custom `tick` event for efficient animation. Cartesian coordinates with user defined origin are possible, it allows *zoom*, *pan* and *drag* interactions.

`canvasInteractor` is the modern and more minimal successor of deprecated [canvas-area](https://github.com/goessner/canvas-area).  
It was primarily implemented for use by my students for web-kinematics projects.

## Read more about it ... example please !

[https://goessner.github.io/canvasInteractor](https://goessner.github.io/canvasInteractor)


## CDN

Use a local instance or following link for `canvasInteractor.js`.
* `https://cdn.jsdelivr.net/gh/goessner/canvasInteractor/canvasInteractor.js`


## FAQ

* __Does not work properly with Mobile Device X and Touch Screen Y ?__
  * Desktop browsers only are addressed primarily.
  * Implementation of touch events is experimental (*pan* works with touch).

* __Can you implement feature X and possibly feature Y ?__
  * `canvasInteractor` serves my personal needs very well as it is.
  * So ... no, I won't.
  * Please go ahead and implement it by yourself.
  * If you think, your enhancement is of common interest, you are very welcome, to send me a pull request.

## Changelog

###  [1.0.0] on April 19, 2018
* Complete rewrite.

## License

`canvasInteractor` is licensed under the [MIT License](http://opensource.org/licenses/MIT)

 © [Stefan Gössner](https://github.com/goessner)
