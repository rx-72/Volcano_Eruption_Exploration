<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import Graph from './Graph.svelte';
    import Globe from './Globe.svelte';

    let volcanos = [];
    let US_volcanos = [];
    let selected = Graph

    onMount(async () => {
        const res = await fetch(
            'Volcano_data.csv',
        );
        const csv = await res.text();
        await d3.csvParse(csv, (d) => {
            volcanos.push({
               year: (new Date(Date.UTC(d["year"]))).getFullYear() + 1,
               month: d["month"],
               day: d["day"],
               name: d["name"],
               location: d["location"],
               country: d["country"],
               elevation: d["elevation"],
               type: d["type"],
               status: d["status"],
               total_deaths: d["total_deaths"],
               total_deaths_description: d["total_deaths_description"],
               total_injuries: d["total_injuries_description"],
               total_damage: d["total_damage_description"],
               houses_destroyed: d["total_houses_destroyed_description"],
               Tsunami_caused: d["Tsunami caused?"], 
               Earthquake_caused: d["Earthquake caused?"],
               Volcano_explosive_index: d["Volcano Explosive Index"],
               longitude: d["longitude"],
               latitude: d["latitude"],
	       damage_caused_rank: d["total"],	
            });
        });
        volcanos = volcanos;
    });

    onMount(async () => {
        const res = await fetch(
            'US_Volcano_data.csv',
        );
        const csv = await res.text();
        await d3.csvParse(csv, (d) => {
            US_volcanos.push({
               year: (new Date(Date.UTC(d["year"]))).getFullYear() + 1,
               month: d["month"],
               day: d["day"],
               name: d["name"],
               location: d["location"],
               country: d["country"],
               elevation: d["elevation"],
               type: d["type"],
               status: d["status"],
               total_deaths: d["total_deaths"],
               total_deaths_description: d["total_deaths_description"],
               total_injuries: d["total_injuries_description"],
               total_damage: d["total_damage_description"],
               houses_destroyed: d["total_houses_destroyed_description"],
               Tsunami_caused: d["Tsunami caused?"], 
               Earthquake_caused: d["Earthquake caused?"],
               Volcano_explosive_index: d["Volcano Explosive Index"],
               longitude: d["longitude"],
               latitude: d["latitude"],
	       damage_caused_rank: d["total"], 
            });
        });
        US_volcanos = US_volcanos;
    });

</script>

<!-- <main>
    <Graph {volcanos} {US_volcanos}/>
</main> -->

<svelte:component this={selected} {volcanos} {US_volcanos}/>

<button class="test" on:click={()=> selected = Graph}>
	US map
</button>
<button class="test" on:click={()=> selected = Globe}>
	World Globe
</button>

<style>
    .test{
        background-color: #04AA6D;
        border: black;
        text-align: center;
        font-size:25px;
        display: inline-block;
        transition-duration: 0.4s;
        margin:5px;
        padding:15px;
	    animation: fade 2s;
    }

    .test:hover{
        background-color: white;
        border :#04AA6D
    }

    @keyframes fade {
        0%,100% { opacity: 0 }
        50% { opacity: 1 }
    }

</style>
