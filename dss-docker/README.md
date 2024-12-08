# Standard Dataiku DSS Dockerfile

This directory contains the Dockerfile which is used by Dataiku to build the standard docker images for DSS.

To rebuild, run:

    DSS_VERSION=13.3.0
    docker build --build-arg dssVersion=${DSS_VERSION} -t dataiku:${DSS_VERSION} .

To run, with data in `DSS_DATADIR` (e.g `~/dss`)

    docker run -d -p 10000:10000 dataiku:${DSS_VERSION} -v ${DSS_DATADIR}:/home/dataiku/dss

The first time initialize install to