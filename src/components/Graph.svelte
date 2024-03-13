<script>
        import * as d3 from 'd3';
        import { onMount } from 'svelte';
	import * as topojson from 'topojson-client';
	import { geoPath, geoAlbersUsa } from 'd3-geo';
	import { draw } from 'svelte/transition';
        import { geoOrthographic, geoGraticule10} from "d3-geo";
	import { json } from "d3-fetch";
        let svg;
        let gx;
        let gy;
        // grabbing the data
        export let volcanos;
        export let US_volcanos;

        const projection = geoAlbersUsa().scale(1300).translate([487.5, 305])
        const path = geoPath().projection(null);
        const yearCategories = ['pre-1800s', '1800s', '1900s', '2000s'];

        let states = [];
        let counties = [];
        let mesh;
        let selected;
	
	let test = 0;
  	let select = 1;
	let mapped = select;
        
        let selectedYear;
        let prevSelectedYear = null;
        
        let selectedLocation = null;
        let prevSelectedLocation = null;
        
        let filterState = {
                year: null, // Initialize with no year filter
                location: null // Initialize with no location filter
         };
        let filteredVolcanos = US_volcanos;
  	let slicing = filteredVolcanos;

        function filterByYear(year) {
                if (year === '1800s') {
                        filterState.year = 1800;
                } else if (year === '1900s') {
                        filterState.year = 1900;
                } else if (year === '2000s') {
                        filterState.year = 2000;
                } else if (year === 'pre-1800s') {
                        filterState.year = 'pre-1800s'; 
                } else {
                        filterState.year = null; // Reset filter in other cases
                }
                updateFilteredData(); 
        }

        function filterByYearAndUpdateClass(category, button) {
                // If same button pressed twice, deactivate it and change category to null
                select = 1;
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
                        prevSelectedYear.setAttribute('class', 'btn btn-outline-primary');
                }
                
                // Set the prevSelectedYear to the current button
                if (button) {
                        button.setAttribute('class', 'btn btn-primary');
                        prevSelectedYear = button;
                };

                filterByYear(category);
        }

        function filterByLocationAndUpdateClass(location, button) {
                // If same button pressed twice, deactivate it
		select = 1;
    		test = 0; 
                if (selectedLocation === location) {
                        selectedLocation = null;
                        location = null;
                } else {
                // Else change selected location to the button's
                        selectedLocation = location;
                }

                 // Deactivates the previously pressed button
                 if (prevSelectedLocation && prevSelectedLocation !== button) {
                        prevSelectedLocation.setAttribute('class', 'btn btn-outline-primary');
                }

                if (button) {
                        button.setAttribute('class', 'btn btn-primary');
                        prevSelectedLocation = button;
                }

                filterState.location = location;
                updateFilteredData();
        }

        function updateFilteredData() {
                filteredVolcanos = US_volcanos.filter(d => {
                        const yearMatches = filterState.year === 'pre-1800s' ? d.year < 1800 : filterState.year !== null ? d.year >= filterState.year && d.year <= filterState.year + 99 : true; 
                        const locationMatches = filterState.location ? d.location === filterState.location : true;
                        // console.log("Filtering by year:", filterState.year);
                        // console.log("Filtering by location:", filterState.location);
                        // console.log("Number of matches:", filteredVolcanos.length);
			slicing = filteredVolcanos;
                        return yearMatches && locationMatches;
                });
        }


        onMount(async () => {
		const us = await fetch('https://cdn.jsdelivr.net/npm/us-atlas@3/counties-albers-10m.json').then(d => d.json())
                updateFilteredData();
		// console.log({ us })
		
		states = topojson.feature(us, us.objects.states).features;
		// console.log({ features })
		
		counties = topojson.feature(us, us.objects.counties).features;

		mesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);

                let min_year = filteredVolcanos.reduce( (min, obj) => (obj.year < min ? obj.year : min), filteredVolcanos[0].year);
                let max_year = filteredVolcanos.reduce( (max, obj) => (obj.year > max ? obj.year : max), filteredVolcanos[0].year);
                console.log(min_year, max_year);
                
                // Function to update SVG when user resizes window
                function updateSize() {

                        var svgWidth = parseFloat(d3.select('svg').style('width'))/3;
                        var svgHeight = parseFloat(d3.select('svg').style('height'));


                        svg.attr("width", svgWidth + margin.left + margin.right)
                           .attr("height", svgWidth + margin.left + margin.right)

                        var fontSize = Math.max(12, (svgWidth - margin.left - margin.right) / 110);

                        // Update the x-axis scale
                        scatterX.range([0, svgWidth]);
                        scatterY.range([0, svgWidth * yScale])

                        svg.select('.x-axis').call(d3.axisBottom(scatterX))
                           .selectAll("text").style('font-size', fontSize + "px");

                        var yOffset = Math.abs( min_year / (max_year - min_year)) * svgWidth;

                        // Update the y-axis scale
                        scatterY.range([0, svgHeight]);
                        svg.select('.y-axis').call(d3.axisLeft(scatterY))
                           .selectAll("text").style('font-size', fontSize + "px");
                        console.log("Update size", svgWidth, svgHeight);

                }

                console.log("Testing");
                
                
                var margin = { top: 20, right: 50, bottom: 50, left: 20 };
                
                var svgWidth = parseFloat(d3.select('svg').style('width'))/3;
                var svgHeight = parseFloat(d3.select('svg').style('height'));



                let initialFontSize = Math.max(12, (svgWidth - margin.left - margin.right) / 110);
                console.log(svgWidth, svgHeight, "Dimensions");
                console.log(margin);

                var svg = d3.select("#scatter")
                        .append("svg")
                        .attr("width", svgWidth + margin.left*3 + margin.right)
                        .attr("height", svgHeight)
                        .append("g")

                        
                // x-axis
                var scatterX = d3.scaleLinear().domain([min_year, max_year]).range([0, svgWidth]);
                svg.append("g")
                        .attr("class", "x-axis")
                        .style('font-size', initialFontSize+'px')
                        .attr("transform",
                                "translate(" + margin.left*3 + "," + (svgWidth + margin.top) + ")")
                        .call(d3.axisBottom(scatterX));
                
                var yOffset = Math.abs( min_year / (max_year - min_year)) * svgWidth;
                let yScale = 3/4;

                // y-axis without tick marks
                var scatterY = d3.scaleLinear().domain([8, 0]).range([0, svgWidth * yScale]);
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
                        .call(d3.axisLeft(scatterY).tickValues([1,2,3,4,5,6,7,8 ]))
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


                var dots = svg.append('g')
                                .selectAll('dot')
                                .data(filteredVolcanos)
                                .enter()
                                .append('circle')
                                .attr('cx', function (d) { return scatterX(d.year); })
                                .attr('cy', function (d) { return scatterY(d.Volcano_explosive_index); })
                                .attr('r', 2)
                                .style('fill', 'red')
                                // .attr('transform', 'translate(' + (margin.left * 3 + yOffset) + ',' + (margin.top + svgWidth/3) + ')');

                                 












                        
                // updateSize when user resizes window
                window.addEventListener("resize", updateSize);
        })

                
        function coord_proj_cx(d) {
                let coords = [  
                        { lat: d['latitude'], long: d['longitude'] },
                        ].map(p => projection([p.long, p.lat]))
                //$: console.log(coords[0][0])
                return coords[0][0]
        }

        function coord_proj_cy(d) {
                let coords = [
		        { lat: d['latitude'], long: d['longitude'] },
	                ].map(p => projection([p.long, p.lat]))
                //$: console.log(d['longitude'])
                //$: console.log(coords[0][1])

                return coords[0][1]
        }

        $: years = d3.group(volcanos, d => d.year);

        $: Tsunami = d3.group(volcanos, d => d.Tsunami_caused);
        $: earthquake = d3.group(volcanos, d => d.Earthquake_caused);

        $: deadly = [volcanos[277], volcanos[517], volcanos[584], volcanos[540], volcanos[291], volcanos[193], volcanos[822], volcanos[574], volcanos[302], volcanos[304]]

        $: US_years = d3.group(US_volcanos, d => d.year);

        $: US_Tsunami = d3.group(US_volcanos, d => d.Tsunami_caused);
        $: US_earthquake = d3.group(US_volcanos, d => d.Earthquake_caused);

        // $: console.log(deadly)

        function explosive(groups) {
                
                return (2.5 * groups.Volcano_explosive_index)
        }

        //console.log(points)
        // debugger;

        function showTooltip(d) {
                d3.select("#tooltip")
                  .style("visibility", "visible")
                  .html(`<b>${d.name}</b><br>Year: ${
                        d.year >= 0 ? d.year : d.year * -1 + " BC"
                  }<br>Location: ${d.location}<br>Explosivity: ${
                        d.Volcano_explosive_index === "" ? "Unknown" : d.Volcano_explosive_index
                  }<br>Destruction Rank (From 1 to 15): ${d.damage_caused_rank}`)
                  .style("left", (event.pageX + 10) + "px")
                  .style("top", (event.pageY + 10) + "px");
                // console.log(d)
        }

        function hideTooltip(d) {
                d3.select("#tooltip")
                  .style('visibility', 'hidden')
        }



        //globe section

        // const projection_globe = geoOrthographic().scale(400);
        // const graticule = geoGraticule10();
        // const path_globe = geoPath(projection_globe)

        // let outline = ({type: "Sphere"});
        // let land, borders;

        // json(
        //         "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json"
        //         ).then((world) => {
	// 	// then pull out the land-shapes
	// 	land = topojson.feature(world, world.objects.land);
	// 	// and border-shapes
	// 	borders = topojson.mesh(world, world.objects.countries, (a, b) => a !== b)
        // });

        // const width = 1200;
	// const height = 2000;

  	$: mapper = select;

  	$: if (test === 0) {
    		slicing = filteredVolcanos.slice(0, mapper);
  	}

