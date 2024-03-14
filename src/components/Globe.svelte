<script>
    //globe section

    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import * as topojson from 'topojson-client';
    import { geoPath, geoAlbersUsa } from 'd3-geo';
    import { draw } from 'svelte/transition';
    import { geoOrthographic, geoGraticule10} from "d3-geo";
    import { json } from "d3-fetch";

    const projection_globe = d3.geoEqualEarth() //.scale(400);
    const graticule = geoGraticule10();
    const path_globe = geoPath(projection_globe)

    let outline = ({type: "Sphere"});
    let land, borders;
    let selected = 1;
    let mapper = selected;
    let test = 0;

         
    var scatterX, scatterY;
    let svgWidth, svgHeight;
    var margin = { top: 20, right: 50, bottom: 50, left: 20 };
    let svg;
    let dots = 'temp';

    export let volcanos;
    export let US_volcanos;

    // json(
       // "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json"
       // ).then((world) => {
       // land = topojson.feature(world, world.objects.land);
       // borders = topojson.mesh(world, world.objects.countries, (a, b) => a !== b)
    // });

    onMount(async () => {
    	const world = await fetch(
      		"https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json"
    	).then((d) => d.json());
    	while (US_volcanos.length === 0) {
                        // Pause for 100 ms
                        await new Promise(resolve => setTimeout(resolve, 100));
                }
                updateFilteredData();

    	land = topojson.feature(world, world.objects.land);
    	// console.log({ features })

    	borders = topojson.mesh(world, world.objects.countries, (a, b) => a !== b);
      let min_year = filteredVolcanos.reduce( (min, obj) => (obj.year < min ? obj.year : min), filteredVolcanos[0].year);
      let max_year = filteredVolcanos.reduce( (max, obj) => (obj.year > max ? obj.year : max), filteredVolcanos[0].year);

      
      // Function to update SVG when user resizes window
      function updateSize() {

              svgWidth = parseFloat(d3.select('svg').style('width'))/3;
              svgHeight = parseFloat(d3.select('svg').style('height'));


              svg.attr("width", svgWidth + margin.left + margin.right)
                  .attr("height", svgWidth + margin.left + margin.right)

              var fontSize = Math.max(12, (svgWidth - margin.left - margin.right) / 110);

              // Update the x-axis scale
              scatterX.range([0, svgWidth]);
              scatterY.range([0, svgWidth * yScale]);

              svg.select('.x-axis').call(d3.axisBottom(scatterX))
                  .selectAll("text").style('font-size', fontSize + "px");
              svg.select('.y-axis').call(d3.axisLeft(scatterY))
                  .selectAll("text").style('font-size', fontSize + "px");

              dots.attr('transform', 'translate(' + (margin.left*3) + ',' + ( svgWidth/4 + margin.top ) + ')');

              console.log("Update size", svgWidth, svgHeight);

      }

      console.log("Globe eruptionIndexing");
      
      svgWidth = parseFloat(d3.select('svg').style('width'))/3;
      svgHeight = parseFloat(d3.select('svg').style('height'));
      if (isNaN(svgHeight)) {
        svgHeight = svgWidth*3/2;
      }
      console.log("SVG Dimensinos: ",svgWidth, svgHeight);

      let initialFontSize = Math.max(12, (svgWidth - margin.left - margin.right) / 110);
      console.log(svgWidth, svgHeight, "Dimensions");

      svg = d3.select("#scatter")
              .append("svg")
              .attr("width", svgWidth + margin.left*3 + margin.right)
              .attr("height", svgHeight)
              .append("g")

              
      // x-axis
      scatterX = d3.scaleLinear().domain([min_year, max_year]).range([0, svgWidth]);
      svg.append("g")
              .attr("class", "x-axis")
              .style('font-size', initialFontSize+'px')
              .attr("transform",
                      "translate(" + margin.left*3 + "," + (svgWidth + margin.top) + ")")
              .call(d3.axisBottom(scatterX).ticks(7));
      
      let yScale = 3/4;

      // y-axis without tick marks
      scatterY = d3.scaleLinear().domain([8, 0]).range([0, svgWidth * yScale]);
      // svg.append("g")
      //         .attr("class", "y-axis")
      //         .call(d3.axisLeft(scatterY))
      //         .attr('text-anchor', 'end')
      //         .style('font-size', '0px')
      //         .attr('transform',
      //                  'translate(' + (margin.left*3 + yOffset) + ',' + (margin.top + svgWidth/4 ) + ')')

      // tick marks
      var yMarks = svg.append("g")
              .attr("class", "y-axis-marks")
              .call(d3.axisLeft(scatterY))
              .attr('text-anchor', 'end')
              .style('font-size', initialFontSize+'px')
              .attr('transform',
                      'translate(' + (margin.left*2.5) + ',' + (margin.top + svgWidth/4 ) + ')')
      // yMarks.select('.domain').style('stroke', 'none');

      // x-axis label
      svg.append('text')
              .attr('class', 'x-label')
              .text("Year")
              .style('font-size', initialFontSize+2 + 'px')
              .attr("transform",
                      "translate(" + (margin.left*3 + svgWidth/2 - (initialFontSize+7) ) + "," + (margin.top + svgWidth + margin.bottom) + ")");

      
      // y-axis label
      svg.append('text')
              .attr('class', 'y-label')
              .attr('text-anchor', 'middle')
              .attr('text-align', 'center')
              .attr('transform', 'translate(' + (margin.left) + ',' + (margin.top + svgWidth * (1-yScale) + (svgWidth*yScale/2)) + ') rotate(-90)')
              .text("Explosivity")
              .style('font-size', initialFontSize + 2 + 'px');


      dots = svg.append('g')
              .selectAll('dot')
              .data(filteredVolcanos)
              .enter()
              .append('circle')
              .attr('cx', function (d) { return scatterX(d.year); })
              .attr('cy', function (d) { return scatterY(d.Volcano_explosive_index); })
              .attr('r', 4 )
              .style('fill', function (d) {return d.Volcano_explosive_index >= 5 ? 'red' : '#ffaa20'})
              .attr('transform', 'translate(' + (margin.left*3) + ',' + ( svgWidth/4 + margin.top ) + ')');



              
      // updateSize when user resizes window
      window.addEventListener("resize", updateSize);

    });

    function populateScatter(svgg) {
                dots = svgg.append('g')
                        .selectAll('dot')
                        .data(filteredVolcanos)
                        .enter()
                        .append('circle')
                        .attr('cx', function (d) { return scatterX(d.year); })
                        .attr('cy', function (d) { return scatterY(d.Volcano_explosive_index); })
                        .attr('r', 4 )
                        .style('fill', function (d) {return d.Volcano_explosive_index >= 5 ? 'red' : '#ffaa20'})
                        .attr('transform', 'translate(' + (margin.left*3) + ',' + ( svgWidth/4 + margin.top ) + ')');
                        
    }

    function calculateAverageExplosivity() {
                const total = filteredVolcanos.reduce( (sum, volcano) => sum + parseInt(volcano.Volcano_explosive_index), 0);
                const avg = total / filteredVolcanos.length;
                console.log("Total:", total, "length: ", filteredVolcanos.length);
                console.log('average explosivity: ',avg, avg.toFixed(2));
                return avg.toFixed(2);
        }

    const width = 1100;
    const height = 600;

    function coord_proj_cx(d) {
                let coords = [  
                        { lat: d['latitude'], long: d['longitude'] },
                        ].map(p => projection_globe([p.long, p.lat]))
                //$: console.log(coords[0][0])
                return coords[0][0]
    }

    function coord_proj_cy(d) {
            let coords = [
		    { lat: d['latitude'], long: d['longitude'] },
	            ].map(p => projection_globe([p.long, p.lat]))
            //$: console.log(d['longitude'])
            //$: console.log(coords[0][1])

            return coords[0][1]
    }

    function showTooltip(d) {
                d3.select("#tooltip")
                  .style("visibility", "visible")
                  .html(`<b>${d.name}</b><br>Year: ${d.year}<br>Country: ${d.country}<br>Explosivity: ${
                    d.Volcano_explosive_index === "" ? "Unknown" : d.Volcano_explosive_index
                }<br>Destruction Rank (From 1 to 15): ${d.damage_caused_rank}`)
                  .style("left", (event.pageX + 10) + "px")
                  .style("top", (event.pageY + 10) + "px");
                console.log(d)
    }

    function hideTooltip(d) {
            d3.select("#tooltip")
                .style('visibility', 'hidden')
    }

    let selectedYear;
    let prevSelectedYear = null;

    let filterState = {
    	year: null, // Initialize with no year filter
    	location: null, // Initialize with no location filter
    };

    let slicing = volcanos;
    let filteredVolcanos = volcanos;
    const yearCategories = ["pre-1800s", "1800s", "1900s", "2000s"];

    function filterByYear(year) {
    if (year === "1800s") {
      filterState.year = 1800;
    } else if (year === "1900s") {
      filterState.year = 1900;
    } else if (year === "2000s") {
      filterState.year = 2000;
    } else if (year === "pre-1800s") {
      filterState.year = "pre-1800s";
    } else {
      filterState.year = null; // Reset filter in other cases
    }
      updateFilteredData();
    }

    function filterByYearAndUpdateClass(category, button) {
    // If same button pressed twice, deactivate it and change category to null

	selected = 1;
    	test = 0;
    	if (selectedYear === category) {
      		selectedYear = null;
      		category = null;
    	} else {
      		// If not, change the selected year to the pressed button
      		selectedYear = category;
    	}

    	// Deactivates the previously pressed button
    	if (prevSelectedYear && prevSelectedYear !== button) {
      		prevSelectedYear.setAttribute("class", "btn btn-outline-primary");
    	}

    	// Set the prevSelectedYear to the current button
    	if (button) {
      		button.setAttribute("class", "btn btn-primary");
      		prevSelectedYear = button;
    	}

    	filterByYear(category);
  }

  function updateFilteredData() {
    filteredVolcanos = volcanos.filter((d) => {
      const yearMatches =
        filterState.year === "pre-1800s"
          ? d.year < 1800
          : filterState.year !== null
            ? d.year >= filterState.year && d.year <= filterState.year + 99
            : true;
      //const locationMatches = filterState.location ? d.location === filterState.location : true;
      // console.log("Filtering by year:", filterState.year);
      // console.log("Filtering by location:", filterState.location);
      // console.log("Number of matches:", filteredVolcanos.length);

      slicing = filteredVolcanos;
      return yearMatches; //&& locationMatches;
    });
    console.log(dots);
    if (dots != 'temp') {
      console.log('populateScatter');
      dots.remove();
      populateScatter(svg);
    }
  }

    //console.log(volcanos);
  $: mapper = selected;
  $: if (test === 0) {
    slicing = filteredVolcanos.slice(0, mapper);
  } 
