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
    && yum -y install curl util-linux bzip2 nginx expat zip unzip freetype libgfortran libgomp libicu-devel libcurl-devel openssl-devel libxml2-devel git acl java-11-openjdk-headless  \
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

RUN alternatives --set java java-11-openjdk.x86_64
# From java-11-openjdk.x86_64
ENV JAVA_HOME=/usr/lib/jvm/jre-11-openjdk

MAINTAINER Dataiku <dss@dataiku.com>
ENV SHELL "/bin/bash"
ENV TERM 1
ENV LANG en_us.utf8
RUN groupadd -r dataiku     && useradd -r -g dataiku -u 500 -d /home/dataiku dataiku     && mkdir /home/dataiku     && chown -Rh dataiku:dataiku /home/dataiku
ENV BUILD_TIMESTAMP=$(date)
COPY "dist" /home/dataiku/installdir/dist
COPY "dku-jupyter" /home/dataiku/installdir/dku-jupyter
COPY "dss-version.json" /home/dataiku/installdir/dss-version.json
COPY "dss-arch.txt" /home/dataiku/installdir/dss-arch.txt
COPY "installer.sh" /home/dataiku/installdir/installer.sh
COPY "lib" /home/dataiku/installdir/lib
COPY "python" /home/dataiku/installdir/python
COPY "python39.packages" /home/dataiku/installdir/python39.packages
COPY "R" /home/dataiku/installdir/R
COPY "resources" /home/dataiku/installdir/resources
COPY "scripts" /home/dataiku/installdir/scripts
COPY "conda.packages" /home/dataiku/installdir/conda.packages

RUN rm -f /home/dataiku/installdir/resources/code-envs/python/image-classification/v1/spec/requirements.txt     && rm -f /home/dataiku/installdir/resources/code-envs/python/object-detection/v1/spec/requirements.txt     && rm -rf /home/dataiku/installdir/lib/ivy/govern-server-run     && rm -f /home/dataiku/installdir/lib/ivy/backend-run/jnr-posix-3.0.44.jar     && rm -rf /home/dataiku/installdir/python36.packages/nbconvert*

RUN /bin/su - dataiku -c "/home/dataiku/installdir/installer.sh -y -t design -d /home/dataiku/data -p 12000 -P python3.9 -j"
RUN /bin/su - dataiku -c "/home/dataiku/data/bin/dssadmin install-R-integration -noDeps -repo https://cloud.r-project.org"
WORKDIR /opt/dataiku
USER root

WORKDIR /home/dataiku
# USER dataiku
USER 500
ENTRYPOINT ["/home/dataiku/data/bin/cde"]
