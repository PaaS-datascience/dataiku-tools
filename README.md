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

Create kubectl secret with the following command to enable usage with the dataiku design instance:

    kubectl create secret generic kubeconfig-secret --from-file=config -n dataiku

You must also create a secret to access to the container registry:

    kubectl -n dataiku create secret docker-registry container-registry-secret  --docker-server ${DOCKER_REGISTRY} --docker-username=${DOCKER_USER} --docker-password=${DOCKER_PASSWORD}

##### Build design node kubernetes image

With `DSS_VERSION`set to be dss version to be build (e.g. 13.5.3):

    cd dss-docker
    DSS_VERSION=13.5.3
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

##### Enabling elastic AI computation with kubernetes

Follow official Dataiku documentation for [Managed Kubernetes clusters](https://doc.dataiku.com/dss/latest/containers/managed-k8s-clusters.html) with [Google GKE](https://doc.dataiku.com/dss/latest/containers/gke/index.html), [Amazon EKS](https://doc.dataiku.com/dss/latest/containers/eks/index.html) or [Microsoft Azure AKS](https://doc.dataiku.com/dss/latest/containers/aks/index.html) or [Custom Kubernetes or Openshift clusters](https://doc.dataiku.com/dss/latest/containers/unmanaged-k8s-clusters.html)

##### Building base and cde images

If admin from UI don't work or if you want to enable custom registry and to publish Dockerfiles, here are some commands to use:

    DOCKER_REGISTRY=rg.fr-par.scw.cloud/mywonderfulregistry
    DOCKER_USER=myuser
    DOCKER_PASSWORD=mypassword
    DSS_VERSION=13.5.3

    kubectl -n dataiku exec -it $(kubectl -n dataiku get pod | grep '2/2' | awk '{print $1}') -- docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}

    # build base image
    kubectl exec -it -n dataiku $(kubectl -n dataiku get pod | grep '2/2' | awk '{print $1}')  -- sh -c "cd dss;./bin/dssadmin build-base-image --type container-exec --with-py311 --with-py39 && docker tag dku-exec-base-ru4oxgmkpuoy4djmkkuvxfng:dss-${DSS_VERSION} ${DOCKER_REGISTRY}/dku-exec-base-ru4oxgmkpuoy4djmkkuvxfng:dss-${DSS_VERSION} && docker push ${DOCKER_REGISTRY}/dku-exec-base-ru4oxgmkpuoy4djmkkuvxfng:dss-${DSS_VERSION}"

    # build cde image
    kubectl exec -it -n dataiku $(kubectl -n dataiku get pod | grep '2/2' | awk '{print $1}') -- sh -c "cd dss;./bin/dssadmin build-base-image --type cde --with-py311 --with-py39 && docker tag dku-cde-base-ru4oxgmkpuoy4djmkkuvxfng:dss-${DSS_VERSION} ${DOCKER_REGISTRY}/dku-cde-base-ru4oxgmkpuoy4djmkkuvxfng:dss-${DSS_VERSION} && docker push ${DOCKER_REGISTRY}/dku-cde-base-ru4oxgmkpuoy4djmkkuvxfng:dss-${DSS_VERSION}"

With those two files built, you will be able to enable following features in Kubernetes:

- VSCode execution within Dataiku
- Jupyter Notebooks
- Recipe code execution
- Webapp / API / ML Scoring API

### Dataiku data lifecycle management

### Git for projects

