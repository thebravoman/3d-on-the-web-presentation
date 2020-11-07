# 3D on the web. WebGL, Babylon JS and Three JS

Kiril Mitov (kmitov [at] axlessoft [dot] com), CTO Axlessoft, October 2020 

Get in touch or follow me:

[thebravoman (twitter)](https://twitter.com/thebravoman)

[kirilmitov (linkedin)](https://www.linkedin.com/in/kirilmitov/)

[kmitov.com (blog)](https://www.kmitov.com)

# Quote to begin with 

> "If the human body was making a new organ every week, doctors would be googling this shit."

(On the state of the JavaScript community; author Unknown; date: beginning of 21 century)

<!-- MarkdownTOC autoanchor='true' autolink='true' -->

- [Example of what we can do we 3D on the web](#example-of-what-we-can-do-we-3d-on-the-web)
    - [First example for an interesting model](#first-example-for-an-interesting-model)
    - [Second example for an interesting model](#second-example-for-an-interesting-model)
- [Organization Context](#organization-context)
- [Personal context](#personal-context)
- [Drawing on the screen](#drawing-on-the-screen)
    - [Canvas2D](#canvas2d)
    - [WebGL](#webgl)
        - [WebGL in 10 steps.](#webgl-in-10-steps)
        - [Why the strange types?](#why-the-strange-types)
        - [Where is the drawRect\(x,y,width,height\)?](#where-is-the-drawrectxywidthheight)
        - [WebGL in one paragraph.](#webgl-in-one-paragraph)
    - [WebGL vs Canvas2D](#webgl-vs-canvas2d)
    - [Three JS](#three-js)
        - [Three JS Example](#three-js-example)
        - [Three JS Status](#three-js-status)
        - [Why we choose it](#why-we-choose-it)
        - [Where it failed us](#where-it-failed-us)
    - [BABYLON JS](#babylon-js)
        - [The playground](#the-playground)
        - [Sandbox](#sandbox)
        - [Why we switched.](#why-we-switched)
        - [Why we stayed.](#why-we-stayed)
        - [Notable examples](#notable-examples)

<!-- /MarkdownTOC -->

<a id="example-of-what-we-can-do-we-3d-on-the-web"></a>
# Example of what we can do we 3D on the web

<a id="first-example-for-an-interesting-model"></a>
## First example for an interesting model

https://sketchfab.com/3d-models/battle-damaged-sci-fi-helmet-pbr-b81008d513954189a063ff901f7abfe4

<a id="second-example-for-an-interesting-model"></a>
## Second example for an interesting model 

https://platform.buildin3d.com/instructions/2-axlessoft-cooler

<a id="organization-context"></a>
# Organization Context

We used WebGL, BABYLON JS, Three JS in the development of the Instructions Steps (IS) framework. IS helped us build buildin3d.com and visualize 3D assembly instructions to end clients. IS has an event-driven plug-in architecture. It consists of 68 plugins. Some plugings use Three.js and others use Babylon JS.

We have started with Three JS until we reached the point were three js was becoming impossible to work with and then we switched to BABYLON JS. We do not use WebGL directly as this would be an overkill of us and our value is added by providing a framework on top be Babylon JS and Three JS

```
Instruction Steps (IS)

    Babylon JS, JS Modeler, Three JS
        
        WebGL
```

<a id="personal-context"></a>
# Personal context

I've always have interest in computer graphics. Not specifically games and I've never been a game developer, but I once tried to build a CAD system on my own. So I have some understanding about rotation, scale, tranfromations, about modeling about delivering graphics. What I did not like was that we have JavaScript but I learned to like it in the last few years and as we joke in the office - "I've become the best JavaScript developer I know of"


<a id="drawing-on-the-screen"></a>
# Drawing on the screen

If you have an API to set the color of a pixel on the screen you can draw anything. 

The question is how to do it easy and fast. 

It all starts with the canvas. HTML 5 gives us the canvas element

```html
<canvas id="km" width="640" height="360" style="border: 1px solid black;"></canvas>
```

<a id="canvas2d"></a>
## Canvas2D

```html
<!DOCTYPE html>
<html>
 <head>
  <meta charset="utf-8"/>
  <script type="application/javascript">
    function draw() {
      var canvas = document.getElementById('km');
      if (canvas.getContext) {
        var ctx = canvas.getContext('2d');

        ctx.fillStyle = 'rgb(255, 0, 0)';
        ctx.fillRect(10, 10, 100, 100);

        ctx.fillStyle = 'rgba(0, 255, 0, 0.5)';
        ctx.fillRect(80, 80, 200, 200);
      } else {
        console.log("Not supported")
      }
    }
  </script>
 </head>
 <body onload="draw();">
   <canvas id="km" width="640" height="360" style="border: 1px solid black;"></canvas>
 </body>
</html>
```

Canvas2D has API methods like

```
ctx.lineWidth = 10;

ctx.strokeRect(x1, y1, x2, y2);

ctx.fillRect(x, y, width, height);

ctx.beginPath();
ctx.moveTo(x1, y1);
ctx.lineTo(..., ...);
ctx.lineTo(..., ...);
ctx.closePath();
ctx.stroke();
```


Had more support initially. Has some hardware acceleration.

Here is an example - http://fhtr.org/gravityring/sprites.html

<a id="webgl"></a>
## WebGL

> WebGL (Web Graphics Library) is a JavaScript API for rendering high-performance interactive 3D and 2D graphics within any compatible web browser without the use of plug-ins. WebGL does so by introducing an API that closely conforms to OpenGL ES 2.0 that can be used in HTML5 <canvas> elements. This conformance makes it possible for the API to take advantage of hardware graphics acceleration provided by the user's device.
> 
    (https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)

In reality WebGL is just a rasterization engine. It draws points, lines, and triangles based on code you supply. It is misleadingly described as "3D engine". It figures out which pixels the 3 points of the triangle corresponds to, and then rasterizes the triangle which is a fancy word for “draws it with pixels”

Follows many of OpenGL ideas an concepts. Which is a world competely different from the JavaScript world.

```html
<canvas id="c"></canvas>
  <script  id="vertex-shader-2d" type="notjs">

  // an attribute will receive data from a buffer
  attribute vec4 a_position;

  // all shaders have a main function
  void main() {

    // gl_Position is a special variable a vertex shader
    // is responsible for setting
    gl_Position = a_position;
  }

</script>
<script  id="fragment-shader-2d" type="notjs">

  // fragment shaders don't have a default precision so we need
  // to pick one. mediump is a good default
  precision mediump float;

  void main() {
    // gl_FragColor is a special variable a fragment shader
    // is responsible for setting
    gl_FragColor = vec4(1, 0, 0.5, 1); // return redish-purple
  }

</script><!--
for most samples webgl-utils only provides shader compiling/linking and
canvas resizing because why clutter the examples with code that's the same in every sample.
See https://webglfundamentals.org/webgl/lessons/webgl-boilerplate.html
and https://webglfundamentals.org/webgl/lessons/webgl-resizing-the-canvas.html
for webgl-utils, m3, m4, and webgl-lessons-ui.
-->
<script src="https://webglfundamentals.org/webgl/resources/webgl-utils.js"></script>
<script type='application/javascript'>
// WebGL - Fundamentals
// from https://webglfundamentals.org/webgl/webgl-fundamentals.html


/* eslint no-console:0 consistent-return:0 */
"use strict";

function createShader(gl, type, source) {
  var shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
  if (success) {
    return shader;
  }

  console.log(gl.getShaderInfoLog(shader));
  gl.deleteShader(shader);
}

function createProgram(gl, vertexShader, fragmentShader) {
  var program = gl.createProgram();
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  gl.linkProgram(program);
  var success = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (success) {
    return program;
  }

  console.log(gl.getProgramInfoLog(program));
  gl.deleteProgram(program);
}

function main() {
  // Get A WebGL context
  var canvas = document.querySelector("#c");
  var gl = canvas.getContext("webgl");
  if (!gl) {
    return;
  }

  // Get the strings for our GLSL shaders
  var vertexShaderSource = document.querySelector("#vertex-shader-2d").text;
  var fragmentShaderSource = document.querySelector("#fragment-shader-2d").text;

  // create GLSL shaders, upload the GLSL source, compile the shaders
  var vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
  var fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);

  // Link the two shaders into a program
  var program = createProgram(gl, vertexShader, fragmentShader);

  // look up where the vertex data needs to go.
  var positionAttributeLocation = gl.getAttribLocation(program, "a_position");

  // Create a buffer and put three 2d clip space points in it
  var positionBuffer = gl.createBuffer();

  // Bind it to ARRAY_BUFFER (think of it as ARRAY_BUFFER = positionBuffer)
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

  var positions = [
    0, 0,
    0, 0.5,
    0.7, 0,
  ];
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);

  // code above this line is initialization code.
  // code below this line is rendering code.

  webglUtils.resizeCanvasToDisplaySize(gl.canvas);

  // Tell WebGL how to convert from clip space to pixels
  gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

  // Clear the canvas
  gl.clearColor(0, 0, 0, 0);
  gl.clear(gl.COLOR_BUFFER_BIT);

  // Tell it to use our program (pair of shaders)
  gl.useProgram(program);

  // Turn on the attribute
  gl.enableVertexAttribArray(positionAttributeLocation);

  // Bind the position buffer.
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

  // Tell the attribute how to get data out of positionBuffer (ARRAY_BUFFER)
  var size = 2;          // 2 components per iteration
  var type = gl.FLOAT;   // the data is 32bit floats
  var normalize = false; // don't normalize the data
  var stride = 0;        // 0 = move forward size * sizeof(type) each iteration to get the next position
  var offset = 0;        // start at the beginning of the buffer
  gl.vertexAttribPointer(
      positionAttributeLocation, size, type, normalize, stride, offset);

  // draw
  var primitiveType = gl.TRIANGLES;
  var offset = 0;
  var count = 3;
  gl.drawArrays(primitiveType, offset, count);
}

main();

</script>
```

<a id="webgl-in-10-steps"></a>
### WebGL in 10 steps. 

1. WebGL uses GL Shader Language (GLSL). 
2. It is strictly type C/C++ like language
3. You develop two functions in GL Shader Language that are called 'shaders'.
4. For WebGl you should develop two shaders - *vertex shader* and *fragmet shader*.
5. What is a 'vertex' is described right here - https://www.mathsisfun.com/geometry/vertices-faces-edges.html. A vertex is like a "reference to a point".
6. The vertex shader is a function. As a result the function should return a position. So it could take information about a 3D object in and its Vertices in a 3D space and return where these Vertices should be positioned on the screen. 

```c
  // an attribute will receive data from a buffer
  attribute vec4 a_position;

  // all shaders have a main function
  void main() {

    // gl_Position is a special variable a vertex shader
    // is responsible for setting
    gl_Position = a_position;
  }
```

7. The second shader for WebGL is *fragment shader*. Fragment shader is again a function an it should compute the color of each pixel.

```c
  void main() {
    // gl_FragColor is a special variable a fragment shader
    // is responsible for setting
    gl_FragColor = vec4(1, 0, 0.5, 1); // return redish-purple
  }
```

8. You set up both shaders in a program. This is done with
```javascript
function createShader(gl, type, source) {
  var shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
  if (success) {
    return shader;
  }

  console.log(gl.getShaderInfoLog(shader));
  gl.deleteShader(shader);
}

function createProgram(gl, vertexShader, fragmentShader) {
  var program = gl.createProgram();
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  gl.linkProgram(program);
  var success = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (success) {
    return program;
  }

  console.log(gl.getProgramInfoLog(program));
  gl.deleteProgram(program);
}
```

```javascript
 // Get A WebGL context
  var canvas = document.querySelector("#c");
  var gl = canvas.getContext("webgl");
  if (!gl) {
    return;
  }

  // Get the strings for our GLSL shaders
  var vertexShaderSource = document.querySelector("#vertex-shader-2d").text;
  var fragmentShaderSource = document.querySelector("#fragment-shader-2d").text;

  // create GLSL shaders, upload the GLSL source, compile the shaders
  var vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
  var fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);

  // Link the two shaders into a program
  var program = createProgram(gl, vertexShader, fragmentShader);

  // look up where the vertex data needs to go.
  var positionAttributeLocation = gl.getAttribLocation(program, "a_position");
```

9. After we've setup the shaders we setup the state which includes buffers and some strage Float32Types and other things

```javascript
var positions = [
0, 0,
0, 0.5,
0.7, 0,
];
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
```

10. After we've set up the shaders and the state. We can now start reading from the buffers. We call one of two main methods *gl.drawArrays* or *gl.drawElements*. When these methods are called, they will call our shaders which will take information from the buffers and should set gl_Position (vertex shader) or gl_FragColor (fragmet shader)

```javascript
// draw
var primitiveType = gl.TRIANGLES;
var offset = 0;
var count = 3;
gl.drawArrays(primitiveType, offset, count);
```

That's it. WebGL in 10 steps.

The whole WebGL API is about setup of the buffers and calling gl.drawArrays or gl.drawElements.

The full WebGL API is located at - https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext

<a id="why-the-strange-types"></a>
### Why the strange types?

The more specific the type, the faster you can work with this data - eg. Float32Array. 

GLSL also has strict types. vec2, vec3, and vec4 which represent 2 values, 3 values, and 4 values respectively. Similarly it has mat2, mat3 and mat4 which represent 2x2, 3x3, and 4x4 matrices.

<a id="where-is-the-drawrectxywidthheight"></a>
### Where is the drawRect(x,y,width,height)?

Gone. WebGL is more "low-level". You can implement drawRect yourself if you want to. 

<a id="webgl-in-one-paragraph"></a>
### WebGL in one paragraph.
You user GL Shader Language to define two functions called shaders. One shader is called 'vertex shader' and calculates positions for different vertices. The second shader is called 'fragment shader' and calculates color for a pixes. You push your data to buffers. You call gl.drawArrays which reads the data from the buffers and calls the shaders. Things are drawn on the screen. 

<a id="webgl-vs-canvas2d"></a>
## WebGL vs Canvas2D

Canvas2D - generally preferred for 2D
WebGL - no problem with 2D, but generally preferred for 3D.
WebGL has access to native 3D API and is generally faster than Canvas2D, but depends could depend on how the browser implements it and what the task is.
WebGL provides more low lever control.

>There's not much they (browsers) can do in the middle to mess up WebGL.
(gman at stackoverflow.com)

<a id="three-js"></a>
## Three JS

>The aim of the project is to create an easy to use, lightweight, 3D library with a default WebGL renderer. The library also provides Canvas 2D, SVG and CSS3D renderers in the examples.

Why - because you can spend your time more productively than developing shaders in WebGL. Most of the shaders are already developed. Why not use a few. You would also like to work with a fewer higher abstractions.

Why 2 - Because at the end of the day you need to say:

```javascript
newScene()
createNewFancyBox()
while(userHasNotClicked()) {
    moveBoxALitteToTheLeft();
}
stopMovingBox();
```

You are not working that much with the low-lever shaders and types of WebGl or the low level methods of Canvas2D. 

<a id="three-js-example"></a>
### Three JS Example

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Animated object</title>
    <meta charset="utf-8">
    <meta content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0" name="viewport">
    <link type="text/css" rel="stylesheet" href="main.css">
  </head>
  <body>
    <div id="container"></div>

    <script type="module">

      import * as THREE from 'https://unpkg.com/three/build/three.module.js'

      let camera, scene, renderer;
      let geometry, material, mesh;

      init();
      animate();

      function init() {

          camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 10 );
          camera.position.z = 1;

          scene = new THREE.Scene();

          geometry = new THREE.BoxGeometry( 0.2, 0.2, 0.2 );
          material = new THREE.MeshNormalMaterial();

          mesh = new THREE.Mesh( geometry, material );
          scene.add( mesh );

          renderer = new THREE.WebGLRenderer( { antialias: true } );
          renderer.setSize( window.innerWidth, window.innerHeight );
          document.body.appendChild( renderer.domElement );

      }

      function animate() {

          requestAnimationFrame( animate );

          mesh.rotation.x += 0.01;
          mesh.rotation.y += 0.02;

          renderer.render( scene, camera );

      }
    </script>

  </body>

</html>

```


<a id="three-js-status"></a>
### Three JS Status

1. Current release is r122
2. A new version at the end of every month.
3. A large community

<a id="why-we-choose-it"></a>
### Why we choose it 

1. File format - there was a library called JSModeler. The author of the library has made the choice to work with Three JS and has implement a few thousands line of code to support .OBJ files out of the box. So we started with Three JS. Somebody else has made the choice for us.

<a id="where-it-failed-us"></a>
### Where it failed us

1. File Formats
2. Support - you are lost
3. Animations from file - they simply did not work out of the box.
4. It was difficult to use

<a id="babylon-js"></a>
## BABYLON JS

> Our mission is to create one of the most powerful, beautiful, and simple Web rendering engines in the world. Our passion is to make it completely open and free for everyone. Up to 3 times smaller and 12% faster, Babylon.js 4.1 includes countless performance optimizations, continuing the lineage of a high-performance engine. With the new Node Material Editor, a truly cross-platform development experience with Babylon Native, Cascaded Shadows, Navigation Mesh, updated WebXR and glTF support, and much much more, Babylon.js 4.1 brings even more power to your web development toolbox.
> 

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

        <title>Babylon.js sample code</title>

        <!-- Babylon.js -->
        <script src="https://code.jquery.com/pep/0.4.2/pep.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/dat-gui/0.6.2/dat.gui.min.js"></script>
        <script src="https://preview.babylonjs.com/ammo.js"></script>
        <script src="https://preview.babylonjs.com/cannon.js"></script>
        <script src="https://preview.babylonjs.com/Oimo.js"></script>
        <script src="https://preview.babylonjs.com/earcut.min.js"></script>
        <script src="https://preview.babylonjs.com/babylon.js"></script>
        <script src="https://preview.babylonjs.com/materialsLibrary/babylonjs.materials.min.js"></script>
        <script src="https://preview.babylonjs.com/proceduralTexturesLibrary/babylonjs.proceduralTextures.min.js"></script>
        <script src="https://preview.babylonjs.com/postProcessesLibrary/babylonjs.postProcess.min.js"></script>
        <script src="https://preview.babylonjs.com/loaders/babylonjs.loaders.js"></script>
        <script src="https://preview.babylonjs.com/serializers/babylonjs.serializers.min.js"></script>
        <script src="https://preview.babylonjs.com/gui/babylon.gui.min.js"></script>
        <script src="https://preview.babylonjs.com/inspector/babylon.inspector.bundle.js"></script>

        <style>
            html, body {
                overflow: hidden;
                width: 100%;
                height: 100%;
                margin: 0;
                padding: 0;
            }

            #renderCanvas {
                width: 100%;
                height: 100%;
                touch-action: none;
            }
        </style>
    </head>
<body>
    <canvas id="renderCanvas"></canvas>
    <script>
        var canvas = document.getElementById("renderCanvas");

        var engine = null;
        var scene = null;
        var sceneToRender = null;
        var createDefaultEngine = function() { return new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true }); };
        var createScene = function () {
            var scene = new BABYLON.Scene(engine);
        
            var light = new BABYLON.PointLight("Omni", new BABYLON.Vector3(0, 100, 100), scene);
            var camera = new BABYLON.ArcRotateCamera("Camera", 0, 0.8, 100, new BABYLON.Vector3.Zero(), scene);
            camera.attachControl(canvas, true);
        
            //Boxes
            var box1 = BABYLON.Mesh.CreateBox("Box1", 10.0, scene);
            box1.position.x = -20;
            var box2 = BABYLON.Mesh.CreateBox("Box2", 10.0, scene);
        
            var materialBox = new BABYLON.StandardMaterial("texture1", scene);
            materialBox.diffuseColor = new BABYLON.Color3(0, 1, 0);//Green
            var materialBox2 = new BABYLON.StandardMaterial("texture2", scene);
        
            //Applying materials
            box1.material = materialBox;
            box2.material = materialBox2;
        
            //Positioning box
            box2.position.x = 20;
        
            // Creation of a basic animation with box 1
            //----------------------------------------
        
            //Create a scaling animation at 30 FPS
            var animationBox = new BABYLON.Animation("tutoAnimation", "scaling.x", 30, BABYLON.Animation.ANIMATIONTYPE_FLOAT,BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE);
            //Here we have chosen a loop mode, but you can change to :
            //  Use previous values and increment it (BABYLON.Animation.ANIMATIONLOOPMODE_RELATIVE)
            //  Restart from initial value (BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE)
            //  Keep the final value (BABYLON.Animation.ANIMATIONLOOPMODE_CONSTANT)
        
            // Animation keys
            var keys = [];
            //At the animation key 0, the value of scaling is "1"
            keys.push({
                frame: 0,
                value: 1
            });
        
            //At the animation key 20, the value of scaling is "0.2"
            keys.push({
                frame: 20,
                value: 0.2
            });
        
            //At the animation key 100, the value of scaling is "1"
            keys.push({
                frame: 100,
                value: 1
            });
        
            //Adding keys to the animation object
            animationBox.setKeys(keys);
        
            //Then add the animation object to box1
            box1.animations.push(animationBox);
        
            //Finally, launch animations on box1, from key 0 to key 100 with loop activated
            scene.beginAnimation(box1, 0, 100, true);
        
            // Creation of a manual animation with box 2
            //------------------------------------------
            setInterval(function () {
        
                //The color is defined at run time with random()
                box2.material.diffuseColor = new BABYLON.Color3(Math.random(), Math.random(), Math.random());
        
            }, 1000);
        
            return scene;
        }
    var engine;
    try {
    engine = createDefaultEngine();
    } catch(e) {
    console.log("the available createEngine function failed. Creating the default engine instead");
    engine = createDefaultEngine();
    }
        if (!engine) throw 'engine should not be null.';
        scene = createScene();;
        sceneToRender = scene

        engine.runRenderLoop(function () {
            if (sceneToRender && sceneToRender.activeCamera) {
                sceneToRender.render();
            }
        });

        // Resize
        window.addEventListener("resize", function () {
            engine.resize();
        });
    </script>
</body>
</html>

```

<a id="the-playground"></a>
### The playground

Quite easy to try something and to share with colleague or community if you have a question or a problem.

https://playground.babylonjs.com/

<a id="sandbox"></a>
### Sandbox 
Very easy to try different files and if they could load.

https://sandbox.babylonjs.com/

<a id="why-we-switched"></a>
### Why we switched.

1. It supported our case out of the box.

<a id="why-we-stayed"></a>
### Why we stayed. 
1. Fastest community support in the world. I've personally commited in Eclipse and Apache foundations and I've also developed and worked with a lot of open source sotware. I've never had the case to have a solution in a number of hours almost every time. 

2. I've seen this in the rails community where I once reported a bug in the morning and a fix was released in the afternoon. With BABYLON I was reported somethng I need and an API was introduced a few days after that and released

3. The API feels more powerfull and richer. But it also feels more enterprise.

<a id="notable-examples"></a>
### Notable examples

https://playground.babylonjs.com/#BCU1XR#0
https://playground.babylonjs.com/#UZ23UH#0
https://playground.babylonjs.com/#ZU5TKG#0
https://playground.babylonjs.com/#8MGKWK#344
https://playground.babylonjs.com/#YB006J#75
https://playground.babylonjs.com/#6UZDJ9#0
https://playground.babylonjs.com/#7149G4#0
https://playground.babylonjs.com/#6ZVKE3#0
https://playground.babylonjs.com/#J0D279#0
