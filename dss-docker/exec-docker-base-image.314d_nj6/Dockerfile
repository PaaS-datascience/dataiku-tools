FROM almalinux:8
WORKDIR /opt/dataiku

RUN . /etc/os-release && case "$VERSION_ID" in \
        7*) echo $'[nginx-stable]\n\
name=nginx stable repo\n\
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/\n\
gpgcheck=0\n\
enabled=1\n\
gpgkey=https://nginx.org/keys/nginx_signing.key\n\
module_hotfixes=true' > /etc/yum.repos.d/nginx.repo;; \
        8*) dnf -qy module enable nginx:1.22;; \
        *) echo 2>&1 'OS version not supported'; exit 1;; \
       esac

RUN yum -y update \
    && yum -y install epel-release \
    && . /etc/os-release && case "$VERSION_ID" in \
        7*) yum -y install procps python3-devel python-devel;; \
        8*) yum -y install procps-ng python36-devel glibc-langpack-en python2-devel;; \
        *) echo 2>&1 'OS version not supported'; exit 1;; \
       esac \
    && yum -y install curl util-linux bzip2 nginx expat zip unzip freetype libgfortran libgomp libicu-devel libcurl-devel openssl-devel libxml2-devel git   \
    && yum -y --setopt=group_package_types="mandatory" install @development \
    && yum -y autoremove \
    && yum clean all

COPY build-python37.sh build/
RUN build/build-python37.sh >/tmp/build-python.log && rm -f /tmp/build-python.log

COPY build-python39.sh build/
RUN . /etc/os-release && case "$VERSION_ID" in \
        7*) build/build-python39.sh >/tmp/build-python.log && rm -f /tmp/build-python.log;; \
        8*) yum -y install python39-devel && yum clean all;; \
        *) echo 2>&1 'OS version not supported'; exit 1;; \
    esac

COPY build-python311.sh build/
RUN build/build-python311.sh >/tmp/build-python.log && rm -f /tmp/build-python.log

COPY build-r41.sh build/
RUN . /etc/os-release && case "$VERSION_ID" in \
        7*) build/build-r41.sh >/tmp/build-r.log && rm -f /tmp/build-r.log;; \
        8*) yum -y install dnf-plugins-core && yum -y config-manager --set-enabled powertools &&  yum -y install R-core-devel && \
         yum -y install freetype-devel libpng-devel libtiff-devel libjpeg-turbo-devel harfbuzz-devel fribidi-devel fontconfig-devel && \
         yum clean all;; \
        *) echo 2>&1 'OS version not supported'; exit 1;; \
    esac

# Copy R build capabilities
COPY install-packages-builtin.sh build/
COPY minimal-packages-base.txt build/
COPY minimal-packages-noconda.txt build/

# Install the equivalent of the base R packages of DSS globally
ENV DKU_R_LIB_SUBPATH=4.x
RUN mkdir -p R/bin R/R.lib/${DKU_R_LIB_SUBPATH}     && rBin=$(command -v R)     && rscriptBin=$(command -v Rscript)     && ln -s "$rBin" "$rscriptBin" R/bin/
ENV DKU_R_DATAIKU_PACKAGES_PATH=/opt/dataiku/R/R.lib/${DKU_R_LIB_SUBPATH}
ENV R_LIBS_USER=${DKU_R_DATAIKU_PACKAGES_PATH}
RUN build/install-packages-builtin.sh R ${DKU_R_LIB_SUBPATH} build/minimal-packages-base.txt https://cloud.r-project.org
RUN build/install-packages-builtin.sh R ${DKU_R_LIB_SUBPATH} build/minimal-packages-noconda.txt https://cloud.r-project.org

# Copy R build capabilities
COPY install-packages-if-needed.sh build/
COPY irkernel-packages-noconda.txt build/

# Install irkernel
RUN build/install-packages-builtin.sh R ${DKU_R_LIB_SUBPATH} build/irkernel-packages-noconda.txt https://cloud.r-project.org
COPY R-exec-wrapper.R R/
COPY R R/R.lib/

COPY _create-virtualenv.sh virtualenv*.pyz install-builtin-env-python-packages.sh resources/builtin-python-env/container-images/ build/
RUN build/_create-virtualenv.sh python3.9 pyenv && \
    build/install-builtin-env-python-packages.sh pyenv/bin/pip && \
    mkdir -p bin && \
    echo -e '#!/bin/bash -e\nexec /opt/dataiku/pyenv/bin/python "$@"' >bin/python && \
    chmod a+x bin/python && \
    rm -rf ~/.cache/pip
COPY dataiku python/dataiku
COPY dataikuapi python/dataikuapi
COPY dataikuscoring python/dataikuscoring
COPY ai_code_assistant python/ai_code_assistant
RUN bin/python  -m compileall -f python || echo "[-] Error precompiling Dataiku Python code (ignored)"
ENV PYTHONPATH=/opt/dataiku/python
COPY web/ /opt/dataiku/web/
COPY resources/nlp /opt/dataiku/resources/nlp/

ENV DKU_SOURCE_LIB_R_PATH=/home/dataiku/lib/instance

WORKDIR /home/dataiku

COPY dss-version.json /opt/dataiku/

# Fake DIP_HOME with tmp folder for R recipes
ENV DIP_HOME=/home/dataiku/fake_dip_home

RUN groupadd -r dataiku     && useradd -r -g dataiku -u 500 -d /home/dataiku dataiku     && mkdir fake_dip_home fake_dip_home/tmp lib lib/project lib/instance plugin     && chown -Rh dataiku:dataiku /home/dataiku

# OpenShift compatibility:
# OpenShift runs containers with an arbitrary uid as an additional security measure
# Thus, we are not "dataiku" and cannot write inside /home/dataiku
# However, we are always gid=0, so we give /home/dataiku to gid 0 and make sure group can
# write into it.
# This is the official recommendation:
# https://docs.openshift.com/container-platform/4.3/openshift_images/create-images.html#images-create-guide-openshift_create-images
#  "Support arbitrary user ids"
#
# More details:
# This is enforced through a Security Context Constraint - see
# https://docs.openshift.com/container-platform/4.3/authentication/managing-security-context-constraints.html
# One of the SCC says
#  Run As User Strategy: MustRunAsRange
#    UID:                    <none>
#    UID Range Min:              <none>
#    UID Range Max:              <none>
# with the range given by an annotation on the project: openshift.io/sa.scc.uid-range=1000540000/10000
#
RUN chgrp -R 0 /home/dataiku && chmod -R 775 /home/dataiku
ENV DKU_CONTAINER_EXEC=1

# USER dataiku
USER 500
ENTRYPOINT ["/opt/dataiku/bin/python", "-m", "dataiku.container.runner"]
