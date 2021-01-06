# carla_maps
This is a repository for processing and storing map data for the CARLA open source vehicle simulator.

## Making Custom Maps for CARLA 

### Generating Custom Maps with OpenStreetMap (Generate maps with OpenStreetMap)
I started trying to build a custom town. This is one of my big goals for this project. I want to run CARLA in a virtual TNTECH Campus.

I have tried to follow this CARLA tutorial [here](https://carla.readthedocs.io/en/latest/tuto_G_openstreetmap/), but I am stuck. 

Progress so far (steps from tutorial in link above)

#### Step 1 -  Obtain a map with OpenStreetMap
I have exported two maps of TNTECH campus as `.osm` files from OpenStreetMap named `map.osm` and `map2.osm`. These are located in `carla/openstreetmap`
This can be done with the web app(openstreetmap.org), or you can get the data directly with the java app `josm` shown below.

#### Step 2 - Prepare Map with Third Party Tools
JAVA is required to run these tools. I am using  `openjdk-8-jre`  which is am older version than the deffault for Ubuntu18.
```
sudo apt-get install openjdk-8-jre
```

Alternatively you can use install `jdk8` with conda which is nice, but we have to use the Borg...

```
conda activate carla
```
```
conda install -c cidermole jdk8
```


##### - JOSM - Extensible Editor for OpenStreetMap(.osm)
This JAVA app can be used to download maps from OpenStreetMap end edit **.osm** files. Read more on the [josm Github](https://josm.openstreetmap.de/). Install the packge with with apt following the instructions [here](https://josm.openstreetmap.de/wiki/Download#Ubuntu)

Setup sources and download keys
```
sudo vim /etc/apt/sources.list
deb https://josm.openstreetmap.de/apt focal universe
wget -q https://josm.openstreetmap.de/josm-apt.key -O- | sudo apt-key add - 
```
Install JOSM 
```
sudo apt-get update
sudo apt-get remove josm josm-plugins

# tested version
sudo apt-get install josm
# development version
sudo apt-get install josm-latest
```
Run the app. Find information in the josm [wiki](https://josm.openstreetmap.de/wiki/Introduction) related to using the tool.
```
cd ~/openstreetmap/josm
java -jar josm.jar
```

JOSM is not working in a conda env right now. So does it require JDK8? 

#### Step 2 -  Convert OpenStreetMap (.osm) to OpenDRIVE format (.xodr) using CARLA
CARLA should be able to do this conversion. I made a script `convert_map.py` to convert the `.osm` file to a `.xodr` file using the sample code in the carla docs. I used `utils/config.py` as a template mainly for the imports lines. This step appears to work and the output file is produced. The line below runs the script
````
cd ~/carla_simulator/carla/PythonAPI/carla/
python3 ${CARLA_ROOT}/PythonAPI/util/convert_map.py
````
###### Angular Distortion Issue! - CARLA 0.9.11 may have solve this - needs testing
The conversion runs but the resulting map is distorted in an angular sense (~20-30 degrees) - not at all useable This is a known issue (https://github.com/carla-simulator/carla/issues/3009). The angular distortion issue can be avoided by using a [osm2xodr](https://github.com/JHMeusener/osm2xodr) instead of CARLA to convert from **.osm** to **.xodr**.

###### Cannot read local schema 
If you will recieve the error below, this is because you must be in the `carla/PythonAPI/carla/` directory to run this script. I assume this is just a path issue that can be addressed. For now, use `cd` as shown above. I found this here (insert link - go find post)
```
Warning: Cannot read local schema '../carla/data/xsd/types_file.xsd', will try website lookup.
```
#### Step 2 Alternate - Convert OpenStreetMap (.osm) to OpenDRIVE format (.xodr) using osm2xodr 

##### - osm2xodr 
This package can be used convert the **.osm** file to **.xodr** file which can be ingested by carla. [osm2xodr](https://github.com/JHMeusener/osm2xodr) 
This is a custom script that I made from the example that came with the package.
**PyProj** and **osmread** are required to for **osm2xodr**. I installed them in the conda environment as shown below.

```
conda activate carla09101
```
```
conda install -c conda-forge pyproj -n carla09101
```
```
pip3 install osmread
```

Run the cscript to convert the files. For now the filenames are hardcoded in the python script, but I would like to add command line arguments for the filenames. 
```
cd <ROOT PATH>/osm2xodr
python3 main.py
```

#### Step 3 - Import into CARLA

I have tried **method a)** by making a copy of `config.py` called `import_map.py`. This script should load the **.xodr** file into the simulator as the map allow you to adjust the parameters of the import.
```
cp ${CARLA_ROOT}/PythonAPI/util/condif.py ${CARLA_ROOT}/PythonAPI/util/import_map.py 
```
With the simulator running, execute the python script to load the new map. There are no errors, but nothing happens except `No nodes loaded.` is shown. 
```
python3 ${CARLA_ROOT}/PythonAPI/util/import_map.py --osm-path=/home/thill/openstreetmap/map2.xodr
Converting OSM data to opendrive
No nodes loaded.
```
I also tried method **b)** from the tutorial and nothing happens. This method is effectively the same as **a)**, and the results are the same. The command and the outout are shown below. Once again, `No nodes loaded.` is shown. 
```
python3 ${CARLA_ROOT}/PythonAPI/util/config.py --osm-path=/home/thill/openstreetmap/map2.xodr
Converting OSM data to opendrive
No nodes loaded.
```

I commented on this Github post here (https://github.com/carla-simulator/carla/issues/3009), but I have not heard anything back yet. Now I feel dumb, but at least I solved my own problem. I may have been reading the tutorials wrong, or they were misleading. 

The solution is to run `config.py` or your custom script with the `-x` option to load the `.xodr` file. This makes sense right?
```
python3 ${CARLA_ROOT}/PythonAPI/util/config.py -x /home/thill/openstreetmap/map2.xodr
```

The map loads in the simulator. You may have to fly around to see them, but the roads from imported from OpenStreetMap should be shown in the map.

#### Step 4 - Add building models 
For now, this is a two step process. 
##### - OSM2WORLD - 
This Java app can be used to convert a **.osm** file into **.obj** file which contains the buildings and other objects and polygons.
Download the package [here](http://osm2world.org/download/) and extract it somewhere reasonable.
```
cd ~/carla_simulator/OSM2World/latest
java -jar OSM2World.jar
```
```
./osm2world.sh --gui
```
Open a .osm file and save as a .obj file

##### - Blender 
This app can be used to convert a **.obj** file into a **.fbx** file which hopefully can be ingested by carla.
```
sudo apt install blender
```
Run the app.
```
blender
```
Click import mesh **.obj** and after model loads export it as a **.fbx** file.

#### Step 5 - Compile CARLA for Map ingestion 
A compiled versiuon of carla is required for map ingestion and some other tools. There are two options shown below for compiling CARLA.

##### Build CARLA from Source (older method)
I succesfully build carla in 18.04 from source following the docs [here](https://carla.readthedocs.io/en/latest/build_linux/). I ran into errors, but I am making progress. 

I was able to perform the map ingestion by preparing the files manually as described below and running the following. This process needs to be documented and tested further.

```
make package ARGS="--packages=Package03"

```
This results in a standalone asset package directory in `carl/Dist`. 

I beleive that this package must be exported to be used for distribution. 

##### Build CARLA + map tools with Docker (reccomended method for map ingestion)
The reccomended procedure is to build CARLA from the latest source into a docker container using `ue4-docker`. 

Clone a copy of CARLA from [github](https://github.com/carla-simulator/carla) before performing the Docker build. The required Dockerfile is in `carla/Util/Docker`.

Follow this [tutorial](https://github.com/carla-simulator/carla/tree/master/Util/Docker) to build CARLA in a docker. This takes a long time and requires alot of memory and storage.

Setup the dependencies and firewall. Also, setup DNS for docker container (see xyz)
```
pip3 install ue4-docker
```
```
ue4-docker setup
```

Build Unreal Engine and CARLA with Docker
```
cd carla/Util/Docker 
```
```
ue4-docker build 4.24.3 --no-engine --no-minimal
```
Build the image with all the prerequisites to build Carla in a Ubuntu 18.04
```
docker build -t carla-prerequisites -f Prerequisites.Dockerfile .
```
Finally create the actual Carla image, it will search for carla-prerequisites:latest:
This last step requies a 14GB download and a long compile. 
```
docker build -t carla -f Carla.Dockerfile .
```
###### Out of Memory issue
The compile operation may crash if the system runs out of memory. This happened several times during the `compiling shaders` process. The default Ubuntu setup with LVM gives a small ~900 MB swap partition. I increased the swap partition siginificantly following these [instuctions](https://blog.sleeplessbeastie.eu/2020/07/29/how-to-expand-swap-partition-after-system-installation/). The instructions are for 20.04 and the device is encrypted so you cannot follow them exactly. This needs to be documented. You can read about issue [#3590](https://github.com/carla-simulator/carla/issues/3590) on Github.

###### rsync issue
With carla:latest right after 0.9.11 release the `docker build` step failed with a different error. You can read about issue [#3758](https://github.com/carla-simulator/carla/issues/3758) on Github. The fix for this is described in the link, but there was some confusion about the Dockerfile. The file I am using is shown here. I need to learn more about docker to understand this completely. 

1. - In `Util/Docker`, create a file called `MyPackage.sh`
2. - `cd ..`
3. - run `cat BuildTools/Package.sh > Docker/MyPackage.sh`
4. - Change line 166 in `MyPackage.sh` to `copy_if_changed "./Unreal/CarlaUE4/Plugins/" "${DESTINATION}/Plugins/"`
5. - `cd Docker`
6. - change `Carla.Dockerfile` to:

```
FROM carla-prerequisites:latest

ARG GIT_BRANCH

USER ue4

RUN cd /home/ue4 && \
  if [ -z ${GIT_BRANCH+x} ]; then git clone --depth 1 https://github.com/carla-simulator/carla.git; \
  else git clone --depth 1 --branch $GIT_BRANCH https://github.com/carla-simulator/carla.git; fi && \
  cd /home/ue4/carla && \
  ./Update.sh && \
  make CarlaUE4Editor && \
  make PythonAPI && \
  make build.utils
WORKDIR /home/ue4/carla
COPY MyPackage.sh .  
RUN cat MyPackage.sh > Util/BuildTools/Package.sh 
RUN make package
RUN rm -r /home/ue4/carla/Dist

WORKDIR /home/ue4/carla
```
After applying the fix from `@will-sloan` the build finished succesfully. 

##### Prepare Files for Map Ingestion
Preparation of the files is [here](https://carla.readthedocs.io/en/latest/tuto_A_add_map/) in a separate page of the carla docs.
One **.xodr** file and one **.fbx** file are required for each map along with one **.json** file for each map asset package.  I wrote the **.json** file manually, but the carla documents claim that this can be generated automatically. The directory structure of these files is described in the docs.   

Move the files to be ingested into `carla/Import/` of the compiled CARLA package which will generate the assets package. 

```
+-- Import
|    +-- Package03
|    |    +-- Map03
|    |    |    +-- Map03.fbx
|    |    |    +-- Map03.xodr
|    |    +-- Package03.json 
|    +-- README.md
```


##### Ingest and Generate Asset Package 
```
cd ~/carla_simulator/carla/Util/Docker
```
```
./docker_tools.py --input ~/carla_simulator/carla/Import --output ~/carla_simulator/carla/Ingested --packages Package03
```
This will generate a compressed file containing the assets package in the specified directory. Example: `Package03_0.9.11-dirty.tar.gz`

#### Step 6 - Importing Asset Package into CARLA 
Move the compressed asset package to the `carla/Import` directory of the package in which you will use `PythonAPI/util/config.py` to set the map. 
Import the assets with the script provided. 
```
./ImportAssets
```
If the import was successful you should see the asset packages under `carla/CarlaUE4/content/`. With the simulator running, change to a newly imported map. 

```
python3 PythonAPI/examples/spawn_npc.py -n 50
^[[DWARNING: cannot parse georeference: ''. Using default values. 
WARNING: cannot parse georeference: ''. Using default values. 
Nav: failed loading binary 
spawned 50 vehicles and 0 walkers, press Ctrl+C to exit.
^C
destroying 50 vehicles

destroying 0 walkers
```

The import seems to have worked. There is a problem with the location. The spawning locations are not where the town is apparantely. YOU ARE SUPER CLOSE!


### Adding a Map with RoadRunner
(https://carla.readthedocs.io/en/latest/tuto_A_add_map/#introduction)
*RoadRunner is the recommended software to create a map due to its simplicity* - CARLA docs
I have not tried to follow this yet because it requires RoadRunner, but I have requested a trial license from MATHWORKS.
