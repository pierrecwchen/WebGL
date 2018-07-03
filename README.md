# WebGL

Here consists of two things: [presentation slide](WebGL_slides.pdf) made by me and other two teammates and [the html code](WebGL_example.html) I made for the presentation. Things are created for Graphic class in 2011 with notepad under windows 7.


<html><head><title>The example of WebGL</title>
<script id="vertexShader" type="x-shader/x-vertex">
attribute vec4 vPosition;
attribute vec4 vColor;
varying vec4 color;
uniform vec3 theta;
void main(){
vec3 angles = radians(theta);
vec3 c = cos(angles);
vec3 s = sin(angles);
mat4 rx = mat4( 1.0, 0.0, 0.0, 0.0,
0.0, c.x, -s.x, 0.0,
0.0, s.x, c.x, 0.0,
0.0, 0.0, 0.0, 1.0);
mat4 ry = mat4( c.y, 0.0, s.y, 0.0,
0.0, 1.0, 0.0, 0.0,
-s.y, 0.0, c.y, 0.0,
0.0, 0.0, 0.0, 1.0);
mat4 rz = mat4( c.z, -s.z, 0.0, 0.0,
s.z, c.z, 0.0, 0.0,
0.0, 0.0, 1.0, 0.0,
0.0, 0.0, 0.0, 1.0);
gl_Position= rx*ry*rz*vPosition;
color=vColor;
}
</script>

<script id="fragmentShader" type="x-shader/x-fragment">
#ifdef GL_ES
precision highp float;
#endif
varying vec4 color;
void main(){
gl_FragColor=color;
}
</script>

<script type="text/javascript">
var gl=null;
function initGL(canvas,opt_attribs) {
if(!window.WebGLRenderingContext){
alert("Your browser cannot support webGL");
}
else{
var names = ["webgl", "experimental-webgl", "webkit-3d", "moz-webgl"];
for (var ii = 0; ii < names.length; ++ii) {
try {
gl = canvas.getContext(names[ii], opt_attribs);
} catch(e) {}
if (gl) {
break;
}
}
}
gl.viewportWidth = canvas.width;
gl.viewportHeight = canvas.height;
}
function loadShader(shaderId) { 
var shaderScript = document.getElementById(shaderId);
if (!shaderScript) { 
log("*** Error: shader script '"+shaderId+"' not found"); 
return null; 
}

// Create the shader object
var shader;
if (shaderScript.type == "x-shader/x-vertex") 
shader = gl.createShader(gl.VERTEX_SHADER); 
else if (shaderScript.type == "x-shader/x-fragment") 
shader = gl.createShader(gl.FRAGMENT_SHADER); 
else {
log("*** Error: shader script '"+shaderId+"' of undefined type '"+shaderScript.type+"'"); 
return null;
}
// Load the shader source
gl.shaderSource(shader, shaderScript.text);

// Compile the shader 
gl.compileShader(shader);
var compiled = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
if (!compiled && !gl.isContextLost()) {
alert(gl.getShaderInfoLog(shader));
// Something went wrong during compilation; get the error
var error = gl.getShaderInfoLog(shader);
log("*** Error compiling shader '"+shaderId+"':"+error);
gl.deleteShader(shader);
return null;
}
return shader; 
} 
var program;
function initShaders(){
var vertexShader = loadShader("vertexShader");
var fragmentShader = loadShader("fragmentShader");
program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);

if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
alert("Could not initialise shaders");
}
}
var vertexBuffer;
var colorBuffer;
var thetaUniformLocation;
var xCood;
var yCood;
var zCood;
var Theta;
var requestId;
var ToChange
function init(){
vertexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
var vertices = [  
// Front face  
-0.5, -0.5,  0.5,  
0.5, -0.5,  0.5,  
0.5,  0.5,  0.5,
-0.5, -0.5,  0.5,
0.5,  0.5,  0.5,  
-0.5,  0.5,  0.5,  

// Back face  
-0.5, -0.5, -0.5,  
-0.5,  0.5, -0.5,  
0.5,  0.5, -0.5,
-0.5, -0.5, -0.5,
0.5,  0.5, -0.5,  
0.5, -0.5, -0.5,  

// Top face  
-0.5,  0.5, -0.5,  
-0.5,  0.5,  0.5,  
0.5,  0.5,  0.5, 
-0.5,  0.5, -0.5,   
0.5,  0.5,  0.5,  
0.5,  0.5, -0.5,  

// Bottom face  
-0.5, -0.5, -0.5,  
0.5, -0.5, -0.5,  
0.5, -0.5,  0.5,
-0.5, -0.5, -0.5,  
0.5, -0.5,  0.5,  
-0.5, -0.5,  0.5,  

// Right face  
0.5, -0.5, -0.5,  
0.5,  0.5, -0.5,  
0.5,  0.5,  0.5, 
0.5, -0.5, -0.5,  
0.5,  0.5,  0.5,  
0.5, -0.5,  0.5,  

// Left face  
-0.5, -0.5, -0.5,  
-0.5, -0.5,  0.5,  
-0.5,  0.5,  0.5,
-0.5, -0.5, -0.5, 
-0.5,  0.5,  0.5,  
-0.5,  0.5, -0.5  
];

gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
var vertexPositionVariableIndex = gl.getAttribLocation(program, "vPosition");
gl.enableVertexAttribArray(vertexPositionVariableIndex);
gl.vertexAttribPointer(vertexPositionVariableIndex, 3, gl.FLOAT, false, 0, 0);
colorBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);

