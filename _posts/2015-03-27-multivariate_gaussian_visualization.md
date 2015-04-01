---
layout: post
title: Multivariate Gaussian Distribution Visualization
tags: math visualization machinelearning gaussian
---

Multivariate Gaussian Distribution or Multivariate Normal Distribution visualization.

# Formula
$$
\mathbf{\mu} = \begin{bmatrix}
\mu_1\\
\mu_2
\end{bmatrix}\\
\mathbf{\Sigma}  = \begin{bmatrix}
\sigma_{1,1} & \sigma_{1,2}\\ 
\sigma_{2,1} & \sigma_{2,2}
\end{bmatrix}\\
f_x(x_1,x_2) = \frac{1}{\sqrt{(2\pi)^2\begin{vmatrix}
\Sigma
\end{vmatrix}}}\exp{(-0.5(\mathbf{x}-\mathbf{\mu})^T\Sigma^{-1}(\mathbf{x}-\mathbf{\mu}))}
$$

# Playground

<div>
<script type="text/javascript" src='/assets/3dplot/SurfacePlot.js'></script>
<script type="text/javascript" src='/assets/3dplot/ColourGradient.js'></script>
<script type="text/javascript" src="/assets/3dplot/glMatrix-0.9.5.min.js"></script>
<script type="text/javascript" src="/assets/3dplot/webgl-utils.js"></script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/1.4.0/math.min.js"></script>
<script id="shader-fs" type="x-shader/x-fragment">
    #ifdef GL_ES
    #endif
    precision mediump float;
    varying vec4 vColor;
    varying vec3 vLightWeighting;
    void main(void)
    {
    gl_FragColor = vec4(vColor.rgb * vLightWeighting, vColor.a);
    }
  </script>
  <script id="shader-vs" type="x-shader/x-vertex">
    attribute vec3 aVertexPosition;
    attribute vec3 aVertexNormal;
    attribute vec4 aVertexColor;
    uniform mat4 uMVMatrix;
    uniform mat4 uPMatrix;
    uniform mat3 uNMatrix;
    varying vec4 vColor;
    uniform vec3 uAmbientColor;
    uniform vec3 uLightingDirection;
    uniform vec3 uDirectionalColor;
    varying vec3 vLightWeighting;
    void main(void)
    {
    gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
    vec3 transformedNormal = uNMatrix * aVertexNormal;
    float directionalLightWeighting = max(dot(transformedNormal, uLightingDirection), 0.0);
    vLightWeighting = uAmbientColor + uDirectionalColor * directionalLightWeighting; 
    vColor = aVertexColor;
    }
  </script>
  <script id="axes-shader-fs" type="x-shader/x-fragment">
    precision mediump float;
    varying vec4 vColor;
    void main(void)
    {
    gl_FragColor = vColor;
    }
  </script>
  <script id="axes-shader-vs" type="x-shader/x-vertex">
    attribute vec3 aVertexPosition;
    attribute vec4 aVertexColor;
    uniform mat4 uMVMatrix;
    uniform mat4 uPMatrix;
    varying vec4 vColor;
    uniform vec3 uAxesColour;
    void main(void)
    {
    gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
    vColor =  vec4(uAxesColour, 1.0);
    } 
  </script>
  <script id="texture-shader-fs" type="x-shader/x-fragment">
    #ifdef GL_ES
    #endif
    precision mediump float;
    varying vec2 vTextureCoord;
    uniform sampler2D uSampler;
    void main(void)
    {
    gl_FragColor = texture2D(uSampler, vTextureCoord);
    }
  </script>
  <script id="texture-shader-vs" type="x-shader/x-vertex">
    attribute vec3 aVertexPosition;
    attribute vec2 aTextureCoord;
    varying vec2 vTextureCoord;
    uniform mat4 uMVMatrix;
    uniform mat4 uPMatrix;
    void main(void)
    {
    gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
    vTextureCoord = aTextureCoord; 
    }
  </script>
