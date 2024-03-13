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
    let hasClicked = false; 
         
    var scatterX, scatterY;
    let svgWidth, svgHeight;
    var margin = { top: 20, right: 50, bottom: 50, left: 20 };
    let svg;
    let projection = geoOrthographic();
    let path = geoPath().projection(projection);
    let dots = 'temp';
    let infoText = "The eruption of Mt. Vesuvius in 79 A.D is perhaps the single most recognized eruption in history, known for destroying the city of Pompeii. But is it the biggest eruption in history? Click the button to find out.";
    export let volcanos;
    export let US_volcanos;
    
    const firstCircleSize = 5; // Significantly smaller
    const secondCircleMultiplier = 10;
    // json(
       // "https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json"
       // ).then((world) => {
       // land = topojson.feature(world, world.objects.land);
       // borders = topojson.mesh(world, world.objects.countries, (a, b) => a !== b)
    // });
     // Initial setup for zoomed-in view
     function setupProjection(scale, rotate, showSecondPoint = false) {
        projection.scale(scale).rotate(rotate).translate([400, 225]).clipAngle(90);
        updateGlobe(showSecondPoint);
    }
     // Function to update the globe and points
     async function updateGlobe(showSecondPoint) {
        const worldDataUrl = "https://cdn.jsdelivr.net/npm/world-atlas/world/50m.json";
        const world = await d3.json(worldDataUrl);
        const countries = topojson.feature(world, world.objects.countries).features;

        d3.select(svg).selectAll("*").remove();

        d3.select(svg).append('path')
            .datum({type: 'Sphere'})
            .attr('fill', '#a4bac7')
            .attr('d', path);

        d3.select(svg).selectAll('path.country')
            .data(countries)
            .enter().append('path')
            .attr('class', 'country')
            .attr('d', path)
            .attr('fill', '#ccc');

        // Original point (Mount Vesuvius 79)
        const vesuviusCoord = projection([14.426, 40.821]);
        d3.select(svg).append('circle')
            .attr('cx', vesuviusCoord[0])
            .attr('cy', vesuviusCoord[1])
            .attr('r', firstCircleSize)
            .attr('fill', 'red');

        d3.select(svg).append('text')
            .attr('x', vesuviusCoord[0] + 10)
            .attr('y', vesuviusCoord[1])
            .text('Mount Vesuvius 79')
            .attr('fill', 'black');

        if (showSecondPoint) {
            // Second point (Mount Etna 1991)
            const etnaCoord = projection([15.004, 37.734]);
            d3.select(svg).append('circle')
                .attr('cx', etnaCoord[0])
                .attr('cy', etnaCoord[1])
                .attr('r', firstCircleSize * secondCircleMultiplier)
                .attr('fill', 'red');

            d3.select(svg).append('text')
                .attr('x', etnaCoord[0] + (firstCircleSize * secondCircleMultiplier) + 10)
                .attr('y', etnaCoord[1])
                .text('Mount Etna 1991')
                .attr('fill', 'black');
        }
    }
    onMount(() => {
        setupProjection(2400, [-12.8333, -42.8333]); // Significantly more zoomed in on Italy
    });

    // This is called when the button is clicked to show the second eruption
    function comparativeEruption() {
        hasClicked = true; 
        infoText = "As you can see, despite its reputation Vesuvius is not even the biggest eruption in present-day Italy! Eruptions can be measured using the Volcanic Explosivity Index, or VEI. Similar to the Richter Scale for Earthquakes, the VEI is logarithmic, meaning that a 2 on the VEI is an eruption 10x as powerful as a 1. The eruption that destroyed Pompeii has a VEI of 5, while the 1991 eruption of Mt. Etna is listed as a 7, meaning that it was 100x as explosive! Knowing this, we can deduce that the eruption of Mt. Vesuvius is not famous for its overwhelming scale, but for the historical lack of preparation and loss of life which it caused.";
        setupProjection(1200, [-12.8333, -42.8333], true); // Zooms out a bit
    }
</script>
<button on:click={comparativeEruption}>Comparative Eruption</button>
<div class="container">
    <svg bind:this={svg} width="800" height="450"></svg>
    <div class="info-text">
      {infoText}
    </div>
  </div>
<style>
    
  .container {
    display: flex;
    align-items: flex-start;
    justify-content: center;
    height: 50%;
  }

  .info-text {
    margin-left: 20px;
    padding: 20px;
    color: black;
    max-width: 300px;
    text-align: left;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }
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
