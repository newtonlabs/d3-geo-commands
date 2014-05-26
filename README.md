# Directions to get SHP for topojson

* Goto [http://www.diva-gis.org/gData](diva-gis)
* Get the administration areas (USA_adm.zip)
* Convert SHP to JSON using all the properties to see what is available

```
topojson -o usa.json -p -- USA_adm2.shp
```

* Again but keep only the fields we are interested in
    * NAME_1 : State Name
    * NAME_2 : County Name
    * TYPE_2 : 'County'

```
topojson -o usa.json -p state=NAME_1,county=NAME_2 -- USA_adm2.shp
```

The data from diva-gis is too detailed at a quantized factor of 10,000.  Need to adjust this.  The smaller the lower the resolution.

Default of 1e4

bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 3.991km (0.0359°) 584m (0.00525°)
topology: 40231 arcs, 318583 points
prune: retained 34410 / 40231 arcs (86%)

Changing to 1e1

```
topojson -o usa.json -p state=NAME_1,county=NAME_2 -q 1e1 -- USA_adm2.shp
```
bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 4434.503km (39.9°) 648.400km (5.83°)
topology: 43 arcs, 97 points
prune: retained 18 / 43 arcs (42%)

Pay attention to the quantization.  This means 4km per pixel on the map resolution.  By changing to 1e1 or (10) we change the quantization to 4000km per pixel on the map dropping our file size from 2.8M to 1.1K


Changing to 1e3
topojson -o ../geo/usa_medium.json -p state=NAME_1,county=NAME_2 -q 5e3 -- USA_adm2.shp

bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 39.950km (0.359°) 5.841km (0.0525°)
topology: 16696 arcs, 51614 points
prune: retained 14369 / 16696 arcs (86%)

Bringing the file down to 1.9mb (362k gzip!)

Now lets simplify:

topojson -o ../geo/usa_medium_simple.json -s 6px -p state=NAME_1,county=NAME_2 -q 5e3 -- USA_adm2.shp; ls -lh ../geo

topojson -o ../geo/usa_medium_simple.json -s 9e-7 -p state=NAME_1,county=NAME_2 -q 5e3 -- USA_adm2.shp; ls -lh ../geo

# cleaner properties
topojson -o ../geo/usa_medium_simple.json -s 9e-7 -p state=NAME_1,county=NAME_2 -q 1e4 -- counties=USA_adm2.shp; ls -lh ../geo

# hmm lakes showing up .. lets add type so we can find them

topojson -o ../geo/usa_medium_simple.json -s 9e-7 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e4 -- counties=USA_adm2.shp; ls -lh ../geo

.attr("class", function(d) { return d.properties.type === 'Water body' ? 'water' : 'feature'})

# tweak for more detail
[~/src/mapnotes/gis]$ topojson -o ../geo/usa_medium_simple.json -s 7e-7 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e5 -- counties=USA_adm2.shp; ls -lh ../geo
bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 399m (0.00359°) 58.4m (0.000525°)
topology: 11668 arcs, 1059760 points
simplification: retained 32148 / 1059760 points (3%)
prune: retained 10822 / 11668 arcs (93%)

# at 40m
[~/src/mapnotes/gis]$ topojson -o ../geo/usa_medium_simple.json -s 7e-7 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e6 -- counties=USA_adm2.shp; ls -lh ../geo
bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 39.9m (0.000359°) 5.84m (0.0000525°)
topology: 9762 arcs, 1584573 points
simplification: retained 28484 / 1584573 points (2%)
prune: retained 9440 / 9762 arcs (97%)
total 22128

# more simple
[~/src/mapnotes/gis]$ topojson -o ../geo/usa_medium_simple.json -s 7e-6 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e6 -- counties=USA_adm2.shp; ls -lh ../geo
bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 39.9m (0.000359°) 5.84m (0.0000525°)
topology: 9424 arcs, 1338728 points
simplification: retained 19989 / 1338728 points (1%)
prune: retained 9151 / 9424 arcs (97%)
total 21936

# super simple!
[~/src/mapnotes/gis]$ topojson -o ../geo/usa_medium_simple.json -s 7e-4 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e6 -- counties=USA_adm2.shp; ls -lh ../geo
bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 39.9m (0.000359°) 5.84m (0.0000525°)
topology: 313 arcs, 34208 points
simplification: retained 632 / 34208 points (2%)
prune: retained 44 / 313 arcs (14%)
total 20608
-rw-r--r--  1 tnewton  staff   2.8M Apr 16 11:44 usa.json
-rw-r--r--  1 tnewton  staff   858K Apr 16 13:21 usa_census.json
-rw-r--r--  1 tnewton  staff   2.8M Apr 16 11:51 usa_default.json
-rw-r--r--  1 tnewton  staff   1.1K Apr 16 12:04 usa_low.json
-rw-r--r--  1 tnewton  staff   1.9M Apr 16 12:14 usa_medium.json
-rw-r--r--  1 tnewton  staff   2.8K Apr 16 14:14 usa_medium_simple.json
-rw-r--r--  1 tnewton  staff   1.7M Apr 16 13:47 usa_medium_simple_allprop.json

# above the trinagles are so simple you start loosing counties...

# final landing
[~/src/mapnotes/gis]$ topojson -o ../geo/usa_medium_simple.json -s 3e-6 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e6 -- counties=USA_adm2.shp; ls -lh ../geo
bounds: -179.15055799999988 18.909858000000042 179.77340800000002 71.39068500000002 (spherical)
quantization: 39.9m (0.000359°) 5.84m (0.0000525°)
topology: 9541 arcs, 1448518 points
simplification: retained 21755 / 1448518 points (2%)
prune: retained 9246 / 9541 arcs (97%)
total 21976
-rw-r--r--  1 tnewton  staff   2.8M Apr 16 11:44 usa.json
-rw-r--r--  1 tnewton  staff   858K Apr 16 13:21 usa_census.json
-rw-r--r--  1 tnewton  staff   2.8M Apr 16 11:51 usa_default.json
-rw-r--r--  1 tnewton  staff   1.1K Apr 16 12:04 usa_low.json
-rw-r--r--  1 tnewton  staff   1.9M Apr 16 12:14 usa_medium.json
-rw-r--r--  1 tnewton  staff   688K Apr 16 14:20 usa_medium_simple.json
-rw-r--r--  1 tnewton  staff   1.7M Apr 16 13:47 usa_medium_simple_allprop.json

# now add in states data

topojson -o ../geo/usa_medium_simple.json -s 3e-6 -p state=NAME_1,county=NAME_2,type=TYPE_2 -q 1e6 -- counties=USA_adm2.shp states=USA_adm1.shp; ls -lh ../geo

<!-- Getting some strange artifacts using this source.  Trying US census located heree -->

<!-- http://www2.census.gov/geo/tiger/TIGER2010DP1/County_2010Census_DP1.zip -->

<!-- -q 1e5 \
    -s 1 \
    --projection 'd3.geo.albersUsa()' \
    --id-property=GEOID10 \
    -p name=NAMELSAD10,pop=+DP0010001 \
    -o $@ \
    -- counties=County_2010Census_DP1.shp

node --max_old_space_size=8192 /usr/local/share/npm/bin/topojson -q 1e5 -s 1 --projection 'd3.geo.albersUsa()' --id-property=GEOID10 -p name=NAMELSAD10 -o usa.json -- counties=County_2010Census_DP1.shp -->






