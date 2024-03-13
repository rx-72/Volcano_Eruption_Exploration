# Writeup
### What we've done
So far, we have created the main visualization which is a geomap of all the documented volcanic eruptions in the U.S. according to [this dataset](https://public.opendatasoft.com/explore/dataset/significant-volcanic-eruption-database/table/). We first did some EDA on the dataset and realized that the `total_deaths` column was not very reliable due to the difficult nature of collecting data on that, so we opted to bin it into discrete ordinal categories. We marked each volcanic eruption as a circle, and we encoded the eruption's [explosive index](https://www.nps.gov/subjects/volcanoes/volcanic-explosivity-index.htm#) into the radius of the circle. We then added some interactive features such as allowing to filter for volcanic regions and time period through buttons. 

### Main challenges
As part of our narrative design, we plan to then allow the user to "zoom out" from the 2D U.S. map into a 3D globe, which we imagine would be difficult to make a smooth transition for. We are using a Scrollytelling form for our site, so we would have to somehow "increment" the zooming out animation based on the scrolling progress. After that, we also have to figure out what marks to use for eruptions on a 3D map, and making it behave intuitively and look aesthetic will be a large portion of the difficulty. Another challenge is to figure out how to encode additional columns, since adding a 2D circle onto a 3D graph might look weird. Our current plan is to use cylinders that have a "height" encoding which represents either `total_deaths` or `total_damage`


# proposal

Narrative idea:
  - iterate through all normal volcanoes and highlight especially deadly ones (ie high death, damage, injury, etc)
     - perhaps give certain unique sections (and interactive buttons) to special eruptions like pompeii (vesuvius) or kraketoa
  - Similarly hypothesize certain eruptions to be more deadly pre interaction
  - Compare earthquake based volcanoes to tsunami based volcanos
  - geo map style? (maybe lab 6 look into)
  - iterate by year (implement auto, speed up, pause, skip to certain year, displaying eruptions overtime overlayed with world map)
  - color or other data reprsentation splitting for country, number of deaths, damage etc. (some sort of categorical data representation, probably look into lecture 3 the more categorical stuff we add)
  - Combine with a narrative story idea that guides the reader about how these eruptions or common ideas will be tested by both us and the reader through the interactive.
    - (Utilize lab 7 narrative style and lab 6 style mapbox and conditioning?) 
