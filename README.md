# d3-histogram

To create a histogram, use
<pre><code>
d3.histogram() 
  .domain()     // domain of the histogram
  .thresholds() // specifies how the histogram generator should create bins
  .value()      // takes data from which the height of each individual bin will be calculated
</code></pre>

<code>.thresholds</code> can take either an array, then it creates bins based on the values of the array or number
in which case it tries to create that many bins for us (not guaranteed)<br>

If We have an <code>xScale</code> then we can pass its domain to the <code>.domain()</code> method for histogram
to use this domain.<br>

Similarly, we can pass <code>xScale.ticks()</code> to <code>.thresholds()</code> method to spread the histogram's bars
evenly.<br>

<code>d3.histogram()</code> method will return a function that we can use to generate bins.<br>

<pre><code>
const histogram = d3.histgram()
                    .domain(xScale.domain())
                    .thresholds(xScale.ticks())
                    .value(data.someProperty)
</code></pre>

To generate bins, we need to pass the data that we want to use to the returned function.

<pre><code>
const bins = histogram(data);
</code></pre>
  
Next, we can obtain a width of individual bars by dividing total width of the svg container by the number of bins (this
information is stored in length property of the bins object <code>bins.length</code>). To account for padding, we need 
to subtract the padding value from the obtained width of single bar.

<pre><code>
  const barWidth = width / bars.length - barPadding  // where width is the width of svg container
</code></pre>


### Example 1. 
<hr>

We have this data at our disposal, each piece of data contains name of the country and a number of births in that country.

<pre><code>
const birthData = [
  { "region": "Åland Islands", "births": 275 },
  { "region": "Albania", "births": 54283 },
  { "region": "Algeria", "births": 588628 }
];
</code></pre>

And we want to create a histogram out of this data, grouping countries by no. of births into some reasonable intervals.<br>

First, we need to specify width and height of our svg container (optionally specify padding for our bars).

<pre><code>
const width = 600,
      height = 600,
      barPadding = 1;
</code></pre>

Next, we need to create a scaling for our <code>x</code> axis by using some d3 scaling function (or we can do 
this step manually without the help of d3 but it is more tedious)

<pre><code>
const xScale = d3.scaleLinear()
                .domain([0, d3.max(birthData, (d) => d.births)])
                .range([0, width]);
</code></pre>

Now that we have our scaling for x set up, we can create a histogram that will use it, obtains our bins and calculate
width of each bin.

<pre><code>
const histogram = d3.histogram()
                    .domain(xScale.domain())
                    .thresholds(xScale.ticks())
                    .value((d) => d.births);
                    
const bins = histogram(birthData);
const barWidth = width / bins.length - barPadding;
</code></pre>

Last thing to do is to create scaling for our y-axis. We want to map lengths of each individual bin (number of data points
in each bin), which are stored in each bin's length property to the height of the svg container.

<pre><code>
const yScale = d3.scaleLinear()
                 .domain([0, d3.max(bins, (d) => d.length)])
                 .range([height, 0]);
</code></pre>

By this point, we are done with the preparations and we can draw our histogram to the svg container, there are the steps
that we will make to accomplish this task.

<ul>
  <li>select our svg container and add attributes width and height to it</li>
  <li>select elements with class <code>bar</code> which will be group <code>g</code> that will store rectangles and text</li>
  <li>use <code>bins</code> as the data for histogram</li>
  <li>enter and append groups<code>g</code></li>
  <li>store this selection so that we can add both rectangles and text to it</li>
</ul>

<pre><code>
const bar = d3.select('svg)
    .attr('width', width)
    .attr('height', height)
  .selectAll('.bin')
  .data(bins)
  .enter()
  .append('g')
    .classed('bin', true)
</code></pre>

In the next step, we append rectangles to the selection that we have stored in <code>bar</code> variable. We need to 
set its <code>x</code>, <code>y</code>, <code>width</code>, <code>height</code> and <code>fill</code> attributes 
accordingly. 

<pre><code>
bars
  .append('rect')
    .attr('x', (d, i) => xScale(d.x0))
    .attr('y', (d) => yScale(d.length))
    .attr('height', (d) => height - yScale(d.length))
    .attr('width', (d) => xScale(d.x1) - xScale(d.x0) - barPadding
    .attr('fill', 'purple');
</code></pre>
    
Lastly, we perform the similar process to append <code>text</code> to the svg 

<pre><code>
bars
  .append('text')
   .text((d) => `${d.x0} - ${d.x1} (bar height: ${d.length})`)
   .attr('transform', 'rotate(-90)')
   .attr('y', (d) => (xScale(d.x1) + xScale(d.x0)) / 2)
   .attr('x', - height + 10)
   .style('alignment-baseline', 'middle');
</code></pre>
   
Most of the text styling requires some testing for the text to look good for the given svg so it should not
concers us too much. One thing to notice is that we are rotating the text to negative 90 degrees 
which swaps its x and y orientation so we have to account for that.

