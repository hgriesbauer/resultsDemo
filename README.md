Querying RESULTS silviculture data using R
================
Hardy Griesbauer
06/04/2020

<!--

  Copyright 2019 Province of British Columbia

  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and limitations under the License.

-->

## Introduction

### bcdata and bcmaps R packages

The `bcdata` and `bcmaps` packages now provide an easy way to query,
download and visualize some BC government datasets through the R
interface. This short vignette will demonstrate a few ways that we can
do this using RESULTS and other datasets. These packages have been
developed by Andy Teucher, Sam Albers, Stephanie Hazlitt and others, and
more information can be found here: <https://bcgov.github.io/bcdata/>

While BC government data can usually be downloaded through your browser
at catalogue.data.gov.bc.ca, I think there are some advantages to using
the `bcdata` package to download data form within R:

1.  You are not limited to download limits;
2.  You can query the data before downloading to extract only what you
    need;
3.  After downloading, you can quickly work with the data in R; and
4.  Development team is very responsive and great to work with\!

## Getting started

First thing we need to do is make sure we have some packages installed.
If you don’t have the package already installed on your machine, run the
following code to install from CRAN:

``` r
# Install bcdata from CRAN
install.packages("bcdata")

# Install bcmaps from CRAN
install.packages("bcmaps")
```

To start the demo, let’s load some other libraries:

``` r
library(tidyverse)
library(bcdata)
library(mapview)
library(bcmaps)
library(bcmapsdata)
library(here)
```

### Different ways to use the bcdata package

The main functions in `bcdata` are:

1.  bcdc\_browse() - Open the catalogue in your default browser
2.  bcdc\_search() - Search records in the catalogue
3.  bcdc\_search\_facets() - List catalogue facet search options
4.  bcdc\_get\_record() - Print a catalogue record
5.  bcdc\_get\_data() - Get catalogue data
6.  bcdc\_query\_geodata() - Get & query catalogue geospatial data
    available through a Web Service

In general, I use an iterative workflow with these packages:

1.  Search for datasets in the catalogue;
2.  Query the dataset to understand the data structure;
3.  Filter for records of interest; and
4.  Download the data into R

<!-- end list -->

``` r
bcdc_search("hydrology")
```

    ## Found 101 matches. Returning the first 100.
    ## To see them all, rerun the search and set the 'n' argument to 101.

    ## List of B.C. Data Catalogue Records
    ## 
    ## Number of records: 100 (Showing the top 10)
    ## Titles:
    ## 1: Hydrology: Hydrologic Zone Boundaries of British Columbia (other, wms, kml)
    ##  ID: 329fd234-8835-4d44-9aaa-97c37bfc8d92
    ##  Name: hydrology-hydrologic-zone-boundaries-of-british-columbia
    ## 2: Hydrology: Hydrometric Watershed Boundaries (other, wms, kml)
    ##  ID: 02c0e328-e871-4d05-a672-8faf99ebfc11
    ##  Name: hydrology-hydrometric-watershed-boundaries
    ## 3: Hydrology: Low Flow Zones (other, wms, kml)
    ##  ID: 83b8cb98-211e-4b8f-bc56-e6e9ec1c39d8
    ##  Name: hydrology-low-flow-zones
    ## 4: Hydrology: Normal Annual Runoff Isolines (1961 - 1990) - Historical (other, wms, kml)
    ##  ID: d6f6ddc7-fbc2-4264-aa30-fc3d5138a6b3
    ##  Name: hydrology-normal-annual-runoff-isolines-1961-1990-historical
    ## 5: Atlas of Canada 1,000,000 National Frameworks Data - Hydrology Lakes (other)
    ##  ID: 04db6ca8-e0c6-4e37-9a86-65f92d00f237
    ##  Name: atlas-of-canada-1-000-000-national-frameworks-data-hydrology-lakes
    ## 6: Hydrology: 10 Year Peak Flow Isolines (Historical) (other, wms, kml)
    ##  ID: fed6d845-b7ff-4512-8f5f-595420cc43e8
    ##  Name: hydrology-10-year-peak-flow-isolines-historical
    ## 7: Hydrology: 100 Year Peak Flow Isolines (Historical) (other, wms, kml)
    ##  ID: dd7d7ff3-9512-499a-ae55-56beedfbbf9c
    ##  Name: hydrology-100-year-peak-flow-isolines-historical
    ## 8: Atlas of Canada 1,000,000 National Frameworks Data - Roads (other)
    ##  ID: a960b4a4-50ac-4a39-9bae-b11b7e8b418c
    ##  Name: atlas-of-canada-1-000-000-national-frameworks-data-roads
    ## 9: University of Victoria Aquifer Stress Evaluation (xlsx)
    ##  ID: 17ffdf71-28f3-4a65-bba2-134622b50e8f
    ##  Name: university-of-victoria-aquifer-stress-evaluation
    ## 10: Indicator Summary Data: Change in Timing and Volume of River Flow in BC (1912-2012) (csv)
    ##  ID: d6f30634-a6a8-45b5-808e-210036f25044
    ##  Name: indicator-summary-data-change-in-timing-and-volume-of-river-flow-in-bc-1912-2012- 
    ## 
    ## Access a single record by calling bcdc_get_record(ID)
    ##       with the ID from the desired record.