<style>
#paper_slider {
}
#brackets {
font-size: 90px;
font-weight: 100;
}
</style>
<div style="width: 100%">
<table>
  <tr>
    <td rowspan="4">$$\mu =$$</td>
    <td rowspan="4"><p id='brackets'>[</p></td>
    <td>&nbsp;</td>
    <td rowspan="4"><p id='brackets'>]</p></td>
    <td rowspan="4">$$=$$</td>
    <td rowspan="4"><p id='brackets'>[</p></td>
    <td>&nbsp;</td>
    <td rowspan="4"><p id='brackets'>]</p></td>
  </tr>
  <tr>
    <td><input type="range" value="0" min="-3" max="3" step="0.1" id="sl_mu_0" onchange="on_change_mu_0()"></input></td>
    <td><p id="tx_mu_0">0</p></td>
  </tr>
  <tr>
    <td><input type="range" value="0" min="-3" max="3" step="0.1" id="sl_mu_1" onchange="on_change_mu_1()"></input></td>
    <td><p id="tx_mu_1">0</p></td>
  </tr>
  <tr>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
  </tr>
</table>
<table>
  <tr>
    <td rowspan="4">$$\Sigma =$$</td>
    <td rowspan="4"><p id='brackets'>[</p></td>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
    <td rowspan="4"><p id='brackets'>]</p></td>
    <td rowspan="4">$$=$$</td>
    <td rowspan="4"><p id='brackets'>[</p></td>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
    <td rowspan="4"><p id='brackets'>]</p></td>
  </tr>
  <tr>
    <td><input type="range" value="1" min="0" max="2.0" step="0.1" id="sl_sigma_00" onchange="on_change_sigma_00()"></input></td>
    <td><input type="range" value="0" min="-2.0" max="2.0" step="0.01" id="sl_sigma_01" onchange="on_change_sigma_01()"></input></td>
    <td><p id="tx_sigma_00">1</p></td>
    <td><p id="tx_sigma_01">0</p></td>
  </tr>
  <tr>
    <td><input type="range" disabled="true" value="0" min="-2.0" max="2.0" step="0.01" id="sl_sigma_10" onchange="on_change_sigma_10()"></input></td>
    <td><input type="range" value="1" min="0" max="2.0" step="0.1" id="sl_sigma_11" onchange="on_change_sigma_11()"></input></td>
    <td><p id="tx_sigma_10">0</p></td>
    <td><p id="tx_sigma_11">1</p></td>
  </tr>
  <tr>
    <td>&nbsp;</td>
    <td>&nbsp;</td>
  </tr>
</table>
</div>
<div style="width:100%;">
<div id='surfacePlotDiv' style="margin-bottom:10px;margin-left:auto; margin-right:auto; width: 450px; height: 450px;"></div>
</div>

<script type='text/javascript'>

var surfacePlot;

var data, options, basicPlotOptions, glOptions, animated, plot1, values;
var sigma = math.matrix([[1, 0], [0, 1]]);
var mu = math.matrix([[0],[0]]);
var tooltipStrings = new Array();
var values = new Array();
var numRows = 55;
var numCols = 55;

var canvas_width = document.getElementById('surfacePlotDiv').clientWidth;
var canvas_height = document.getElementById('surfacePlotDiv').clientHeight;

function computeValues() {
  var inv_sigma = math.inv(sigma);
  var det_sigma = math.det(sigma);
  var idx = 0;
  var f1 = 1.0/(math.sqrt(math.pow(2*math.pi, 2)*det_sigma));
  for (var i = 0; i < numRows; i++)
  {
    values[i] = new Array();
    var x = (i*6/numRows)-3;
    for (var j = 0; j < numCols; j++)
    {
      var y = (j*6/numCols)-3;
      var x_m_u = math.subtract(math.matrix([[x],[y]]), mu);
      var fe = -0.5*math.multiply(math.multiply(math.transpose(x_m_u), inv_sigma), x_m_u);
      var value = f1*math.exp(fe);
      values[i][j] = value;
      tooltipStrings[idx] = "x:" + x + ", y:" + y + " = " + value;
      idx++;
    }
  }
}