</script>       

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">

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

<div class="buttons">
        {#each Array.from(new Set(US_volcanos.map(d => d.location))) as location}
                <button 
                        class="btn"
                        class:btn-outline-primary={selectedLocation !== location}
                        class:btn-primary={selectedLocation === location}
                        on:click={() => filterByLocationAndUpdateClass(location)}>
                        {location}
                </button>
        {/each}
</div>

<button
  class="one"
  on:click={() => (select += 1)}
  on:click={() => updateFilteredData()}
  on:click={() => (test = 0)}
>
  Get Next Eruption
</button>
<button
  class="all"
  on:click={() => (test = 1)}
  on:click={() => updateFilteredData()}
  on:click={() => (select = 0)}
>
  Get All Eruptions
</button>

<table class="map_plot_split">
        <tr>
                <th class="map_col">
                        <div class="volcanos">  
                                <svg viewBox="-260 10 1500 650">
                                        <!--    If all the necessary data is loaded in  -->
                                        {#if states && counties && mesh}
                                                <g fill="white" stroke="black">
                                                        {#each states as feature, i}
                                                                <path d={path(feature)} 
                                                                 on:mouseover={() => selected = feature} 
                                                                 class="state" 
                                                                 in:draw={{ delay: i * 50, duration: 500 }} 
                                                                 />
                                                        {/each}
                                                </g>
 
                                                {#each slicing as d, i}
                                                                <!-- svelte-ignore a11y-interactive-supports-focus -->
                                                                <!-- svelte-ignore a11y-mouse-events-have-key-events -->
                                                                <circle
								 class="erupt"
                                                                 cx={coord_proj_cx(d)}
                                                                 cy={coord_proj_cy(d)}
                                                                 r={4*(d.Volcano_explosive_index)+4}
                                                                 fill={d.Volcano_explosive_index >= 5 ? 'red' : '#ffca20'}
                                                                 opacity={0.6}
                                                                 stroke="gray"
                                                                 role="button"
                                                                 order={`${d.year}`}
                                                                 aria-label={`Volcano ${d.name}, Year: ${d.year}, Location: ${d.location}`}
                                                                 on:mouseover={
                                                                        showTooltip(d)
                                                                 }
                                                                 on:mouseleave={
                                                                        hideTooltip(d)
                                                                 }
                                                                />
							{#if i + 1 === select}
                						<circle
                  						class="erupt"
                  						cx={coord_proj_cx(d)}
                 						cy={coord_proj_cy(d)}
                  						r={(4*(d.Volcano_explosive_index)+4)/2}
                  						fill='magenta'
                						/>
                					{/if}
                                                {/each}

                                                <circle cx="950" cy="500" r="45" fill="orange" opacity={0.6} />
                                                <text x="1000" y="500" font-size="20px"> Explosivity less than 5 </text>
                                                <circle cx="950" cy="600" r="45" fill="red" opacity={0.6} />
                                                <text x="1000" y="600" font-size="20px"> Explosivity greater </text>
                                                <text x="1000" y="620" font-size="20px"> than or equal to 5 </text>
                                        {/if}
                                </svg>
                        
                                <!-- <svg
                                        {width}
                                        {height}
                                        viewBox="-120 -250 {width} {height}"
                                        style:max-width="100%"
                                        style:height="auto"
                                        style:display="block"
                                        style:margin="auto"
                                        style:max-height="100%"
                                        >   
                        
                                        <path d={path_globe(outline)} fill="#12dbff"/>
                                        <path d={path_globe(graticule)} stroke="black" fill="none"/>
                                        <path d={path_globe(land)} fill="green"/>
                                        <path d={path_globe(borders)} fill="none" stroke="black" />
                                        <path d={path_globe(outline)} fill="none" stroke="black" />
                                </svg> -->
                        </div>
                </th>
                <th class="plot_col">
                        <div id="scatter">
                        </div>
                </th>
        </tr>
</table>

<div id='tooltip'/>
<style>
        #tooltip {
            background-color: white;
            padding: 8px;
            border: 1px solid black;
            border-radius: 4px;
            position: absolute;
            text-align:left;
            visibility: hidden;

        }

	.volcanos {
                animation: fadeIn 1s;
                animation-delay: 0s; 
        }

        @keyframes fadeIn {
                0% { opacity: 0; }
                100% { opacity: 1; }
        }
        .map_plot_split {       
                text-align: center;
                width: 100%;
        }
        .map_col {
                width: 60%;
        }
        .plot_col {
                width: 30%;
        }

	.all {
    		background-color: #8b0000;
    		border: white;    
    		transition-duration: 0.4s;
    		color: white;
    		margin:5px;  
    		text-align: center;  
  	}

  	.all:hover{
        	background-color: red;
        	border :#04AA6D;
        }
  
  	.one {
    		background-color: orange;
    		border: white;    
    		transition-duration: 0.4s;
    		color: white;    
  	}

  	.one:hover{
        	background-color: red;
        	border :#04AA6D;
    	}

 	.erupt {
    		animation: fadeIn 0.5s;
  	}
</style>