Records with *wms* are spatial files that can be queried and downloaded
using `bcdc_query_geodata`.

### Silviculture data in BC

Silviculture information in BC is largely collected and tracked through
the RESULTS database (Reporting Silviculture Updates and Land Status
Tracking System). The database stores silviculture-related information
on the management of openings, disturbances, silviculture activities and
obligation declarations as required by the Forest and Range Practices
Act. Data in RESULTS are publically available through
catalogue.data.gov.bc.ca. More information on RESULTS can be found here:
<https://www2.gov.bc.ca/gov/content/industry/forestry/managing-our-forest-resources/silviculture/silviculture-reporting-results>

## Application of bcdata and bcmaps to RESULTS data

### Reforestation with climate considerations in the Prince George District

The Omineca Region’s climate action plan has targets around
reforestation with tree species that will likely remain well-adapted to
projected future climates. Douglas-fir and western larch have been
identified as tree species that will remain viable into the future.

*Question: how much western larch is being planted in the Prince George
District?*

We can gather information to answer this question using RESULTS -
silviculture forest cover dataset. First, let’s take a look at the
dataset contents using
    `bcdc_query_geodata`:

``` r
bcdc_query_geodata("results-forest-cover-silviculture") 
```

    ## Warning: It is advised to use the permanent id ('258bb088-4113-47b1-b568-ce20bd64e3e3') rather than the name of the record ('results-forest-cover-silviculture') to guard against future name changes.

    ## Querying 'results-forest-cover-silviculture' record
    ## * Using collect() on this object will return 866981 features and 159 fields
    ## * At most six rows of the record are printed here
    ## ------------------------------
    ## Simple feature collection with 6 features and 159 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: 1147890 ymin: 676633.7 xmax: 1411085 ymax: 1083545
    ## epsg (SRID):    3005
    ## proj4string:    +proj=aea +lat_1=50 +lat_2=58.5 +lat_0=45 +lon_0=-126 +x_0=1000000 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs
    ## # A tibble: 6 x 160
    ##   id    FOREST_COVER_ID STOCKING_STANDA~ OPENING_ID STANDARDS_UNIT_~
    ##   <chr>           <int>            <int>      <int> <chr>           
    ## 1 WHSE~         3994018          1805957    1486231 2               
    ## 2 WHSE~         3994007               NA    1248495 <NA>            
    ## 3 WHSE~         3993995          2263260    1728926 1               
    ## 4 WHSE~         3994067               NA    1120935 <NA>            
    ## 5 WHSE~         3994009          1404558    1248495 A               
    ## 6 WHSE~         3994057          1211894    1120935 B               
    ## # ... with 155 more variables: SILV_POLYGON_NUMBER <chr>,
    ## #   SILV_POLYGON_AREA <dbl>, SILV_POLYGON_NET_AREA <dbl>,
    ## #   SILV_NON_MAPPED_AREA <int>, STOCKING_STATUS_CODE <chr>,
    ## #   STOCKING_TYPE_CODE <chr>, STOCKING_CLASS_CODE <chr>,
    ## #   SILV_RESERVE_CODE <chr>, SILV_RESERVE_OBJECTIVE_CODE <chr>,
    ## #   TREE_COVER_PATTERN_CODE <chr>, REENTRY_YEAR <chr>,
    ## #   REFERENCE_YEAR <int>, SITE_INDEX <int>, SITE_INDEX_SOURCE_CODE <chr>,
    ## #   BGC_ZONE_CODE <chr>, BGC_SUBZONE_CODE <chr>, BGC_VARIANT <chr>,
    ## #   BGC_PHASE <chr>, BEC_SITE_SERIES <chr>, BEC_SITE_TYPE <chr>,
    ## #   BEC_SERAL <chr>, IS_SILV_IMPLIED_IND <chr>,
    ## #   FOREST_COVER_SILV_TYPE <chr>, S_FOREST_COVER_LAYER_ID <int>,
    ## #   S_TOTAL_STEMS_PER_HA <chr>, S_TOTAL_WELL_SPACED_STEMS_HA <int>,
    ## #   S_WELL_SPACED_STEMS_PER_HA <int>, S_FREE_GROWING_STEMS_PER_HA <int>,
    ## #   S_CROWN_CLOSURE_PERCENT <chr>, S_BASAL_AREA <chr>,
    ## #   S_SPECIES_CODE_1 <chr>, S_SPECIES_PERCENT_1 <int>,
    ## #   S_SPECIES_AGE_1 <int>, S_SPECIES_HEIGHT_1 <dbl>,
    ## #   S_SPECIES_CODE_2 <chr>, S_SPECIES_PERCENT_2 <int>,
    ## #   S_SPECIES_AGE_2 <chr>, S_SPECIES_HEIGHT_2 <chr>,
    ## #   S_SPECIES_CODE_3 <chr>, S_SPECIES_PERCENT_3 <int>,
    ## #   S_SPECIES_CODE_4 <chr>, S_SPECIES_PERCENT_4 <chr>,
    ## #   S_SPECIES_CODE_5 <chr>, S_SPECIES_PERCENT_5 <chr>,
    ## #   S_MORE_SPECIES_EXIST_IND <chr>, S_SILV_LABEL <chr>,
    ## #   S1_FOREST_COVER_LAYER_ID <int>, S1_TOTAL_STEMS_PER_HA <int>,
    ## #   S1_TOTAL_WELL_SPACED_STEMS_HA <int>,
    ## #   S1_WELL_SPACED_STEMS_PER_HA <int>, S1_FREE_GROWING_STEMS_PER_HA <int>,
    ## #   S1_CROWN_CLOSURE_PERCENT <int>, S1_BASAL_AREA <chr>,
    ## #   S1_SPECIES_CODE_1 <chr>, S1_SPECIES_PERCENT_1 <int>,
    ## #   S1_SPECIES_AGE_1 <int>, S1_SPECIES_HEIGHT_1 <int>,
    ## #   S1_SPECIES_CODE_2 <chr>, S1_SPECIES_PERCENT_2 <chr>,
    ## #   S1_SPECIES_AGE_2 <chr>, S1_SPECIES_HEIGHT_2 <chr>,
    ## #   S1_SPECIES_CODE_3 <chr>, S1_SPECIES_PERCENT_3 <chr>,
    ## #   S1_SPECIES_CODE_4 <chr>, S1_SPECIES_PERCENT_4 <chr>,
    ## #   S1_SPECIES_CODE_5 <chr>, S1_SPECIES_PERCENT_5 <chr>,
    ## #   S1_MORE_SPECIES_EXIST_IND <chr>, S1_SILV_LABEL <chr>,
    ## #   S2_FOREST_COVER_LAYER_ID <int>, S2_TOTAL_STEMS_PER_HA <int>,
    ## #   S2_TOTAL_WELL_SPACED_STEMS_HA <int>,
    ## #   S2_WELL_SPACED_STEMS_PER_HA <int>, S2_FREE_GROWING_STEMS_PER_HA <int>,
    ## #   S2_CROWN_CLOSURE_PERCENT <int>, S2_BASAL_AREA <chr>,
    ## #   S2_SPECIES_CODE_1 <chr>, S2_SPECIES_PERCENT_1 <int>,
    ## #   S2_SPECIES_AGE_1 <int>, S2_SPECIES_HEIGHT_1 <int>,
    ## #   S2_SPECIES_CODE_2 <chr>, S2_SPECIES_PERCENT_2 <chr>,
    ## #   S2_SPECIES_AGE_2 <chr>, S2_SPECIES_HEIGHT_2 <chr>,
    ## #   S2_SPECIES_CODE_3 <chr>, S2_SPECIES_PERCENT_3 <chr>,
    ## #   S2_SPECIES_CODE_4 <chr>, S2_SPECIES_PERCENT_4 <chr>,
    ## #   S2_SPECIES_CODE_5 <chr>, S2_SPECIES_PERCENT_5 <chr>,
    ## #   S2_MORE_SPECIES_EXIST_IND <chr>, S2_SILV_LABEL <chr>,
    ## #   S3_FOREST_COVER_LAYER_ID <int>, S3_TOTAL_STEMS_PER_HA <int>,
    ## #   S3_TOTAL_WELL_SPACED_STEMS_HA <int>,
    ## #   S3_WELL_SPACED_STEMS_PER_HA <int>, S3_FREE_GROWING_STEMS_PER_HA <int>,
    ## #   S3_CROWN_CLOSURE_PERCENT <int>, S3_BASAL_AREA <chr>,
    ## #   S3_SPECIES_CODE_1 <chr>, ...