function setUp()
{
  surfacePlot = new SurfacePlot(document.getElementById("surfacePlotDiv"));

  data = {nRows: numRows, nCols: numCols, formattedValues: values};


  // Don't fill polygons in IE < v9. It's too slow.
  var fillPly = true;

  // Define a colour gradient.
  var colour1 = {red:0, green:0, blue:255};
  var colour2 = {red:0, green:255, blue:255};
  var colour3 = {red:0, green:255, blue:0};
  var colour4 = {red:255, green:255, blue:0};
  var colour5 = {red:255, green:0, blue:0};
  var colours = [colour1, colour2, colour3, colour4, colour5];

  // Axis labels.
  var xAxisHeader = "x_1";
  var yAxisHeader = "x_2";
  var zAxisHeader = "f_x";

  var renderDataPoints = false;
  var background = '#ffffff';
  var axisForeColour = '#000000';
  var hideFloorPolygons = true;
  var chartOrigin = {x: 0, y: 0};

  // Options for the basic canvas pliot.
  basicPlotOptions = {fillPolygons: fillPly, tooltips: tooltipStrings, renderPoints: renderDataPoints }

  // Options for the webGL plot.
  var xLabels = [-3, -2, -1, 0, 1, 2, 3];
  var yLabels = [-3, -2, -1, 0, 1, 2, 3];
  var zLabels = [0, 0.1, 0.2, 0.3, 0.4]; // These labels ar eused when autoCalcZScale is false;
  glOptions = {xLabels: xLabels, yLabels: yLabels, zLabels: zLabels, chkControlId: "allowWebGL", autoCalcZScale: false, animate: false};

  // Options common to both types of plot.
  options = {xPos: 0, yPos: 0, width: canvas_width, height: canvas_height, colourGradient: colours, 
    xTitle: xAxisHeader, yTitle: yAxisHeader, zTitle: zAxisHeader, 
    backColour: background, axisTextColour: axisForeColour, hideFlatMinPolygons: hideFloorPolygons, origin: chartOrigin};

  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
}

setUp();

function on_change_mu_0() {
  new_mu = parseFloat(document.getElementById("sl_mu_0").value);
  document.getElementById("tx_mu_0").innerHTML = new_mu;
  mu.subset(math.index(0,0), new_mu);
  console.log("Mu: " + mu);
  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
}
function on_change_mu_1() {
  new_mu = parseFloat(document.getElementById("sl_mu_1").value);
  document.getElementById("tx_mu_1").innerHTML = new_mu;
  mu.subset(math.index(1,0), new_mu);
  console.log("Mu: " + mu);
  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
}
function on_change_sigma_00() {
  new_sigma = parseFloat(document.getElementById("sl_sigma_00").value);
  document.getElementById("tx_sigma_00").innerHTML = new_sigma;
  sigma.subset(math.index(0,0), new_sigma);
  console.log("Sigma: " + sigma);
  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
}
function on_change_sigma_01() {
  new_sigma = parseFloat(document.getElementById("sl_sigma_01").value);
  document.getElementById("tx_sigma_01").innerHTML = new_sigma;
  sigma.subset(math.index(0,1), new_sigma);
  console.log("Sigma: " + sigma);
  /*
  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
  */
  document.getElementById("sl_sigma_10").value = document.getElementById("sl_sigma_01").value;
  on_change_sigma_10();
}
function on_change_sigma_10() {
  new_sigma = parseFloat(document.getElementById("sl_sigma_10").value);
  document.getElementById("tx_sigma_10").innerHTML = new_sigma;
  sigma.subset(math.index(1,0), new_sigma);
  console.log("Sigma: " + sigma);
  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
}
function on_change_sigma_11() {
  new_sigma = parseFloat(document.getElementById("sl_sigma_11").value);
  document.getElementById("tx_sigma_11").innerHTML = new_sigma;
  sigma.subset(math.index(1,1), new_sigma);
  console.log("Sigma: " + sigma);
  computeValues();
  surfacePlot.draw(data, options, basicPlotOptions, glOptions);
}

</script>
</div>

## Notes

 * $$\sigma_{2,1}$$ is disabled because $$\Sigma$$ is a symmetric matrix
 * $$\Sigma$$ should be an invertible matrix. If its not then an error will be thrown on the console of your browser and a blank plot will happen
 * You can hold `<Shift>+<Left-Mouse-Button>` over the plot to zoom-in/out


