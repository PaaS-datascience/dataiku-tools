# Standard Dataiku DSS Dockerfile

This directory contains the Dockerfile which is used by Dataiku to build the standard docker images for DSS.

To rebuild, run (around 30 minutes):

    DSS_VERSION=13.3.0
    docker build --build-arg dssVersion=${DSS_VERSION} -t dataiku:${DSS_VERSION} .

To run, with data in `DSS_DATADIR` (e.g `~/dss`)

    mkdir -p ${DSS_DATADIR}
    docker run -d -p 10000:10000 -v ${DSS_DATADIR}:/home/dataiku/dss dataiku:${DSS_VERSION}

The first time you run, post install process will setup `${DSS_DATADIR}` - this may be 10 min long