The output from this query shows that this dataset has 866,728 features
and 159 fields. Each feature is a treatment unit within a harvested
opening, and contains information on the leading five tree species that
are present in each treatment unit, including stems per hectare, age,
and height.

This dataset would be too large (\~1GB) to download efficiently. So,
let’s use `filter` to refine our query:

1.  Filter openings that are in Prince George District; and
2.  Filter treatment units that have Douglas-fir or western larch
    present.

First, let’s filter for openings in the PG District. We’ll use `bcdata`
package to do this by downloading a shapefile for the PG District and
use that as a bounding box:

``` r
# First, let's take a look at the natural resource district dataset
bcdc_query_geodata("natural-resource-nr-district")
```

    ## Querying 'natural-resource-nr-district' record
    ## * Using collect() on this object will return 23 features and 12 fields
    ## * At most six rows of the record are printed here
    ## ------------------------------
    ## Simple feature collection with 6 features and 12 fields
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: 831051 ymin: 455592.3 xmax: 1749980 ymax: 1368536
    ## epsg (SRID):    3005
    ## proj4string:    +proj=aea +lat_1=50 +lat_2=58.5 +lat_0=45 +lon_0=-126 +x_0=1000000 +y_0=0 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs
    ## # A tibble: 6 x 13
    ##   id    DISTRICT_NAME ORG_UNIT ORG_UNIT_NAME REGION_ORG_UNIT
    ##   <chr> <chr>         <chr>    <chr>         <chr>          
    ## 1 WHSE~ Cariboo-Chil~ DCC      Cariboo-Chil~ RCB            
    ## 2 WHSE~ Cascades Nat~ DCS      Cascades Nat~ RTO            
    ## 3 WHSE~ Sea to Sky N~ DSQ      Sea to Sky N~ RSC            
    ## 4 WHSE~ Stuart Necha~ DVA      Stuart Necha~ ROM            
    ## 5 WHSE~ Okanagan Shu~ DOS      Okanagan Shu~ RTO            
    ## 6 WHSE~ Selkirk Natu~ DSE      Selkirk Natu~ RKB            
    ## # ... with 8 more variables: REGION_ORG_UNIT_NAME <chr>,
    ## #   FEATURE_CODE <chr>, FEATURE_NAME <chr>, OBJECTID <int>,
    ## #   SE_ANNO_CAD_DATA <chr>, FEATURE_AREA_SQM <dbl>,
    ## #   FEATURE_LENGTH_M <dbl>, geometry <POLYGON [m]>