</script>

<link
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
  rel="stylesheet"
  integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
  crossorigin="anonymous"
/>

<div class="buttons">
  {#each yearCategories as category}
    <button
      class="btn"
      class:btn-outline-primary={selectedYear !== category}
      class:btn-primary={selectedYear === category}
      on:click={() => filterByYearAndUpdateClass(category)}
    >
      {category}
    </button>
  {/each}
</div>

<button
  class="one"
  on:click={() => (selected += 1)}
  on:click={() => updateFilteredData()}
  on:click={() => (test = 0)}
>
  Get Next Eruption
</button>
<button
  class="all"
  on:click={() => (test = 1)}
  on:click={() => updateFilteredData()}
  on:click={() => (selected = 0)}
>
  Get All Eruptions
</button>
<table class='map_plot_split'>
  <tr>
    <th class='map_col'>
      <div class="globe"> 

          <svg
          {width}
          {height}
          viewBox="-120 0 {width} {height}"
          style:max-width="100%"
          style:height="auto"
          style:display="block"
          style:margin="auto"
          style:max-height="100%"
          >   

              <path d={path_globe(outline)} fill="#09e6ed"/>
              <path d={path_globe(graticule)} stroke="none" fill="none"/>
              <path d={path_globe(land)} fill="lightgreen"/>
              <path d={path_globe(borders)} fill="none" stroke="black" />
              <path d={path_globe(outline)} fill="none" stroke="black" />

              {#each slicing as d, i}
                      <circle
              class="erupt"
                          cx={coord_proj_cx(d)}
                          cy={coord_proj_cy(d)}
                          r={4*(d.Volcano_explosive_index)+4}
                    fill={d.Volcano_explosive_index >= 5 ? 'red' : '#ffca20'}
                          opacity={0.6}
                    stroke="gray"
                    role="button"
                          on:mouseover={
                                  showTooltip(d)
                          }
                          on:mouseleave={
                                  hideTooltip(d)
                          }
                      />
          {#if i + 1 === selected}
                  <circle
                  class="erupt"
                    cx={coord_proj_cx(d)}
                    cy={coord_proj_cy(d)}
                    r={(4 * d.Volcano_explosive_index + 4) / 2}
                    fill="black"
                  />
		<circle
                  class="special"
                    cx={coord_proj_cx(d)}
                    cy={coord_proj_cy(d)}
                    r={(4 * 5+ 4)}
                    fill="blue"
		    on:mouseover={showTooltip(d)}
                    on:mouseleave={hideTooltip(d)}
                  />
                {/if}
              {/each}

            <circle cx="-45" cy="490" r="20" fill="orange" opacity={0.6} />
            <text x="-10" y="495" font-size="16px"> Explosivity less than 5 </text>
            <circle cx="-45" cy="550" r="30" fill="red" opacity={0.6} />
            <text x="-10" y="550" font-size="16px"> Explosivity greater </text>
            <text x="-10" y="565" font-size="16px"> than or equal to 5 </text>
          </svg>
      </div>
    </th>

    <th class="plot_col">
      <div id="scatter">
      </div>
      <p>Total # Eruptions: {filteredVolcanos.length}</p>
      <p>Avg Explosivity: {calculateAverageExplosivity(filteredVolcanos)}</p>
    </th>
  </tr>
</table>
<div id='tooltip'/>

<style>
    .globe {
        animation: fadeIn 2.5s;
    }

    @keyframes fadeIn {
        0% { opacity: 0; }
        100% { opacity: 1; }
    }

    @keyframes fadeOut {
        100% { opacity: 0; }
        0% { opacity: 1; }
    }

    #tooltip {
        background-color: white;
        padding: 8px;
        border: 1px solid black;
        border-radius: 4px;
        position: absolute;
        text-align:left;
        visibility: hidden;
    }

  .all {
    background-color: #8b0000;
    border: white;
    transition-duration: 0.4s;
    color: white;
    margin: 5px;
    text-align: center;
  }

  .all:hover {
    background-color: red;
    border: #04aa6d;
  }

  .one {
    background-color: orange;
    border: white;
    transition-duration: 0.4s;
    color: white;
  }

  .one:hover {
    background-color: red;
    border: #04aa6d;
  }

  .erupt {
    animation: fadeIn 0.5s;
  }

  .special {
    animation: fadeIn 2s;
    animation: fadeOut 5s;
    animation-fill-mode: forwards;
  }

  .map_plot_split {       
                text-align: center;
          width: 100%;
  }
  .map_col {
          width: 60%;
  }
  .plot_col {
          width: 40%;
  }
</style>
