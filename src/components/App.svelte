<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';
  import Graph from './Graph.svelte';

  const width = 928;
  const height = 500;
  const marginTop = 10;
  const marginRight = 10;
  const marginBottom = 20;
  const marginLeft = 40;

  let data = [];
  let targetYear = "1964";
  let targetCountryCode = "USA";

  onMount(async () => {
      const res = await fetch(
          '/gdp_gini_filtered_data_with_na.csv',
      );
      const csv = await res.text();
      await d3.csvParse(csv, (d) => {
          data.push({
             year: +d["Year"],
             country_name: d["Country Name"],
             country_code: d["Country Code"],
             indicator_code: d["Indicator Code"],
             indicator_name: d["Indicator Name"],
             value: +d["Value"],
          });
      });
      data = data;
  });

  let lorenzData = [];

onMount(async () => {

 const res = await fetch('lorenz_simulation.csv'); 

 const csv = await res.text();

 lorenzData = d3.csvParse(csv, function(d){
    return{
        gini: +d.gini,
        pop: +d.population,
        income: +d.income
    };
 })

 console.log(lorenzData);

 drawLorenz(lorenzData);


});


function drawLorenz(lorenzData){
var margin = {top: 30, right: 30, bottom: 45, left: 60},
width = 460 - margin.left - margin.right,
height = 400 - margin.top - margin.bottom;

var svg = d3.select("#lorenzCurve")
.append("svg")
.attr("width", width + margin.left + margin.right)
.attr("height", height + margin.top + margin.bottom)
.append("g")
.attr("transform",
      "translate(" + margin.left + "," + margin.top + ")");

// x and y scales
var x = d3.scaleLinear()
  .domain(d3.extent(lorenzData, function(d){return d.pop;}))
  .range([ 0, width ]);

svg.append("g")
  .attr("transform", "translate(0," + height + ")")
  .call(d3.axisBottom().scale(x).tickFormat(d3.format(".0%")));

var y = d3.scaleLinear()
  .domain(d3.extent(lorenzData, function(d){return d.income;}))
  .range([ height, 0]);

svg.append("g")
  .call(d3.axisLeft().scale(y).tickFormat(d3.format(".0%")));


// 45 degree line
svg.append('line')
    .style("stroke", "darkgreen")
    .style("stroke-dasharray", ("3, 3"))
    .style("stroke-width", 3)
    .attr("x1", width)
    .attr("y1", 0)
    .attr("x2", 0)
    .attr("y2", height);


// original lorenz curve at gini = 0.1
var gini = 0.1;
//var filteredLorenz = filterData(lorenzData, gini);

function filterData(data, giniValue){
    return data.filter(function(entry){
        return entry.gini===giniValue;
    });
};

// initialize line
var line = svg.append('g')
    .append("path")
        .datum(filterData(lorenzData, gini))
        .attr("d", d3.line()
            .x(function(d) { return x(d.pop)})
            .y(function(d) { return y(d.income)})
            )
        .attr("stroke", function(d){return "darkblue"})
        .style("stroke-width", 3)
        .style("fill", "none")



function update(gini){
    // new data
    var filteredLorenz = filterData(lorenzData, gini)

    line.datum(filteredLorenz)
        .transition()
        .duration(1000)
        .attr("d", d3.line()
            .x(function(d){return x(d.pop)})
            .y(function(d){return y(d.income)})
        )
        .attr("stroke", function(d){return "darkblue"})
};

// slider updates gini value and draws new lorenz curve
d3.select("#giniSlider").on("input", function(){
    gini = Number(this.value);
    update(gini);
    document.getElementById("giniVal").textContent=`Gini Coefficient: ${gini}`;
});

// axis lables 
svg.append("text")
    .attr("class", "x label")
    .attr("text-anchor", "end")
    .attr("x", width)
    .attr("font-size", 14)
    .attr("y", height+40)
    .text("Percent of Population");

svg.append("text")
    .attr("class", "y label")
    .attr("text-anchor", "end")
    .attr("y", -50)
    .attr("dy", ".75em")
    .attr("font-size", 14)
    .attr("transform", "rotate(-90)")
    .text("Percent of Income");

//chart title 
svg.append("text")
    .attr("x", (width / 2))             
    .attr("y", 0 - (margin.top / 2))
    .attr("text-anchor", "middle")  
    .style("font-size", "14px") 
    .style("text-decoration", "underline")  
    .text("Lorenz Curve and Gini Coefficient");

// shading?
// tooltip to show what percent of population has what percent of income

};

</script>

