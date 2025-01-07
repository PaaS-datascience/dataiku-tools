# dataiku-tools

This is an unofficial fork from Dataiku' dataiku-tools repository.

## Goals

- Provide up to date automation tools (docker, kubernetes, ansible not supported for now)
- Define service management policies

## Dataiku documentation sources


### Industry data science use cases - Dataiku Gallery
https://gallery.dataiku.com/
Next steps - training by industry

### Dataiku Community
https://community.dataiku.com/

links to every doc

### Training - Dataiky Academy
https://academy.dataiku.com
Next steps - training policy

### Software documentation
https://doc.dataiku.com/

### Dataiku Knowledge base
https://knowledge.dataiku.com/

### Dataiku developer guide
https://developer.dataiku.com/


## Service management policies

### Management levels & support

#### Dataiku global architecture

#### Dataiku support levels

Support levels are defined by Dataiku for each feature and service :
- Supported (default)
- Experimental
- Tier 2 support
- Not supported
- Public Preview
- Deprecated

cf https://doc.dataiku.com/dss/latest/troubleshooting/support-tiers.html

#### SaaS platform

This is the recommended mode by dataiku and offers a


#### Self-managed lifecycle - Kubernetes

##### Build design node kubernetes image

With `DSS_VERSION`set to be dss version to be build (e.g. 13.1.1):

    cd dss-docker
    DSS_VERSION=13.3.1
    docker build --build-arg dssVersion=${DSS_VERSION} -t dataiku:${DSS_VERSION} .

Docker image release note:

- design node is in almalinux 9 but dss will build other containers in almalinux 8 (e2e consistency to be tested)
- supports R
- includes python 3.11 support (3.10+ required for markitdown - note: dataiku don't support python 3.12)
- supports graphics exports
- includes docker & kube binaries to enable to build images for k8s container execution

##### Local run (test mode)

To run, with data in `DSS_DATADIR` (e.g `~/dss`)

    mkdir -p ${DSS_DATADIR}
    docker run -d -p 10000:10000 -v ${DSS_DATADIR}:/home/dataiku/dss dataiku:${DSS_VERSION}

##### Deploy design node to kubernetes

    cd kubernetes
    kubectl apply -f dataiku.yaml

Kubernetes configurqtion release note:

- supports docker-in-docker (dind) with a sidecar container (support) as it is mandatory to enable dssadmin cli to build containers for k8s exectution


### Dataiku data lifecycle management

### Git for projects

