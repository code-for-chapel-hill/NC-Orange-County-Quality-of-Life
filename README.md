Quality of Life Dashboard v2
=====================

A joint project between the City of Charlotte, Mecklenburg County, and UNCC. The first release of the dashboard is now in the [v1](https://github.com/tobinbradley/Mecklenburg-County-Quality-of-Life-Dashboard/tree/v1) branch.

The current (master) version is ready for production, but we decided to add a new calculation at the last minute requiring new data deliverables and code before we launch. You can see a reasonably current verision of the pre-launch site [here](http://mcmap.org/qol). Our launch date is looking like May 2015.

The dashboard uses D3 for visualizations. The good news is besides all the cool features in the new version has it is less than half the size over the wire and loads twice as fast. The bad news is IE8 absolutely will not work. I'm sorry/you're welcome.

Here's a handy [YouTube Tutorial](https://www.youtube.com/watch?v=9p9onu_e4Og) on customizing the Dashboard for your area of interest if reading isn't your bag.

We hope you find this project useful. Patches are always welcome!


## Setting Up

### Install node
Installing [node](http://nodejs.org/) is a piece of cake. On Windows, download the install file and click Next until it stops asking you to click Next. On MacOS I have no idea, but there's probably a draggy-droppy involved. Any Linux distro will have nodejs in its repos, so do a `sudo pacman -S nodejs` or `sudo apt-get install nodejs` or whatever package management magic you system employs.

To make sure everything went well here, when you're done installing pull up a terminal (that's DOS for you Windows types) and type `node --version`. You should get some like `v0.10.28`.

### Set up the project
I'm using [gulp](http://gulpjs.com/) as the build/dev system, because awesome. We'll be using [topojson](https://github.com/mbostock/topojson) to encode our geography.


``` terminal
npm install -g gulp topojson
git clone https://github.com/tobinbradley/Mecklenburg-County-Quality-of-Life-Dashboard.git
cd Mecklenburg-County-Quality-of-Life-Dashboard
npm install
```

### Grab some data
You'll need data and app configuration information to get rolling. From your project directory, pull in the data and generate your configuration.

``` terminal
git clone https://github.com/tobinbradley/mecklenburg-quality-of-life-data.git data
gulp datagen
```

### Finishing touches
The search function is set up to use Mecklenburg's search API's and won't work for other areas. To swap Mecklenburg's search out for a generic search that will work for any area:

``` terminal
gulp init
```

Finally, build the project.

``` terminal
gulp build
```

### Fire it up!
The default gulp task starts [BrowserSync](https://github.com/BrowserSync/browser-sync) and launches your current web browser to view the site. Live reload is enabled, so changes will automatically refresh in your browser.

``` terminal
gulp
```

## Customizing the Dashboard
Data in the dashboard comes in three pieces:

<div>
<img src="http://i.imgur.com/2WAS10m.png" style="max-width: 100%">
</div>

* The neighborhood geography topojson with your neighborhood id.
* The metric data. You will need 1 to 4 files depending on the calculation and what you want to show.
* The metric metadata.

### Topojson
The first thing you'll need to do is convert your neighborhood layer to topojson. There are a lot of options during conversion, which you can peruse [here](https://github.com/mbostock/topojson/wiki/Command-Line-Reference). You should start out with an Esri shapefile or geojson file projected to WGS84 (EPSG:4326). You can preview, manipulate, and project your data with [QGIS](http://www.qgis.org/).

Here's a good general call to create your topojson file, but do play around with the options if you want more or less line generalization:

    topojson -o geography.topo.json -s 7e-11 --id-property=id_field your_shapefile.shp

With `id_field` being the field in the shapefile you want to use for your neighborhood identifier. Copy that file into `data`. Make note of what your shapefile was named - you'll need that information when you update `config.js`.

### Metrics
Metric files are simple CSV files named to reflect what they are. The are stored in `data/metric`. The format is always the same:

     id,y_2012,y_2014

 With `y_XXXX` representing the year of that data column. Null values are empty, like:

     id,y_2012,y_2014
     12,,400

Depending on what you want to do with the metric, you will name your files slightly different things. There are three types of metrics, with the type being set in `config.js`. Each type will look for different file names.

* **sum**: This is for raw variables, like population. It will look for a file name beginning with *r*, like `r[X].csv`.
* **mean**: This is for normalized variables, like population density. It will look for a file name beginning with *n*, like `n[X].csv`.
* **weighted**: This will normalize data or perform a weighted average and will look for two files - a raw file beginning with *r*, like `r[X].csv`, and a weighting variable beginning with *d*, like `d[X].csv`. Suppose you wanted to have a weighted average for people per acre (population density). If you specify normalize, it'll look for a raw population in `rX.csv` and the number of acres for the denominator in `dX.csv` and produce a weighted average.

The CSV's are converted to JSON by running `gulp datagen`.

**Tip 1: CSV column case sensitivity**

Spreadsheet software often likes to capitalize the first letter in the `y_XXXX` field. That will turn your life into a furious ball of nothing. You can all your files at once on Linux like so:

     for f in *.csv; do sed -e 's/\(.*\)/\L\1/' $f > out/$f; done

**Tip 2: Sort on the ID**

Your CSV files should be sorted on the metric ID. On Linux to auto do this to all your files:

    export LC_NUMERIC=C
    for f in *.csv; do sort -n $f > out/$f; done???

**Tip 3: Beware the hanging zero**

Some identifiers like Census tracts can have a hanging zero, like 541.10. If you are manipulating your data files in spreadsheet software, make sure your ID column is being treated as a string and not a number or it will get dropped and your life will be destroyed.


### Metadata
Metadata files are in markdown and are named for the metric, like `m1.md`, and are located in `data/meta`. Your metadata file needs to maintain a structure with h2 and h3's laid out like this:

    ## Title of Metric
    Median age of poodles

    ### Whis is this important?
    Because we like poodles.

    ### About the Data
    I hang out at dog parks and type stuff in my phone. Circa 1986.

    ### Additional Resources
    Dog pound yo.

When processing the HTML from the markdown, the h2 and h3 tags are used in a really awful lefty-righty kind of way. If you insert another h2 or h3, you will be boned. Same if you drop one. If you want to format the metadata differently, you must edit `src/scripts/functions/metadata.js` to meet your needs. It won't be hard, but not doing it and jacking with the markdown will lead to disappointment.

After you add metadata, run `gulp datagen` to convert the markdown to HTML.

**Tip: Don't add additional H2 (##) or H3 (###) tags**

Because we're using those as choppers for layout, adding more of those will screw up your formatting.

**Tip: Beware weird, non-web safe characters characters**

Don't edit Markdown in Word. You're welcome.

### Customize config.js and site.json
`data/config/config.js` has knobs you will need to turn to set up the dashboard for your area. It is all well documented there. Each metric has a JSON description with a few required and many optional properties. That are documented in the file.

With `data/config/site.json` you can change a number of parameters in the site itself, like titles, links, etc.

*Note the weighted type doesn't work yet. It doesn't break, it just does the same thing as mean.*

### Odds and ends
You will have a few additional things to fiddle with, all of which you'll find in either `src/index.html` or `src/report.html`. Different app titles, logos, making some custom charts for your report - that kind of thing. It'll be pretty easy. At the bottom of report.html you will find example chart templates which will require a bit more thought, but all of the chart properties are logically laid out in data attributes on the canvas elements.

## Build for deployment
Building for deployment does all of the niceties for you - JavaScript concatenation and minification, LESS preprocessing/auto-prefixing/minification, Markdown conversion, CSV to JSON conversion, image optimzation, and cache busting. From the root folder run:

    gulp build

Copy the contents of the `dist` folder to your production web server and you're good to go.

## Unit Tests
There are a series of unit tests for the calculations to make sure the outputs are correct and that nulls are handled appropriately. These execute via [QUnit](http://qunitjs.com/) and can be fired via gulp.

``` bash
gulp test
```