<main>
  <!-- svelte-ignore a11y-missing-attribute -->
  <html>
    <head>
      
      <title>DSC 106 Final Group Project</title>
      <!-- <meta charset="utf-8"/>
      <link rel="icon" href="%sveltekit.assets%/favicon.png" /> -->
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      
    </head>

    <body style="margin:100px">
      <h1>Two Methods for Measuring Economic Health</h1>
      <h2>Gross Domestic Product per Capita and the Gini Coefficient</h2>
  
      <div>
        <h3>Gross Domestic Product (GDP)</h3>
        <p>There are many ways to measure the strength of a country's economy such as:</p>
        <ul>
          <li>Trade (imports and exports)</li>
          <li>Labor force participation (employed, unemployed, and not in the labor force)</li>
          <li>Income</li>
        </ul>
        <p>GDP tends to be the most popular statistic for indicating a nation's wealth given it measures the "monetary value of final goods and services" which is the sum of all consumption, investment, government purchases, and net exports (exports minus imports) over a given period of time.</p>
      </div>
  
      <!-- <div style="display: contents">
        <svg id="gdp_choropleth" width="400" height="300"></svg>
        <script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>
        <script src="https://d3js.org/d3-geo-projection.v2.min.js"></script>
      </div>
   -->
      <div>
        <h3>Why could GDP possibly be flawed?</h3>
        <p>GDP doesn't give a full picture of how economic prosperity is distributed given:</p>
        <ul>
          <li>Large countries may have greater GDPs due to simply having a large population, leading many to prefer using GDP per capita.</li>
          <li>A high GDP per capita may also not indicate that all of the nation's people are benefitting much from the total wealth due to how it's distributed across the population.</li>
        </ul>
        <h3>The Gini Index</h3>
        <p>Economic inequality within a nation can be indicated with the Gini index which is derived from the Lorenz curve, a graph of the population's income distribution. The graph has the percent of the population on the x-axis and the proportion of overall income earned by said percent of the population on the y-axis. A line angled at 45 degrees (therefore having a slope of 1) represents perfect equality (so that half of the population earns exactly half of the income for example) an the Gini index is the area between the Lorenz curve and said line at 45 degrees, meaning a greater Gini index indicates greater inequality.</p>
      </div>
  
      <div id="lorenzCurve"></div>
      <input type="range" name="range" class="slider" id="giniSlider" value="0.1"
      min="0.1" max="0.5" step="0.1" oninput="sliderChange(this.value)">
      <span id="giniVal" style="font-size:14px">Gini Coefficient: 0.1</span>
          
        <br>
  
      <!-- <div style="display: contents">
        <svg id="gini_choropleth" width="400" height="300"></svg>
        <script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>
        <script src="https://d3js.org/d3-geo-projection.v2.min.js"></script>
      </div> -->
  
      <div>
        <p>Comparing the Two Measurements:</p>
        <ul>
          <li>There are pros and cons to using each of the measurements.</li>
          <li>GDP measures economic activity well, but fails to provide information on what proportion of the population is involved in said activity.</li>
          <li>The Gini index descrives wealth distribution well, but does not indicate the level of either cumulative or average wealth.</li>
          <li>A more accurate picture can be acquired through looking at both measures.</li>
        </ul>
      </div>
  
      <div style="display: contents">
        <svg id="scatterplot" width="400" height="300"></svg>
      </div>
  
      <div>
        <h2>Demo Writeup</h2>
        <h3>What have you done so far?</h3>
        <p>This demo contains two of our planned visualizations and pulls from two datasets we cleaned. One is a scatterplot that displays the gini index and GDP per capita data for each country for any year chosen by the user. The other is the Lorenz curve which is shaped by user input of the gini index. This demo also contains much of the background information and analysis that will make up the final product's text.</p>
        <h3>What will be the most challenging of your project to design and why?</h3>
        <p>The most challenging aspects of finishing the project will be adding more interactive elements to our visualizations without them interfering with how said visualizations are initially displayed. Making these visualizations (and therefore interactive components) aesthetically pleasing without impeding on how effectively they communicate our message will also be challenging due in part to the subjective nature of many of the design decisions that will need to be made. Significant time may also need to be invested into forming more of a storyline that helps viewers explore our topics and better understand the economics-related message we our aiming to convey. Continuing to become familiar with d3.js syntax as well as other technical skills (such as working with GeoJSON data) will also take some time in order to accomplish the previously-mentioned tasks.</p>
      </div>    


    </body>
    </html>
</main>

<style>

    @import
    url("https://fonts.googleapis.com/css2?family=League+Spartan:wght@100..900&display=swap");
    @import
    url("https://fonts.googleapis.com/css2?family=League+Spartan:wght@100..900&family=Libre+Baskerville:ital,wght@0,400;0,700;1,400&display=swap");
  /* Write your CSS here */
  @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap');

  :root {
    --color-bg: #ffffff;
    --color-outline: #c2c2c2;
    --color-shadow: hsl(0, 0%, 0%, 0.1);
    --color-text: #3f4252;
    --color-bg-1: hsla(0, 0%, 0%, 0.2);
    --color-shadow-1: hsl(0, 0%, 96%);
  }

  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }


    main {
        text-align: left;
        color: #27323F;
        font-family:'Libre Baskerville', serif;
        font-size:12px;
        line-height: 1.8em;
        
    }

    h1 {
        font-family:'League Spartan', sans-serif;
        font-size: 24pt;
        padding:20px;
        text-align:center;
    }
    h2 {
        font-size: 12pt;
        padding:20px
    }
    h3 {
        font-size: 12pt;
        padding:20px
    }
    p {
        font-size: 10pt;
        padding:10px;
    }
    li {
      text-indent:5px;
      margin:25px;
    }
    
    body {background-color:#DDE6ED;}
</style>