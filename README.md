# resource-projects-etl

This repository contains a library for Extract, Transform and Load processes for ResourceProjects.org.

You can report issues with current transformations, or suggest sources which should be added to this library using the GitHub issue tracker.

This GitHub repository builds two different docker images:
* The Dockerfile in the root dir builds https://registry.hub.docker.com/u/bjwebb/resource-projects-etl/
* The Dockerfile in the `load` dir builds https://registry.hub.docker.com/u/bjwebb/resource-projects-etl-load/

## Processes
Each process, located in the **process** folder consists of a collection of files that either (a) document a manual transformation of the data; or (b) perform an automated transformation.

Folders may contain:

* A README.md file describing the transformation
* An extract.sh or extract.py file to fetch the file
* A data/ subfolder where the extracted data is stored during conversion (ignored by git)
* A transform.py file which runs the transformations
* A meta.json file, containing the meta-data which transform.py will use
* A prov.ttl file containing provenance information (using [PROV-O](www.w3.org/TR/prov-o)) to be merged into the final graph

The output of each process should be written to the root data/ folder, from where it can be loaded onto the ResourceProjects.org platform.


## Running locally

### Requirements

* Python 3
* Bash

### Run

```
virtualenv .ve --python=/usr/bin/python3
source .ve/bin/activate
pip install -r requirements.txt
./transform_all.sh
```

You will then have some data in the data/ directory. Currently the load step can only be run with docker (see "Using a data directory on the host system" below).

## Running with docker

### Requirements

Docker **1.7** (actual requirement may be >=1.6, but 1.7 is what's been tested. This is required because the docker library python image doesn't work otherwise).

### Running from docker hub

```
docker rm -f rp-etl rp-load
docker run --name rp-etl -v /usr/src/app/data -v /usr/src/app/ontology bjwebb/resource-projects-etl
docker run --name rp-load --link virtuoso:virtuoso --volumes-from virtuoso --volumes-from rp-etl --rm bjwebb/resource-projects-etl-load
```

To run the last command you will need [virtuoso container running](https://github.com/NRGI/resourceprojects.org-frontend/#pre-requisites).


### Using a data directory on the host system

To transform the data, and put it in ./data on the host system, run:

```
docker rm -f rp-etl
docker run --name rp-etl -v `pwd`/data:/usr/src/app/data bjwebb/resource-projects-etl
```

To load the data, you can then run:

```
docker run --name rp-load --link virtuoso:virtuoso --volumes-from virtuoso -v `pwd`/data:/usr/src/app/data -v `pwd`/ontology:/usr/src/app/ontology --rm bjwebb/resource-projects-etl-load
```

This load step can also be used to load data not generated using a dockerized step.

### Building docker images

```
docker build -t bjwebb/resource-projects-etl .
docker build -t bjwebb/resource-projects-etl-load load
```

Then run as described above. (You may want to use a different name for your own images, so as not to get confused with those actually from docker hub).

# License

```
Copyright (c) 2015 Natural Resource Governance Institute

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
```