var colors = [  
1.0,  1.0,  1.0,  1.0,    // Front face: white
1.0,  1.0,  1.0,  1.0,
1.0,  1.0,  1.0,  1.0,
1.0,  1.0,  1.0,  1.0,
1.0,  1.0,  1.0,  1.0,
1.0,  1.0,  1.0,  1.0,  
1.0,  0.0,  0.0,  1.0,    // Back face: red
1.0,  0.0,  0.0,  1.0,
1.0,  0.0,  0.0,  1.0,
1.0,  0.0,  0.0,  1.0,
1.0,  0.0,  0.0,  1.0,
1.0,  0.0,  0.0,  1.0,  
0.0,  1.0,  0.0,  1.0,    // Top face: green
0.0,  1.0,  0.0,  1.0,
0.0,  1.0,  0.0,  1.0,
0.0,  1.0,  0.0,  1.0,
0.0,  1.0,  0.0,  1.0,
0.0,  1.0,  0.0,  1.0,  
0.0,  0.0,  1.0,  1.0,    // Bottom face: blue
0.0,  0.0,  1.0,  1.0,
0.0,  0.0,  1.0,  1.0,
0.0,  0.0,  1.0,  1.0,
0.0,  0.0,  1.0,  1.0,
0.0,  0.0,  1.0,  1.0,  
1.0,  1.0,  0.0,  1.0,    // Right face: yellow
1.0,  1.0,  0.0,  1.0,
1.0,  1.0,  0.0,  1.0,
1.0,  1.0,  0.0,  1.0,
1.0,  1.0,  0.0,  1.0,
1.0,  1.0,  0.0,  1.0,  
1.0,  0.0,  1.0,  1.0,     // Left face: purple
1.0,  0.0,  1.0,  1.0,
1.0,  0.0,  1.0,  1.0,
1.0,  0.0,  1.0,  1.0,
1.0,  0.0,  1.0,  1.0,
1.0,  0.0,  1.0,  1.0,  
];

gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors), gl.STATIC_DRAW);
var colorPositionVariableIndex = gl.getAttribLocation(program, "vColor");
gl.enableVertexAttribArray(colorPositionVariableIndex);
gl.vertexAttribPointer(colorPositionVariableIndex, 4, gl.FLOAT, false, 0, 0);
gl.useProgram(program);
thetaUniformLocation = gl.getUniformLocation(program, "theta");

xCood = 0.0;
yCood = 0.0;
zCood = 0.0;
Theta = [xCood, yCood, zCood];
ToChange=true;
gl.enable(gl.DEPTH_TEST);
gl.clearColor(0.0, 0.0, 0.0, 1.0);
}
window.requestAnimFrame=(function(){
return window.requestAnimationFrame||
window.webkitRequestAnimationFrame||
window.mozRequestAnimationFrame||
window.oRequestAnimationFrame||
window.msRequestAnimationFrame||
function(callback){
return window.setTimeout(callback,1000/60);
};
})();

window.cancelRequestAnimFrame=(function(){
return window.cancelCancelRequestAnimationFrame||
window.webkitCancelRequestAnimationFrame||
window.mozCancelRequestAnimationFrame||
window.oCancelRequestAnimationFrame||
window.msCancelRequestAnimationFrame||
window.clearTimeout;
})();
function display(){
gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 36);
gl.uniform3fv(thetaUniformLocation, Theta);
if(ToChange){
xCood += 2.0;
yCood += 5.0;
zCood += 0.5;
}
Theta = [xCood, yCood, zCood];
gl.flush();
}
function webGLStart(){
var canvas = document.getElementById("Example");
initGL(canvas);
initShaders();
init();
//setInterval(display,10);
canvas.addEventListener('webglcontextlost', handleContextLost, false);
canvas.addEventListener('webglcontextrestored', handleContextRestored, false);
var drawLoop=function(){
display();
requestId=window.requestAnimFrame(drawLoop,canvas);
}
drawLoop();
function handleContextLost(e){
e.preventDefault();
if(requetId!==undefined){
window.cancelRequestAnimFrame(requestId);
requestId=undefined;
}
}
function handleContextRestored(){
initGL(canvas);
initShaders();
init();
drawLoop();
}
function click(){
ToChange=!ToChange;
}
canvas.onclick=click;
}
</script>

</head>

<body onload="webGLStart();">
<canvas id="Example" style="border: none;" width="500" height="500"></canvas>
</body>
</html>
