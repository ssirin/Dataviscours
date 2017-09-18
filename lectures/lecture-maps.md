---
layout: code-lecture
title:  Maps
permalink: /lectures/lecture-maps/
nomenu: true
---

## Maps

Before we start talking about how to draw maps, a word of caution: maps are heavily over-used. A lot of information that is printed on top of maps would be better of in another type of chart. If we compare data of the five largest cities in the US, we don't need to do that on a map, everyone knows where New York, Los Angeles, Chicago, Houston, and Philadelphia are, but if we plot this on a map we give up our most important visual channel: position. We're no longer free to place things where we want!

But let's get to how we do maps with D3. Generally, there are two approaches:


 1. **Street Map with Data**: If you want to show something in the context of a real street map, your best bet is to use something like the [Google Maps API](https://developers.google.com/maps/?hl=en) - [here's an example of how it's used with D3](http://bl.ocks.org/mbostock/899711), or the [OpenStreetMap API](http://wiki.openstreetmap.org/wiki/API). You can use D3 to draw things on top of those, but you'll mainly work with the API provided by the vendor.

 2. **Data Maps**: If you want to present data on an abstract map, e.g., only showing counties or state borders, D3 is the way to go! Data maps are mostly used for when you want to communicate specific information or trends. In these maps you have full control over how the maps is colored, and how to encode information onto the map. 


 We'll be taking about both street maps and data maps, giving examples of how to go about using each one.

### Street Maps 
***

 Let's start off with street maps. These are used when the spatial context of your data is very import. That is, you care about the ability to navigate to a specific geographic location. You also get out of the box features such as zooming and panning. An example of a dataset where street maps would be a good choice would be visualizing yelp reviews on a map. 

 The most common street map used is the [Google Maps API](https://developers.google.com/maps/documentation/javascript/tutorial). 

 The key steps in getting a simple google map up and running on your web page are as follows: 

1. **[Get an API Key for use with Google Maps](https://developers.google.com/maps/documentation/javascript/get-api-key)** 

2. **Load the Google Map Javascript API using the ``<script>`` tag and using the newly created API key.** 

3. **Create a div element with an id that will be hold the map**

4. **Call the google.maps.Map endpoint to create a map inside our newly created div.**

Let's see this in action: 

{% include code.html id="google_map" file="google_map.html" code="" js="false" preview="true" %}


####  The Map Object

``` JS
map = new google.maps.Map(document.getElementById("map"), {...});
```

The map object defines a single map on the page. You must create a new object for each new instance of a map that you want on the page.  The parameters for the map constructor are as follows: 

``` Map(mapDiv:Node, opts?:MapOptions )	``` 

where mapDiv is the DIV element where you want the map to live and MapOptions are the parameters that are used for creating this map. Of these parameters, only two are required: **center, and zoom**. 

``` JS
map = new google.maps.Map(document.getElementById('map'), {
  center: {lat: -34.397, lng: 150.644},
  zoom: 8,
  mapTypeId: 'terrain' // Optional
});
```

#### Zoom Levels 

When you're setting the zoom levels programatically, it can be helpful to know how the numeric zoom level translates to the amount of detail your user can see in the map. Here is a helpful conversion table. 

Zoom Level  | Level of Detail           
------|------------------
1  	| World   
5  | Landmass/Continent   
10 | City    
15 | Streets 
20 | Buildings 

#### Map Types 

The following map types are available in the API:

1. `roadmap` displays the default road map view. This is the default map type.
2. `satellite` displays Google Earth satellite images
3. `hybrid` displays a mixture of normal and satellite views
4. `terrain` displays a physical map based on terrain information.

You can also set the mapTypeID programatically (say as a result of a user action) with : 

``` JS
map.setMapTypeId('terrain');
```

### Styled Maps

Other than these basic customizations, you can go the extra mile and really change the look and feel of your google map with [Styled Maps](https://developers.google.com/maps/documentation/javascript/styling). 


#### Adding D3 to Google Maps

Once you have a google map set up, you might want to add a layer with your d3 visualization of geolocated markers. Let's step through an example of how to do that. 


Walk through d3 v4 version of [this example](https://bl.ocks.org/mbostock/899711)


### Data Maps 
***


Now that we've covered using the Google Maps API, let's talk about creating maps using purely D3. These maps are usually made with the intent of showing the distribution of data that has a meaningful geographic component. A common example are political maps that show the republican/democrat tendencies as a function of state. In these maps, the specific data and trends are the focus of the visualization. 


Before we jump into rendering the map itself, let's take a look at the format in which geographic data is ususally handled on the web: geoJSON and topoJSON. 

#### GeoJSON/TopoJSON

The [GeoJSON format](http://geojson.org/) describes the contained geography as a combination of longitude and latitude coordinates, so that each entry forms a polygon.  

[TopoJSON variety](https://github.com/mbostock/topojson/wiki) is a topological geospatial data interchange format based on GeoJSON. 


1. TopoJSON is an extension of GeoJSON
2. Rather than representing geometries discretely, geometries in TopoJSON files are stitched together from shared line segments called arcs.
3. TopoJSON eliminates redundancy, offering much more compact representations of geometry than with GeoJSON;
4. Typical TopoJSON files are 80% smaller than their GeoJSON equivalents.


Because D3 only handles data in the GeoJSON format, there is a d3 library that does the job of converting TopoJSON to GeoJSON: 

``` JS
<script src="http://d3js.org/topojson.v1.min.js"></script>

``` 

We'll be taking a closer look at how to do this conversion in an example below. 


Let's take a closer look at what a GeoJSON file looks like. Here is a sample from the [data file containing US states](us-states.json).


{% highlight javascript linenos %}
{
     "type":  "FeatureCollection",
     "features":
     [
         {
             "type": "Feature",
             "id": "01",
             "properties": {"name": "Alabama"},
             "geometry": {
                "type": "Polygon",
                "coordinates": [[[-87.359296, 35.00118], [-85.606675, 34.984749], [-85.431413,34.124869],[-85.184951,32.859696], ...
 }
 {% endhighlight %}

 You can see that the coordinates are within the geometry object, and that the properties tell us that this is the shape representing Alabama.
 
#### Using Projections 

 You might be able to tell that the coordinates above use latitude and longitude - which are spherical coordinates! Mapping these onto a 2D surface like your screen requires a projection. There are many projections, with various advantages and disadvantages - we'll talk about them in class.
  
 Here is an example of how to use projections to transform lat/lon values into screen coordinate pixel values: 
 
 
 ``` JS
 
 let projection = d3.geoAlbersUsa() 
 
 let d={lat:-30,lon:111};
 
 let[cx,cy]= projection([d.lon, d.lat])
 
 ```
 
 D3 supports a long list of projections, including: 
 
* d3.geoAlbers - the Albers equal-area conic projection.
* d3.geoAlbersUsa - a composite Albers projection for the United States.
* d3.geoAzimuthalEqualArea - the azimuthal equal-area projection.
* d3.geoAzimuthalEquidistant - the azimuthal equidistant projection.
* d3.geoConicConformal - the conic conformal projection.
* d3.geoConicEqualArea - the conic equal-area (Albers) projection.
* d3.geoConicEquidistant - the conic equidistant projection.
* conic.parallels - set the two standard parallels.
* d3.geoEquirectangular - the equirectangular (plate carreé) projection.
* d3.geoGnomonic - the gnomonic projection.
* d3.geoMercator - the spherical Mercator projection.

[Here is a showreel of all the projections supported by D3](http://bl.ocks.org/mbostock/3711652).


 Once projected to screen coordinates, the polygons can be easily converted into an SVG path with `d3.geoPath()` (as always, see the [API documentation here](https://github.com/d3/d3-geo/blob/master/README.md#geoPath)).
 
 the d3.geoPath() function is an SVG path generator that takes in several different types of GeoJSON objects: 
 
1. Point - a single position.
2. MultiPoint - an array of positions.
3. LineString - an array of positions forming a continuous line.
4. MultiLineString - an array of arrays of positions forming several lines.
5. Polygon - an array of arrays of positions forming a polygon (possibly with holes).
6. MultiPolygon - a multidimensional array of positions forming multiple polygons.
7. GeometryCollection - an array of geometry objects.
8. Feature - a feature containing one of the above geometry objects.
9. **FeatureCollection - an array of feature objects.**
10. **Feature - a feature containing one of the above geometry objects.**

For the purpose of this example we are going to give examples that use the Feature Collection. 



#### Adding markers on top of D3 Map


Here is an example for how we can draw marks on top of maps, in this case the size of cities:

{% include code.html id="d3_cities" file="d3_cities.html" code="" js="false" preview="true" %}

Here is an example for a choropleth map, coloring each state by its agricultural output. The trick here is to join the data about the ouptut to the geography information:

{% include code.html id="d3_choropleth" file="d3_choropleth.html" code="" js="false" preview="true" %}

## Next Steps

This concludes our introduction to D3 and JavaScript. We will (probably) have another lecture on designing larger systems, event handling, etc in the next couple of weeks.