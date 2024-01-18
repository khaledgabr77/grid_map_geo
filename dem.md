# digital elevation model (DEM)

we go through steps to generate DME file and how to using with `grid map geo` package and inside our docker `d2dTracker_sim` contianer.

## How to download digital elevation model (DEM)

### Install QGIS

you will need to download `qgis` for `Debian/Ubuntu` you can follow the following or go [here](https://www.qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu)

First install some tools you will need for this instructions:

```bash
sudo apt install gnupg software-properties-common
```

Now install the QGIS Signing Key, so QGIS software from the QGIS repo will be trusted and installed:

```bash
sudo mkdir -m755 -p /etc/apt/keyrings  # not needed since apt version 2.4.0 like Debian 12 and Ubuntu 22 or newer
sudo wget -O /etc/apt/keyrings/qgis-archive-keyring.gpg https://download.qgis.org/downloads/qgis-archive-keyring.gpg
```

Add the QGIS repo for the latest stable QGIS (3.34.x Prizren) to `/etc/apt/sources.list.d/qgis.sources`:

```bash
Types: deb deb-src
URIs: https://qgis.org/debian
Suites: your-distributions-codename
Architectures: amd64
Components: main
Signed-By: /etc/apt/keyrings/qgis-archive-keyring.gpg
```

Update your repository information to reflect also the just added QGIS one:

```bash
sudo apt update
```

Now, install QGIS:

```bash
sudo apt install qgis qgis-plugin-grass
```

once installed `qgis` go to Earthdata website and register from this [link](https://urs.earthdata.nasa.gov//users/new) to create a new account, the account will give you access to install all plugins you need.


### download digital elevation model (DEM)

open `QGIS`:

```bash
sudo qgis
```

add map and srtm plugins:

### Map Plugin

select `plugins` from toolbar and seach for `QuickMapServices` and click on install plugin.

ensure that the plugin is installed by move to toolbar and select the `web` then `QuickMapServices` then move to `OSM` then select `OSM Standard`. you will find the map is loaded.

once the map loaded go to the place where you want to generate DEM and zoom the map until you find the place where you want to.

### SRTM Plugin

select `plugins` from toolbar and seach for `SRTM-Downloader` and click on install plugin.

ensure that the plugin is installed by move to toolbar and select the `plugins` then move to `SRTM-Downloader` select `SRTM-Downloader`.

this will open to you a new window, select `Set canvas extent` this will load to you coordinate `North`, `West`, `East` and `South`. you can edit the values if you want.

then select the `Output path` where you want to save your file.

once you finsihed click on `Download` wait until it is finished. it will ask you to added the `user name` and `passward` you register in `Earthdata` website.

after the download is finished will show the area where you selected with `gray` scale.

then move to `layers` in the left and right click on the generated map which will end with `.hgt` and select `properties`, this will open new window. 

select `symbology` and change the `rander type` to be `singleband pseudcolor`. 
then click on `Classify`, then click `ok` you will find the coverted to colored version.

once you finished you will need to save the DEM:
move to `layers` in the left and right click on the generated map which will end with `.hgt` right click and choose `Export` will open a new window select `Save As`.
first ensure the formate is `GeoTIFF`, then click `file name` and choose a new name to the file. then change the `GRS` to `Project CRS:...` then select `ok`.

you will now have a file with `.tif` extension.


### grid_map_geo

clone the `grid_map_geo` package:

```bash
git clone https://github.com/khaledgabr77/grid_map_geo.git
```

#### Setup

Install the dependencies. This package depends on gdal, to read georeferenced images and GeoTIFF files.

```bash
source /opt/ros/humble/setup.bash
rosdep update
# Assuming the package is cloned in the src folder of a ROS workspace...
rosdep install --from-paths src --ignore-src -y
```

Build the package

```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --packages-up-to grid_map_geo
```

the file you generated `.tif` will be High Resolution and will be to do some processing on the file. 

inside the terminal:

```bash
gdalwarp -ts <width> <height> <srcDEM> <targetDEM>
```
you can select small width and height like `129*129` or `150*150`

once you did the setup for the package you need to copy the generate file `<targetDEM>` to resources folder.

then open the `load_tif_launch.xml` launch file and change the `tif_path` to your file name `<targetDEM>`.

once you have finished you can Running the package

```bash
source install/setup.bash
ros2 launch grid_map_geo load_tif_launch.xml
```

i will leave my output file `kk_dem.tif` you can use it to launch the package. that is means you can run the package if you don't have `.tif` file.