We can see that there are 23 NR districts in the province. District
codes are stored in the ‘ORG\_UNIT’ column Let’s filter districts for
DPG (Prince George) and download it as a shapefile.

``` r
dpg<-  # Create new variable called dpg
  bcdc_query_geodata("natural-resource-nr-district") %>%  # query the nr district dataset 
  filter(ORG_UNIT=="DPG") %>% # filter for Prince George District
  collect() # and download it 

# Alternatively, load from workspace
# load(here("data","dpg.RData"))
```

This is a small file, and shouldn’t take long to download. We can plot
it to double check:

``` r
dpg %>% 
  ggplot()+
  geom_sf()
```

![](README_files/figure-gfm/plot%20DPG-1.png)<!-- -->

Now we have a shapefile that we can use as a bounding box to filter and
download records in the RESULTS - silviculture layer. We’ll just
download openings that have larch planted, to make for a smaller file
size.

*Note: the chunk below can take a minute or so to run. To save time,
I’ve already run the chunk and saved the output to an .RData file,
which we’ll load into our workspace in a subsequent chunk*

``` r
# First, let's list what tree species we are interested in
 sppList=c("LW")
 
# Now let's filter the records and download the data

  treesDPG<- # create new variable by
    
    bcdc_query_geodata("results-forest-cover-silviculture") %>% # querying the results silviculture layer and
   
    filter(INTERSECTS(dpg)) %>% # filter for records that are within the DPG and
    
    filter(S_SPECIES_CODE_1 %in% sppList| # filter for our species within any of the five species per TU
            S_SPECIES_CODE_2 %in% sppList|
            S_SPECIES_CODE_3 %in% sppList|
            S_SPECIES_CODE_4 %in% sppList|
            S_SPECIES_CODE_5 %in% sppList) %>% 
    
    collect() # and download the file
```

