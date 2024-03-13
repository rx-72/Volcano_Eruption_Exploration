<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import App from "../components/App.svelte";

  let count, index, offset, progress;
</script>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>
  <!-- <div class="background" slot="background">

  </div> -->

  <div class="foreground" slot="foreground">
    <section class="WriteUp">
      <h1 class="title">Are Volcanoes Becoming Deadlier?</h1>
      <h2> A visualization by David Lycke, Dhilan Sunanda Bong, and Joshua Huang</h2>
      <div class='pad'></div>
      <p class="para">
        Volcano eruptions, while volatile, don't happen often and we are also able to predict when they'll occur. <br> 
        However, with the world facing record high temperatures because of climate change, <br>
        experts theorized that this warming may lead to increasing frequency of some natural phenomenoms, <br>
        including volcanic eruptions. 
      </p>
      <p class='para'>
        As such, we could expect to see increasing counts of volcanic eruptions with greater explositivity over the upcoming years. <br> 
        This could prove problematic as consecutive eruptions will whittle down our livable space, all the while our population continues to skyrocket. <br> 
        Along with that, multiple volcano eruptions on a large scale could lead to a cooling event which could destroy ecosystems.
      </p>
      <p class="para">Let's put this theory to the test. To determine the activity extent of volcanos currently, <br>
        we've plotted multiple historically recorded volcano eruptions ranging from all the way in the BC period to the near present. <br> 
        Try exploring and hovering the US map first and compare the occurences and intensity of eruptions between years. <br>
        For an extra activity, try to find the eruptions with the highest destructive rank. Then, switch to the global map and try exploring similar or different trends! </p>
        
    </section>
    <section class="Graph">
      US Volcano records over the past 200+ years. Choose one or multiple
      filters and explore! Switch to the world map for even further exploration:
  
      <App />
    </section>
    <section class="Conclusion">
      <h1 class="title">Overall Conclusions:</h1>
      <p class="para"> Starting with the US map, you'll notice a lot of the more explosive volcanos were more from the past pre 1800s. <br>
          This isn't too suprising since this age covered a much larger gap in time compared to the 1800s, 1900s and 2000s. <br>
          Aggregating on the timelines themsleves, we seem to see a much smaller decrease in explosiveness for volcanos before <br>
          briefly rising again in the 1900s and then dropping again. Aggregating on just the areas themselves, aside from the <br>
          Alaska related areas themselves, all saw larger explosiveness during the early pre 1800s and then slowly decreased in <br>
          the later years. Alaska areas were the exception, which tended to saw similar exposivenss in the pre 1800s and 1900s <br>
          but also saw breaks with 0 recorded volcano eruptions from the 1900s and the 2000s.
      </p>
      <p class="para">
          Switching over to the world map, we see a sort of similar trend where we have a very high concentration of explosive volcanos <br>
          in the pre 1800s. Moving into the 1800s, we see a much smaller scale plot of volcano eruptions with a lower scale of explosivity <br>
          with exception to Indonesia which witnessed 2 large scale eruptions (along with some of the deadliest eruptions as well). The <br>
          1900s saw a general scale increase in these volcanos (particularly in the eastern side of the globe) along with a few more red marked <br>
          eruptions. Finally the 2000s witnessed a more lowered or mellowed out scale eruptions with just a few orange dots (although this could <br>
          be because the 2000s to present is a much shorter time range than the 1800 and 1900s). 
      </p>
      <p class="para">
          Just looking at the results alone, we realistically come to the conclusion that eruption occurences are decreasing, with both the US map <br>
          and the world map demonstrating large scale decreasing or zigzag eruption counts and explosivity ranges per every year range. This is a <br>
          conclusion to come forward to but it lacks some insight in how the data grouped by since the pre-1800s holds a much larger time range <br>
          (since it's effectively everything before 1800) along with the 2000s holding a much smaller time range (since the 2000s just begun). <br>
          That said the pre-1800s holds a similar count in volcanos to the later time ranges (but with higher explosivity ranges which makes sense <br>
          realistically since larger scale eruptions are more likely to occur in larger time ranges) but the main factor towards making a conclusion <br>
          on this plot is that the fact that the 1800s to 1900s saw an increase and boom in both occurences and exposivity range. This is important <br>
          since climate change began occuring near the 1850s to 1900s. Therefore the evidence of occurences and scale being factored with climate <br>
          change is supported by the plot and we can see some evidence of both occuring on the US and world map. The next step would be to follow <br>
          up this confirmation with further data exploration and data binning from the 1800s to present.
      </p>
      </section>
  </div>
</Scroller>

<style>
  .Graph {
    height: 110vh;
    background-color: lightblue; /* 20% opaque */
    /* color: white; */
    outline: black;
    text-align: center;
    max-width: 5000px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
    width: 100%;
    margin-right: auto;
    margin-left: auto;
    margin-top: 0px;
  }

  .WriteUp {
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: black;
    text-align: center;
    max-width: 5000px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  .Conclusion {
    height: 120vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: black;
    text-align: center;
    max-width: 5000px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  .title {
    padding: 1em;
  }

  .para {
    padding: 1em
  }

  h2 {
    text-align:center;
    font-size: 18px;
    color:gray;
    margin: 2px;
  }
  .pad {
    margin:15vh;
  }
</style>
