# Creating your first satellite image

In this hands-on training, you'll learn how to find, download, combine, and turn data captured by satellites into ready-for-publication images. You'll learn three ways to do this by using point and click tools, command line and Google Earth Engine.

[Best tutorial on using photoshop to process Landsat](https://earthobservatory.nasa.gov/blogs/elegantfigures/2013/10/22/how-to-make-a-true-color-landsat-8-image/)

[Best tutorial on using landsat util to process Landsat](https://www.developmentseed.org/blog/2014/08/29/landsat-util/)

## First things first

Request access to Google Earth Engine code environment

https://signup.earthengine.google.com/#!/

The machines we used already had the following required software installed:
* [Landsat Util](https://pythonhosted.org/landsat-util/installation.html)
* [Adobe Photoshop](https://www.adobe.com/products/photoshop/free-trial-download.html)

The Photoshop process can be achived in GIMP as well, but the method is very different

## Where does this data come from

We didn't do this in the session but these tools provide various ways to download Landsat scenes. I think Libra is the easiest to use.

* [Libra from Development Seed](https://libra.developmentseed.org)
* [Landsat Util](https://pythonhosted.org/landsat-util/)
* [Earth Explorer](https://earthexplorer.usgs.gov/)

If you're doing this at home, you'll have a few more files in your directory than we used during the session.

## Point and click with photoshop

1. Open the three images in photoshop
	* You want the X_B2.tif X_B3.tif and X_B4.tif files 

2. Open the channels panel

3. Click the menu button on that panel and select "Merge Channels..."

4. Select "RGB Color" from mode, and keep channels at 3, click okay

5. Specify your channels 
	* Red: X_B4.tif
	* Green: X_B3.tif
	* Blue: X_B2.tif

	Click OK

6. Your image is really dark. Lighten it up by going to `Image > Adjustments > Levels`
	take the white carrot and drag it to the edge of the histogram, click okay

7. Open the layers panel, create a new Curves adjustment layer by clicking the half-filled circle button at the bottom of the panel.

8. Zoom to something that looks like it's white, click the white eye dropper in the panel, then click the white area perhaps a cloud or building roof.

9. In panel, select each channel from the RGB drop down, and drag the black carrot for each to the point on the histogram where the curve gets steep.

10. Switch back to the RGB view in the histogram and pull the middle of the white line up

Using this method will remove all of the geographic metadata from your image. [Here is a way to get it back](https://gis.stackexchange.com/a/108703) using the command line tool GDAL.

## On the command line with landsat-util

1. Open up the command line and navigate to the folder with your images

2. run `landsat process LC80230312014138LGN00 --pansharpen --bands 432`

3. the output tells you where your file is located typically in your home directory inside of a folder at `landsat/processed/{landsat ID}`


## Using Google Earth Engine

1. Navigate to https://code.earthengine.google.com/

2. Create a new script

3. Pick a point you want to focus on

4. Define your parameters

```
var o = {
  start: ee.Date('2013-01-01'),
  finish: ee.Date('2018-03-10'),
  target: geometry,
  cloud_cover_lt: 0.8,
  bands:["B4", "B3", "B2"]
}
```

5. Load all of the landsat scenes

```
var filteredCollection = ee.ImageCollection("LANDSAT/LC08/C01/T1_RT")
```

6. Filter to the one that is over your point

```
var filteredCollection = ee.ImageCollection("LANDSAT/LC08/C01/T1_RT")
  .filterBounds(o.target)

```


7. Limit to those that have low cloud coverage

```
var filteredCollection = ee.ImageCollection("LANDSAT/LC08/C01/T1_RT")
  .filterBounds(o.target)
  .filterMetadata("CLOUD_COVER", "less_than", o.cloud_cover_lt)
```

8. Pick the most recent of those

```
var scene = ee.Image(filteredCollection.sort("DATE_ACQUIRED", false).first());
```

9. Set your parameters based on the data

You can do it this way by manually defining the extent of your data

```
var params = {
  bands: o.bands,
  max: [10000,10000,10000],
  min: [1000,1000,1000]
}

```


More complexly you can try to do it automatically by programatically calculating the extent of your data

```
var bandMax = filteredCollection.median()
    .reduceRegion({
      geometry: o.target,
      reducer: ee.Reducer.max(),
      scale: 60,
      tileScale: 16
    })
    
  var bandMin = filteredCollection.median()
    .reduceRegion({
      geometry: o.target,
      reducer: ee.Reducer.min(),
      scale: 60,
      tileScale: 16
    })


var params = {
  bands: o.bands,
  max: o.bands.map(function(b){return bandMax.get(b).getInfo() * 1.5}),
  min: o.bands.map(function(b){return bandMin.get(b).getInfo() * 0.15})
}

```

10. Plot your scene in the environment

```
Map.addLayer(scene, params)
```

11. Save to your google drive as a jpeg

```
var export_image = scene.visualize(params)

Export.image.toDrive({
    image: export_image,
    description: "my_scene_from_nicar",
    scale: 30,
    maxPixels: 240000000000
})
```

12. Click the "Run" button to exicute your code. In the "Tasks" panel clikc "Run" on the item created to begin the image export process.

Exporting images from Earth Engine can be slow. To speed it up, try drawing a shape on the map viewer and using it to crop with. Then update the export section to look a like this.

```
var export_image = scene.visualize(params)

Export.image.toDrive({
    image: export_image,
    description: "my_scene_from_nicar",
    scale: 30,
    region: geometry, #make sure this matches the name of whatever shape you drew 
    maxPixels: 240000000000
})
```


## Now what?

These are some stories that have used Landsat imagery to various ends. Some simply make use of true-color images like we made here. Some combine bands that capture reflections that are invisible to humans to detect vegetation health or highlight land use.

* [The island Bangladesh is thinking of putting refugees on is hardly an island at all](https://qz.com/1075444/the-island-bangladesh-is-thinking-of-putting-refugees-is-hardly-an-island-at-all/) by Quartz
* [Who is the Wet Prince of Bel Air? Here are the likely culprits](https://www.revealnews.org/article/who-is-the-wet-prince-of-bel-air-here-are-the-likely-culprits/) by Reveal
* [Welcome to Fabulous Las Vegas: While supplies last](https://projects.propublica.org/las-vegas-growth-map/) by Propublica
* [A Rogue State Along Two Rivers](https://www.nytimes.com/interactive/2014/07/03/world/middleeast/syria-iraq-isis-rogue-state-along-two-rivers.html) by The New York Times