Let’s load this dataset into our workspace:

``` r
load(here("data","treesDPG.RData"))

dim(treesDPG)
```

    ## [1] 169 162

There are 169 treatment units planted with larch in DPG.

Now we can plot the openings and see how things
look:

``` r
mapview(treesDPG,zcol="S_SILV_LABEL",legend=FALSE)+dpg
```

![](README_files/figure-gfm/map%20larch%20plantations%20in%20DPG-1.png)<!-- -->

### Summaries

We can also create some quick summaries of the data, treating the
attribute table as a data frame in R:

#### Question: what is the age distribution of larch plantations in DPG?

``` r
treesDPG %>% 
  st_drop_geometry() %>% # drop geometry
  mutate(age=2020-REFERENCE_YEAR+S_SPECIES_AGE_1) %>% 

  # plotting
  ggplot()+
  aes(x=age,y=FEATURE_AREA_SQM/10000)+
  geom_bar(stat="sum")+
  ylab("Sum treatment unit area (ha)")+
  theme(legend.position="none")+
  scale_x_continuous(name="Stand age", limits=c(2,30),
                     breaks=seq(from=5,to=30,by=5))+
  ggtitle("Area planted with larch by age in DPG, as of 2020")
```

![](README_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

#### Where are the older larch plantations in DPG?

``` r
treesDPG %>% 
  mutate(age=2020-REFERENCE_YEAR+S_SPECIES_AGE_1+1) %>% # create age column
  filter(age>20) %>% # filter for stand ages older than 20
  mapview(zcol="age") # map
```

![](README_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

#### Question: what is the BEC distribution of larch plantations in DPG?

The `bcmaps` package contains BEC spatial data, which can be used to
look at larch distribution by BGC unit in DPG. BEC data are loaded into
the workspace using the `bec()` function, and then can be joined to an
existing dataset using the `st_join` function. This takes awhile, so I
did it beforehand.

``` r
# First, download BEC data 
 becData<-bec() 

# Now join with dataset
treesDPG<-  
  st_join(treesDPG,bgcPG[,"MAP_LABEL"])
```

Let’s map blocks by BGC unit:

``` r
mapview(treesDPG,zcol="MAP_LABEL")
```

![](README_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

#### Summarize area planted with larch by BGC unit

``` r
treesDPG %>% 
  st_drop_geometry() %>% # drop geometry
  group_by(MAP_LABEL) %>% # group polygons by BGC unit
  summarise(Area=sum(FEATURE_AREA_SQM)/10000) %>% 
 
   # plotting
  ggplot()+
  aes(x=MAP_LABEL,y=Area)+
  geom_bar(stat="Identity")+
  ylab("Sum treatment unit area (ha)")+
  theme(legend.position="none")+
  ggtitle("Area planted with larch by BGC unit in DPG")+
  xlab("BGC unit")
```

![](README_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

-----
